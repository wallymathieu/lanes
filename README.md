# lanes

[![Build Status][gh-actions-badge]][gh-actions]
[![LFE Versions][lfe badge]][lfe]
[![Erlang Versions][erlang badge]][versions]
[![Tags][github tags badge]][github tags]

[![][logo]][logo-large]

*A slightly more general LFE HTTP routing library than lfest*

## Introduction

The lanes project aims to offer some of the YAWS-specific features of the [lfest project](https://github.com/lfex/lfest) to a wider selection of BEAM-based web servers. This is done with the understanding that the original design of lfest (and thus the design inherited in the lanes project) is not optimal.

For now, though, we are focused on the immediate and practical needs of LFE application developers.

## Dependencies

* Erlang 20+
* `rebar3`

## Compatibility

Releases of Elli map to the following versions in its dependencies:

* `0.3.0` - LFE 2.1.1, Erlang 20-26, Rebar 3.22, rebar2_lfe 0.4.2 (with examples using logjam 1.0.5, Elli 3.3.0, Barista 0.3.2)
* `0.2.0` - LFE 2.0.1, Erlang 20-25, Rebar 3.16, rebar3_lfe 0.3.1 (with examples using logjam 1.0.0, Elli 3.3.0, Barista 0.3.2)

## Usage

Create your application/service routes with the `(defroutes ...)` form.
Here is an example that is compatible with [Elli](https://github.com/elli-lib/elli):

```cl
(include-lib "lanes_elli/include/macros.lfe")

(defroutes
  ;; top-level
  ('GET #"/"
        (lanes.elli:ok "Welcome to the Volvo Store!"))
  ;; single order operations
  ('POST #"/order"
         (progn
           (lanes-elli-data:create-order (lanes.elli:get-data req))
           (lanes.elli:accepted)))
  ('GET #"/order/:id"
        (lanes.elli:ok
         (lanes-elli-data:get-order id)))
  ('PUT #"/order/:id"
        (progn
          (lanes-elli-data:update-order id (lanes.elli:get-data req))
          (lanes.elli:no-content)))
  ('DELETE #"/order/:id"
           (progn
             (lanes-elli-data:delete-order id)
             (lanes.elli:no-content)))
  ;; order collection operations
  ('GET #"/orders"
        (lanes.elli:ok
         (lanes-elli-data:get-orders)))
  ;; payment operations
  ('PUT #"/payment/order/:id"
        (progn
          (lanes-elli-data:make-payment id (lanes.elli:get-data req))
          (lanes.elli:no-content)))
  ('GET #"/payment/order/:id"
        (lanes.elli:ok
         (lanes-elli-data:get-payment-status id)))
  ;; error conditions
  ('ALLOWONLY ('GET 'POST 'PUT 'DELETE)
              (lanes.elli:method-not-allowed))
  ('NOTFOUND
   (lanes.elli:not-found "Bad path: invalid operation.")))
```

For full context, be sure to see the code in `./examples`.

### Consuming Routes

#### Barista

[Barista](https://github.com/lfex/barista) is a thin wrapper around the
Erlang standard library's httpd, written in LFE. The `lanes-barista`
module supports a `defroutes` macro that generates a `handle/3` function
and allows barista web applications to dispatch based upon request method and
path.

#### Cowboy

TBD

#### Elli

Wriing Elli applications in LFE with lanes is very similar as in Erlang:
the only difference is that you don't create a `handle/3` function. Instead,
you use `defroutes` which creates `handle/3` under the covers for you.
Everything else is vanilla Elli. To be clear, one still needs to provide the
`handle/2` and `handle_event/3` functions in the module where `defroutes` is
called.

#### Nova

TBD

#### YAWS

WARNING: YAWS support is old and needs to be re-visited to make sure everything
still works ...

The YAWS `lanes` plugin creates a `routes/3` function which can then
be called in the `out/1` function that is required of a
[YAWS appmod](http://yaws.hyber.org/appmods.yaws) module.
For an example of this in action, see
[this mini REST-api](https://github.com/lfex/yaws-rest-starter/blob/master/src/yrests-store-3.lfe).

A few important things to note here:)

* Each route is composed of an HTTP verb, a path, and a function to execute
  should both the verb and path match.
* The function call in the route has access to the `arg-data` passed from
  YAWS; this contains all the data you could conceivably need to process a
  request. (You may need to import the `yaws_api.hrl` in your module to
  parse the data of your choice, though.)
* If a path has a segment preceded by a colon, this will be converted to a
  variable by the `(defroutes ...)` macro; the variable will then be
  accessible from the function you provide in that route.
* The `(defroutes ...)` macro generates the `routes/3` function; it's
  three arguments are the HTTP verb (method name), the path info (a list of
  path segments, with the `":varname"` segments converted to`varname`/
  variable segments), and then the `arg-data` variable from YAWS.

More details:

lanes needs to provide YAWS with an `out/1` function. The location of this
function is configured in your `etc/yaws.conf` file in the
`<appmods ...>` directives (it can be repeated for supporting multiple
endpoints).

YAWS will call this function with one argument: the YAWS `arg` record
data. Since this function is the entry point for applications running under
YAWS, it is responsible for determining how to process all requests.

The `out/1` function in lane+YAWS-based apps calls the `routes/3` function
generated by the `(defroutes ...)` lanes/YAWS mamcro.

When a lanes-based project is compiled, the `routes/3` function is available for use via
whatever modules have defined routes with `defroutes`.

## License [&#x219F;](#contents)

Apache Version 2 License

Copyright © 2014-2021, Duncan McGreggor <oubiwann@gmail.com>

[//]: ---Named-Links---

[logo]: priv/images/logo.jpg
[logo-large]: priv/images/logo-large.jpg
[gh-actions-badge]: https://github.com/lfex/lanes/workflows/ci%2Fcd/badge.svg
[gh-actions]: https://github.com/lfex/lanes/actions
[lfe]: https://github.com/rvirding/lfe
[lfe badge]: https://img.shields.io/badge/lfe-2.0-blue.svg
[erlang badge]: https://img.shields.io/badge/erlang-201%20to%2025-blue.svg
[versions]: https://github.com/lfex/lanes/blob/master/.github/workflows/cicd.yml
[github tags]: https://github.com/lfex/lanes/tags
[github tags badge]: https://img.shields.io/github/tag/lfex/lanes.svg
