# Ring Websocket Async

A Clojure library for using [core.async][] with [Ring's][] websocket API
(currently in alpha testing).

[core.async]: https://github.com/clojure/core.async
[ring's]: https://github.com/ring-clojure/ring

## Installlation

Add the following dependency to your deps.edn file:

    org.ring-clojure/ring-websocket-async {:mvn {"0.1.0-SNAPSHOT"}}

Or to your Leiningen project file:

    [org.ring-clojure/ring-websocket-async "0.1.0-SNAPSHOT"]

## Usage

The most convenient way to get started is to use the `go-websocket`
macro. Here's an example that echos any message received back to the
client:

```clojure
(require '[clojure.core.async :as a :refer [<! >!]]
         '[ring.websocket.async :as wsa])

(defn echo-websocket-handler [request]
  (wsa/go-websocket [in out err]
    (loop []
      (when-let [mesg (<! in)]
        (>! out mesg)
        (recur)))))
```

The macro sets three binding variables:

* `in`  - the input channel
* `out` - the output channel
* `err` - the error channel

Then executes its body in a core.async `go` block. When the block
completes, the websocket is closed by the server. If the client closes
the websocket, the associated channels are also closed.

To close the connection from the server with an error code, you can use
the `closed` function to send a special message to the output channel:

```clojure
(defn closes-with-error [request]
  (wsa/go-websocket [in out err]
    (>! out (wsa/closed 1001 "Gone Away"))))
```

A Ring websocket response will be returned by the macro, so you can
directly return it from the handler.

If you want more control, you can use the lower-level
`websocket-listener` function. The following handler example is
equivalent to the one using the `go-websocket` macro:

```clojure
(defn echo-websocket-handler [request]
  (let [in  (a/chan)
        out (a/chan)
        err (a/chan)]
    (go (try
          (loop []
            (when-let [mesg (<! in)]
              (>! out mesg)
              (recur)))
          (finally
            (a/close! out))))  ;; closes the websocket
    {:ring.websocket/listener (websocket-listener in out err)}))
```

## License

Copyright © 2023 James Reeves

Released under the MIT license.
