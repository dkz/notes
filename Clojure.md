# Clojure

Default REPL experience can be improved a little by altering `.lein/profiles.clj`.

```clojure
{:user
 {:dependencies
  [[org.clojure/tools.namespace "1.4.4"]
   [org.clojure/tools.trace "0.7.11"]
   [com.bhauman/rebel-readline "0.1.4"]]
  :aliases
  {"rebl" ["trampoline" "run" "-m" "rebel-readline.main"]}}
 :standalone
 {:dependencies
  [[aleph "0.6.1"]]
  :repl-options {:init
                 (require '[clojure.repl :refer [doc dir]])}}}
```

`lein rebl` enables [REBL](https://github.com/bhauman/rebel-readline),
but unfortunately requires an existing project in order to work properly.

Default project template lacks convenient tooling.
I used the following `project.clj` for the latest project:

```clojure
(defproject sample-project "0.1.1"
  :dependencies
  [[org.clojure/clojure "1.11.1"]
   [org.clj-commons/byte-streams "0.3.2"]
   [lambdaisland/uri "1.15.125"]
   [environ "1.2.0"]
   [cheshire "5.11.0"]
   [aleph "0.6.1"]]
  :main ^:skip-aot sample-project.core
  :target-path "target/%s"
  :profiles {:dev {:source-paths ["dev"]
                   :repl-options {:init-ns user}
                   :dependencies [[org.clojure/tools.namespace "1.4.4"]
                                  [org.clojure/tools.trace "0.7.11"]]}
             :uberjar {:aot :all
                       :jvm-opts ["-Dclojure.compiler.direct-linking=true"]}})
```

And the following `dev/user.clj`:

```clojure
(ns user
  (:require
    [clojure.repl :refer :all]
    [clojure.pprint :refer [pprint print-table]]
    [clojure.tools.namespace.repl :refer [refresh refresh-all]]
    [clojure.tools.trace :refer [trace-vars untrace-vars]]
    [manifold.deferred :as d]
    [sample-project.core :as core]))

(def system nil)

(defn init
  []
  (alter-var-root #'system (constantly (core/start (core/system)))))

(defn stop
  []
  (alter-var-root #'system (fn [s] (when s (core/stop s)))))

(defn reset
  []
  (stop)
  (refresh :after 'user/init))
```

This template ensures `user` namespace gets populated with tools to facilitate
REPL experimentation.

List of immensely useful libraries so far:

- [Environ](https://github.com/weavejester/environ), `[environ "1.2.0"]`: managing environment settings
- [Cheshire](https://github.com/dakrone/cheshire), `[cheshire "5.11.0"]`: JSON library
- [Aleph](https://github.com/clj-commons/aleph), `[aleph "0.6.1"]`: asynchronous communications and HTTP client library
- [URI](https://github.com/lambdaisland/uri), `[lambdaisland/uri "1.15.125"]`: URI construction/manipulation
- [Bytestreams](https://github.com/clj-commons/byte-streams), `[org.clj-commons/byte-streams "0.3.2"]`: utility for working with byte streams and sequences

