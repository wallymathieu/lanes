# lfest [![Build Status](https://travis-ci.org/lfe/lfest.png?branch=master)](https://travis-ci.org/lfe/lfest)

<img src="resources/images/Banners-And-Confetti.png"/>

*Macros and functions for LFE+REST on YAWS*


Introduction
============

REST is a party, and you know it.


Dependencies
------------

This project assumes that you have [rebar](https://github.com/rebar/rebar)
and [lfetool]() installed somwhere in your ``$PATH``.

This project depends upon the following, which are automatically installed
to the ``deps`` directory of this project when you run ``make compile``:

* [LFE](https://github.com/rvirding/lfe) - Lisp Flavored Erlang; needed to
  compile
* [YAWS]() - needed for the header file


Installation
============

Just add it to your ``rebar.config`` deps:

```erlang

{deps, [
    ...
    {lfest, ".*", {git, "git@github.com:lfe/lfest.git", "master"}}
  ]}.
```

If you have created your project with ``lfetool``, you can download
``lfeest`` with the following:

```bash
$ make get-deps
```

Or, you can have it download automatically when you compile:

```bash
$ make compile
```


Usage
=====

Create your application/service routes with the ``(defroutes ...)`` form.
Here is an example:

```cl
(include-lib "deps/lfest/include/macros.lfe")

(defroutes
  ;; top-level
  ('GET "/"
        (lfest-html-resp:ok "Welcome to the Volvo Store!"))
  ;; single order operations
  ('POST "/order"
         (create-order (lfest:get-data arg-data)))
  ('GET "/order/:id"
        (get-order id))
  ('PUT "/order/:id"
        (update-order id (lfest:get-data arg-data)))
  ('DELETE "/order/:id"
           (delete-order id))
  ;; order collection operations
  ('GET "/orders"
        (get-orders))
  ;; payment operations
  ('GET "/payment/order/:id"
        (get-payment-status id))
  ('PUT "/payment/order/:id"
        (make-payment id (lfest:get-data arg-data)))
  ;; error conditions
  ('ALLOWONLY
    ('GET 'POST 'PUT 'DELETE)
    (lfest-json-resp:method-not-allowed))
  ('NOTFOUND
    (lfest-json-resp:not-found "Bad path: invalid operation.")))
```

Note that this creates the ``routes/3`` function which can then be called
in the ``out/1`` function.

A few important things to note here:

* Each route is composed of an HTTP verb, a path, and a function to execute
  should both the verb and path match.
* The function call in the route has access to the ``arg-data`` passed from
  YAWS; this contains all the data you could conceivably need to process a
  request. (You may need to import the ``yaws_api.hrl`` in your module to
  parse the data of your choice, though.)
* If a path has a segment preceded by a colon, this will be converted to a
  variable by the ``(defroutes ...)`` macro; the variable will then be
  accessible from the route function.
* The ``(defroutes ...)`` mamcro generates the ``routes/3`` function; it's
  three arguments are the HTTP verb (method name), the path info (a list of
  path segments, with the ``":varname"`` segments converted to ``varname``/
  variable segments), and then the ``arg-data`` variable from YAWS.


Concepts
========

lfest needs to provide YAWS with an ``out/1`` function. The location of this
function is configured in your ``etc/yaws.conf`` file in the
``<appmods ...>`` directives (it can be repeated for supporting multiple
endpoints).

YAWS will call this function with one argument: the YAWS ``arg`` record
data. Since this function is the entry point for applications running under
YAWS, it is responsible for determining how to process all requests.

The ``out/1`` function in lfest-based apps calls the ``routes/3`` function
generated by the ``(defroutes ...)`` mamcro.

The route definition macro does some pretty heavy remixing of the routes
defined in ``(defroutes ...)``. The route definition given in the "Usage"
section above actually expands to the following LFE before being compiled to
a ``.beam``:

```cl
 #((define-function routes
     (match-lambda
       (('GET () arg-data)
        (call 'lfest-html-resp 'ok "Welcome to the Volvo Store!"))
       (('POST ("order") arg-data)
        (create-order (call 'lfest 'get-data arg-data)))
       (('GET ("order" id) arg-data) (get-order id))
       (('PUT ("order" id) arg-data)
        (update-order id (call 'lfest 'get-data arg-data)))
       (('DELETE ("order" id) arg-data) (delete-order id))
       (('GET ("orders") arg-data) (get-orders))
       (('GET ("payment" "order" id) arg-data) (get-payment-status id))
       (('PUT ("payment" "order" id) arg-data)
        (make-payment id (call 'lfest 'get-data arg-data)))
       ((method p a)
        (when
         (not
          (if (call 'erlang '=:= 'GET method)
            'true
            (if (call 'erlang '=:= 'POST method)
              'true
              (if (call 'erlang '=:= 'PUT method)
                'true
                (call 'erlang '=:= 'DELETE method))))))
        (call 'lfest-json-resp 'method-not-allowed))
       ((method path arg-data)
        (call 'lfest-json-resp 'not-found "Bad path: invalid operation."))))
   6)
```

When it is compiled, the ``routes/3`` function is available for use from
wherever you have defined your routes.
