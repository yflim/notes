
# Table of Contents

1.  [Tools](#org57f22ce)
2.  [Basics / Miscellanea](#orgfc87204)
    1.  [Help](#orgce09968)
3.  [Syntax](#org472b274)
4.  [Collections](#org0a300d8)
    1.  [walk, pre-walk, post-walk](#org9323c9f)
5.  [Sequences](#orgfc59096)
6.  [Transient data structures](#org869315b)
7.  [Functions beget functions](#org893420c)
8.  [Doing](#org1fb3836)
9.  [Branching](#org2056042)
10. [Macros](#orge5c8e38)
11. [Namespaces](#org96fbfb0)
    1.  [Dependencies](#org45b21e4)
12. [Destructuring](#orgf088053)
    1.  [Sequential destructuring](#org93166e1)
    2.  [Associative destructuring](#orga10b1a3)
        1.  [Keyword arguments](#orgf9c4309)
        2.  [Namespaced keywords](#org0dd58f2)
        3.  [Nested maps](#org75bb3ca)
13. [Abstractions](#org96ef204)
    1.  [Multimethods](#orgbe1867c)
    2.  [Objects](#orgbf64971)
14. [State](#orgc6d12d0)
15. [Reading clojure characters](#org201dafd)
    1.  [Symbols and Vars](#org679d3cf)
    2.  [`^`: Metadata](#orga034dc0)
16. [IO](#org6317e17)
17. [Vars and global environment](#orgb4d2c6c)
18. [Testing](#org51d4242)
    1.  [Kaocha](#org5a42f7a)
19. [Web](#org1dc5bee)
    1.  [clj-http](#org38e035d)



<a id="org57f22ce"></a>

# Tools

-   Find dependency versions:

`$ clj -X:deps find-versions :lib clojure.java-time/clojure.java-time`

-   Run build function, e.g. `$ clj -T:build install`
-   Create new library: `clojure -Tclj-new lib :name myname/mylib`


<a id="orgfc87204"></a>

# Basics / Miscellanea

-   Unused argument: \_, e.g. `(fn [_] do-something)`

-   Anonymous functions:
    
        (fn [msg] (println msg))
        ;; equivalent:
        #(println %)
        ;; positional args: %1, %2, etc.

-   Useful functions:
    
    -   `name`: Returns the name String of a string, symbol or keyword, e.g.
    
        (name :a)
        (name "a")
        (name 'a)

-   Using a main
    
    -   With entry function `run`:
    
    `$ clj -X hello/run`
    
    -   Or using entry function `-main`
    -   Launching a script:
        `$ clj -M /path/to/myscript.clj arg1 arg2 arg3`, where
        `*command-line-args*` => `("arg1" "arg2" "arg3")`
    -   [Babashka?](https://book.babashka.org/)

-   `*clojure-version*`

-   `#_` (optionally followed by a space) tells the reader to ignore the next form completely.
    -   Can be stacked to omit multiple forms

,#+begin<sub>src</sub> clojure
[1 2 3 #\_ 4 5]
;; => [1 2 3 5]
{:a 1, #\_#\_ :b 2, :c 3}
;; => {:a 1, :c 3}
\#+end<sub>src</sub>

-   Equality: `=` is a value-based operation, while `identical?` tests whether its arguments are the same object

    (identical? [1] [1])
    ;; => false
    (= [1] [1])
    ;; => true

-   Value of last evaluated expression: `*1`. Analogously, `*2` and `*3` (no higher).

-   Shell commands:

    (require '[clojure.java.shell :as sh])
    (sh/sh "ls" "-aul")


<a id="orgce09968"></a>

## Help

-   Print public vars (including functions) in a namespace: `(dir namespace)`
-   All namespaces: `(all-ns)`
-   All public definitions in all currently-loaded namespaces matching regex or stringable thing: `(apropos str-or-pattern)`
-   View source code for `sym`: `(source sym)`


<a id="org472b274"></a>

# Syntax

-   `..`
    E.g.: `(.. System (getProperties) (get "os.name"))`
    expands to `(. (. System (getProperties)) (get "os.name"))`


<a id="org0a300d8"></a>

# Collections

-   `vector` vs. `vec`

    (vector '(1 2 3))
    ;; => [(1 2 3)]
    (vec '(1 2 3))
    ;; => [1 2 3]

-   `replace`: `(replace smap)`, `(replace smap coll)`

Note: 1. Also applies to sequences. 2. similarities with and differences from `map`

    (replace [10 9 8 7 6] [0 2 4])
    ;; [10 8 6]
    (replace {2 :two, 4 :four} [4 2 3 4 5 6 2])
    ;; [:four :two 3 :four 5 6 :two]
    (map {2 :two, 4 :four} [4 2 3 4 5 6 2])
    ;; (:four :two nil :four nil nil :two)
    (replace [] [0])
    ;; [0]
    (map [] [0])
    ;; IndexOutOfBoundsException ...

-   `index`: `(index xrel ks)`

,#+begin<sub>src</sub> clojure
(require &rsquo;[clojure.set :refer [index]])
(def weights #{ {:name &rsquo;betsy :weight 1000}
                {:name &rsquo;jake :weight 756}
                {:name &rsquo;shyq :weight 1000} })
(index weights [:weight])
;; {{:weight 756}  #{{:name jake, :weight 756}},
;; {:weight 1000} #{{:name shyq, :weight 1000}
;;                  {:name betsy, :weight 1000}}}
\#+end<sub>src</sub>

-   ArrayMap a.k.a. array-map, e.g. `(array-map :a 10)`
    -   Maintains key order. Linear lookup performance, &ldquo;only suitable for *very small* maps&rdquo;.
    -   Only maintains order when un-&ldquo;modified&rdquo;. Subsequent assoc-ing &ldquo;eventually&rdquo; causes it to &ldquo;become&rdquo; a hash-map.

-   `into`: `(into)`, `(into to)`, `(into to from)`, `(into to xform from)`

Conjoin all items of `from` into `to`, e.g.

    (into (sorted-map) [{:a 1} {:c 3} {:b 2}])
    ;; => {:a 1, :b 2, :c 3}
    (into [] {1 2, 3 4})
    ;; => [[1 2] [3 4]]
    (into '() [1 2 3])
    ;; => (3 2 1)

-   `reduced`: `(reduced x)` wraps x such that a `reduce` terminates with the value x

-   `subvec`: `(subvec v start)` `(subvec v start end)`

    (subvec [1 2 3 4 5 6 7] 2)
    ;; => [3 4 5 6 7]
    (subvec [1 2 3 4 5 6 7] 2 4)
    ;; => [3 4]

-   `zipmap`: `(zipmap keys vals)`

    (zipmap [:a :b :c :d :e] [1 2 3 4 5])
    ;; {:a 1, :b 2, :c 3, :d 4, :e 5}
    (zipmap [:a :b :c] [1 2])
    ;; {:a 1, :b 2}
    (zipmap [:a :b :c] (repeat nil))
    ;; {:a nil, :b nil, :c nil}


<a id="org9323c9f"></a>

## TODO walk, pre-walk, post-walk


<a id="orgfc59096"></a>

# Sequences

-   `repeat`: `(repeat x)`, `(repeat n x)`. Note: result is lazy.

    (take 5 (repeat "x"))
    ;; => ("x" "x" "x" "x" "x")
    (repeat 5 "x")
    ;; => ("x" "x" "x" "x" "x")

-   `list*`

    (list* 1 2 [3 4])
    ;; => (1 2 3)
    ;; Corner cases:
    (list* nil [1 2])
    ;; => (nil 1 2)
    (list* 1 nil)
    ;; => (1)
    (list* () [1 2])
    ;; => (() 1 2)
    (list* 1 ())
    ;; => (1)


<a id="org869315b"></a>

# Transient data structures

-   Vectors, hash-maps, and hash-sets supported
-   `(transient ds)`: O(1) creation from persistent data structures
-   Shares structure with persistent source
-   Interim return values must be returned
-   `(persistent! ds)`: O(1) creation of persistent data structure when done building results
    -   Transient must not be used after: all operations will throw exceptions.
-   Same code structure as functional version: `conj!`, `pop!`, `assoc!`, `dissoc!`, `disj!`


<a id="org893420c"></a>

# Functions beget functions

-   `apply`
    
        (def strs ["a" "b" "c"])
        (apply str strs) ;; => (str "a" "b" "c")
        ;; => "abc"
        (apply str 1 strs) ;; => (str 1 "a" "b" "c")
        ;; => "1abc"

-   `comp`: `((comp f g) x)` => `(f (g x))`

-   `juxt` (possibly abusing notation): ((juxt a b c) & args) => [(a & args) (b & args) (c & args)]
    
        ((juxt str #(* 2 %)) 1)
        ;; => ["1" 2]
        ((juxt #(println %1 %2) #(* %1 %2)) 1 2)
        1 2
        ;; => [nil 2]

-   `partial`, e.g.
    
        (defn add [x y] (+ x y))
        (def add-5 (partial add 5))
        (add-5 10)
        ;; => 15


<a id="org1fb3836"></a>

# Doing

-   `doseq`: `(doseq seq-exprs & body)`
    
    -   Repeatedly executes body with bindings and filterings as provided by &ldquo;for&rdquo;. Does not retain head of sequence. Returns nil.
    
        ;; Multiplies every x by every y. Prints -1, -2, -3, 0, ...
        (doseq [x [-1 0 1]
                y [1  2 3]]
          (prn (* x y)))
        ;; Prints 1, 4, 9
        (doseq [[x y] (map list [1 2 3] [1 2 3])]
          (prn (* x y)))
        ;; Prints 3, 8
        (doseq [[[a b] [c d]] (map list (sorted-map :1 1 :2 2) (sorted-map :3 3 :4 4))]
          (prn (* b d)))
        ;; Prints :1 1, :2 2, :3 3
        (doseq [[k v] {:1 1 :2 2 :3 3}]
          (prn k v))
    
    where
    
        (map list [1 2 3] [1 2 3])
        ;; => ((1 1) (2 2) (3 3))
        (map list (sorted-map :1 1 :2 2) (sorted-map :3 3 :4 4))
        ;; => (([:1 1] [:3 3]) ([:2 2] [:4 4]))

-   `dorun`: Force side effects in producing lazy sequences. Does not retain head of sequence; returns nil.
    
        (dorun 5 (repeatedly #(println "hi")))
        ;; prints "hi" 6 times
        (let [x (atom 0)]
          (dorun (take 10 (repeatedly #(swap! x inc))))
          @x)
        ;; => 10
        (dorun (map #(println "hi" %) ["mum" "dad" "sister"]))
        hi mum
        hi dad
        hi sister
        ;; => nil

-   `doall`: Like `doseq` except the sequence head is retained and returned; entire seq kept in memory. E.g.:
    `(doall (map println [1 2 3]))`
    1
    2
    3
    ;; => (nil nil nil)


<a id="org2056042"></a>

# Branching

-   `when`, `when-not`, `when-let`, `when-first`

-   `cond`: `(cond & clauses)`
    Clauses: a set of test-expr pairs. Returns the expr corresponding to the first logical-true test. E.g.
    
        (defn pos-neg-or-zero
        "Determines whether or not n is positive, negative, or zero"
        [n]
        (cond
          (< n 0) "negative"
          (> n 0) "positive"
          :else "zero"))

-   `condp`: `(condp pred expr & clauses)`
    Each clause takes either form:
    `text-expr result-expr`
    `text-expr :>> result-fn`
    -   For each clause, `(pred test-expr expr)` is evaluated.
        -   Binary clause matches: result-expr is returned.
        -   Ternary clause matches: the result of calling result-fn, a unary function, with the result of the predicate is returned.
    -   E.g.:
        
            (condp some [1 2 3 4]
              #{0 6 7} :>> inc
              #{4 5 9} :>> dec
              #{1 2 3} :>> #(+ % 3)
              42 ; default)
            ;;=> 3

-   `cond->`: `(cond-> expr & clauses)`
    where clauses is a set of test-form pairs. Threads expr through each form corresponding to a test evaluating to true. Does not short-circuit after the first true test.
    
        (cond-> 1          ; we start with 1
          true inc       ; the condition is true so (inc 1) => 2
          false (* 42)   ; the condition is false so the operation is skipped
          (= 2 2) (* 3)) ; (= 2 2) is true so (* 2 3) => 6
        ;;=> 6

-   `case`: `(case e & clauses)`
    where a clause can take either form:
    `test-constant result-expr`
    `(test-constant1 ... test-constantN)  result-expr`
    where e is matched (in constant time, unlike `cond` and `condp`) against each test-constant in the latter. A single default expression can follow the clauses; if none is provided and no clause matches, an IllegalArgumentException is thrown.
    
    -   The test-constants are not evaluated and must be compile-time literals, and need not be quoted.
    -   E.g.
    
        (let [myseq '(1 2)]
        (case myseq
              (()) "empty seq"
              ((1 2)) "my seq"
              "default"))
        ;;=> "my seq"
        (let [myseq '(1 2)]
        (case myseq
              () "empty seq"
              '(1 2) "my seq"
              "default"))
        ;; => "my seq"
        (let [myvec [1 2]]
          (case myvec
                [] "empty vec"
                (vec '(1 2)) "my vec"
                "default"))
        ;;=> "default"


<a id="orge5c8e38"></a>

# Macros

-   `some->`: `(some-> expr & forms)`
    When `expr` is not nil, threads it into the first form (via ->), and when that result is not nil, through the next etc.
    
        (-> {:a 1} :b inc)
        ;; NullPointerException   clojure.lang.Numbers.ops (Numbers.java:942)
        (some-> {:a 1} :b inc)
        ;; nil

-   Return an expression, e.g.
    
        (defmacro ret-export-db [approach-fn conn path]
          `(~approach-fn @~conn ~path))
        (ret-export-db export-db-tc conn "/test/path.txt")
    
    Invoking `ret-export-db` as above evaluates to `(export-db-tc @conn "/test/path.txt")`


<a id="org96fbfb0"></a>

# Namespaces

-   Important namespaces: clojure.core, clojure.repl
-   Require functions that can be referred to unqualified:
    `(ns ns-name (:require [my.namespace :refer [compute other-fn]]))`
-   Force loading of all identified libs even if already loaded:
    `(require '[learn-cljs.import-fns.format :as fmt] :reload)`
-   Instruct REPL to operate in namespace:
    `(in-ns 'learn-cljs.import-fns.format)`
-   Current namespace: `*ns*`
-   Remove mapping for symbol from namespace:
    `(ns-unmap 'user 'base-config)` or `(ns-unmap *ns* 'base-config)`
-   Unalias namespace: `(ns-unalias *ns* 'm)`
-   Refresh all namespaces:
    
        (require '[clojure.tools.namespace.repl :refer [refresh]])
        (refresh)


<a id="org45b21e4"></a>

## Dependencies

-   require vs. use vs. import:
    
    -   require: load Clojure library
    -   use: same as require, and additionally refers to loaded definitions from current ns
    -   import: for Java classes and interfaces only, e.g.
    
        (import java.util.Date)
        (def *now* (Date.))
        ;; import multiple classes in a namespace
        (ns wanderung.datascript
          (:require [wanderung.core :as w :refer [create-source]])
          (:import [wanderung.core IConnection IExtract])

-   How to use local dependencies
    
    -   In dependency folder, run $ mvn install
    
    OR
    
    -   [Leiningen checkout dependencies](https://github.com/technomancy/leiningen/blob/master/doc/TUTORIAL.md#checkout-dependencies)
        Update local dependencies without running `lein install` and restarting REPL every time a change is made


<a id="orgf088053"></a>

# Destructuring


<a id="org93166e1"></a>

## Sequential destructuring

    (let [[a _ b _] '(1 2 3 4)] (println _))
    ;; => 4
    
    (def numbers [1 2 3 4 5])
    (let [[x & remaining :as all] numbers]
      (apply prn [remaining all]))
    ;; => (2 3 4 5) [1 2 3 4 5]
    
    (def my-line [[5 10] [10 20]])
    (let [[[a b :as group1] [c d :as group2]] my-line]
      (println a b group1)
      (println c d group2))
    ;; => 5 10 [5 10]
    ;; => 10 20 [10 20]


<a id="orga10b1a3"></a>

## Associative destructuring

    (def my-map {:a "A" :b "B" :c 3 :d 4})
    (let [{a :a, x :x, :or {x "Not found!"}, :as all} my-map]
      (println "I got" a "from" all)
      (println "Where is x?" x))
    ;= I got A from {:a "A" :b "B" :c 3 :d 4}
    ;= Where is x? Not found!
    
    (def client {:name "Super Co."
                 :location "Philadelphia"
                 :description "The worldwide leader in plastic tableware."})
    (let [{:keys [name location description]} client]
      (println name location "-" description))
    ;= Super Co. Philadelphia - The worldwide leader in plastic tableware.
    
    (def string-keys {"first-name" "Joe" "last-name" "Smith"})
    (let [{:strs [first-name last-name]} string-keys]
      (println first-name last-name))
    ;= Joe Smith
    
    (def symbol-keys {'first-name "Jane" 'last-name "Doe"})
    (let [{:syms [first-name last-name]} symbol-keys]
      (println first-name last-name))
    ;= Jane Doe
    
    (def multiplayer-game-state
      {:joe {:class "Ranger"
             :weapon "Longbow"
             :score 100}} ;...
      )
    (let [{{:keys [class weapon]} :joe} multiplayer-game-state]
      (println "Joe is a" class "wielding a" weapon))
    ; or
    (let [{:joe/keys [class weapon]} multiplayer-game-state]
      (println "Joe is a" class "wielding a" weapon))
    ;= Joe is a Ranger wielding a Longbow


<a id="orgf9c4309"></a>

### Keyword arguments

    (defn configure [val options]
      (let [{:keys [debug verbose] :or {debug false, verbose false}} options]
        (println "val =" val " debug =" debug " verbose =" verbose)))
    (configure 12 {:debug true})
    ;;val = 12  debug = true  verbose = false
    
    (defn configure [val & {:keys [debug verbose]
                            :or {debug false, verbose false}}]
      (println "val =" val " debug =" debug " verbose =" verbose))
    (configure 10)
    ;;val = 10  debug = false  verbose = false
    (configure 12 :verbose true :debug true)
    ;;val = 12  debug = true  verbose = true
    (configure 12 {:verbose true :debug true})
    ;;val = 12  debug = true  verbose = true
    (configure 12 :debug true {:verbose true})
    ;;val = 12  debug = true  verbose = true


<a id="org0dd58f2"></a>

### Namespaced keywords

    (def human {:person/name "Franklin"
                :person/age 25
                :hobby/hobbies "running"})
    (let [{:keys [hobby/hobbies]
           :person/keys [name age]
           :or {age 0}} human]
      (println name "is" age "and likes" hobbies))
    ;= Franklin is 25 and likes running
    
    (let [{:person/keys [age]
           hobby-hobbies :hobby/hobbies
           person-name :person/name} human]
      (println person-name "is" age "and likes" hobby-hobbies))
    ;= Franklin is 25 and likes running
    
    (require '[person :as p])
    (let [person {::p/name "Franklin", ::p/age 25}
          {:keys [::p/name ::p/age]} person]
      (println name "is" age))
    ;= Franklin is 25


<a id="org75bb3ca"></a>

### Nested maps

    (def john-smith {:f-name "John"
                     :l-name "Smith"
                     :phone "555-555-5555"
                     :address {:street "452 Lisp Ln."
                               :city "Macroville"
                               :state "Kentucky"
                               :zip "81321"}
                     :hobbies ["running" "hiking" "basketball"]
                     :company "Functional Industries"})
    (defn print-contact-info
      [{:keys [f-name l-name phone company]
        {:keys [street city state zip]} :address
        [fav-hobby second-hobby] :hobbies}] ...)


<a id="org96ef204"></a>

# Abstractions


<a id="orgbe1867c"></a>

## Multimethods

    (defmulti do-math (fn [operation x y] operation))
    (defmethod do-math :add [_ x y] (+ x y))
    (defmethod do-math :subtract [_ x y] (+ x y))
    (do-math :add 1 2)


<a id="orgbf64971"></a>

## Objects

-   Where required: `(require '[clojure.reflect :refer [reflect]])`
-   Show methods (and other information) for an object: `(reflect some-obj)`
    Note that only methods directly defined on the object&rsquo;s class are listed.
-   Show methods (including those inherited) on an object&rsquo;s class:
    `(.getMethods (.getClass (tc/rows some-obj :as-maps)))`


<a id="orgc6d12d0"></a>

# State

-   Set validator on atom creation, e.g.:

`(atom init-val 0 :validator validator-fn)`
`(atom init-val {:a 1} :validator validator-fn)`


<a id="org201dafd"></a>

# Reading clojure characters


<a id="org679d3cf"></a>

## Symbols and Vars

A Symbol (can) references a Var, which holds/represents a value (which may be a function).

-   Symbols

    (defn squared [x] (* x x))  ;; evaluates to a Var
    ;; => #'user/squared
    (def n 2)   ; evaluates to a Var
    ;; => #'user/n
    (type 'squared)
    ;; => clojure.lang.Symbol
    (type 'n)
    ;; => clojure.lang.Symbol
    (type 'random-symbol)
    ;; => clojure.lang.Symbol
    'squared
    ;; => squared
    (symbol "squared")  ;; takes string, keyword, or Var argument
    ;; => squared

-   Vars

    (resolve 'squared)
    ;; => #'user/squared
    (var squared)
    ;; => #'user/squared
    #'squared   ;; shorthand
    ;; => #'user/squared
    (type (var squared))
    ;; => clojure.lang.Var
    squared
    ;; => #function[user/squared]
    @(resolve 'squared)
    ;; => #function[user/squared]
    (type @(resolve 'squared))
    ;; => user$squared
    (@(resolve (symbol "squared")) 2)   ;; Deref (with @) the Var named by the Symbol squared
    ;; => 4
    ((resolve (symbol "squared")) 2)    ;; A Var asked to act as a function defers to its value
    ;; => 4

-   Symbol can be used as a function: it looks itself up in its argument
    `('foo {'foo 2})` is equivalent to `(get {'foo 2} 'foo)`

-   Gotcha: `(#' sym)`, `(var sym)`, and `(resolve sym)` are **not** equivalent!
    
        (defn deref-fn-sym [sym] @#'sym)
        ;; Syntax error compiling var at ... Unable to resolve var: sym in this context
        (defn deref-fn-sym [sym] @(var sym))
        ;; Syntax error compiling var at ... Unable to resolve var: sym in this context
        (defn deref-fn-sym [sym] @(resolve sym))
        ;; => #'user/deref-fn-sym


<a id="orga034dc0"></a>

## `^`: Metadata

-   A map of values that can be attached to various forms
-   Provides extra information; can be used for documentation, compilation warnings, typehints, and other features

    (def ^{:debug true} five 5) ;; or the next
    (def ^:debug five 5) ;; equivalent shorthand
    (meta #'five)
    ;; => {:ns #<Namespace user>, :name five, :column 1, :debug true, ...}
    
    ;; Metadata is attached to the form that follows it
    (def m ^:hi [1 2 3])
    (meta (var m))
    ;; => {:line 35, :column 8, ... :name m, :ns #namespace[user]}
    (meta m)
    ;; => {:hi true}
    
    ;; Type hint
    (def ^Integer five 5)
    (meta #'five)
    ;; => {:ns #<Namespace user>, ... :tag java.lang.Integer}


<a id="org6317e17"></a>

# IO

-   Read EDN file: `(clojure.edn/read (java.io.PushbackReader. (io/reader "test.edn")))`


<a id="orgb4d2c6c"></a>

# Vars and global environment

-   Temporarily redefine var (function in this example):

    (alter-var-root #'datahike.api/connect
                    (fn [f] (fn [& args] (println "instrumented") (apply f args))))


<a id="org51d4242"></a>

# Testing

    (require '[clojure.test :refer [run-tests test-vars]])
    (require 'foo.bar-test :reload-all)
    ; for good measure
    (dir foo.bar-test)
    (meta #'foo.bar-test/test1)
    ; run all tests
    (run-tests 'foo.bar-test)
    ; run one test
    (test-vars [#'foo.bar-test/test2])


<a id="org5a42f7a"></a>

## Kaocha

`bin/kaocha --focus test.ns`
`bin/kaocha --fail-fast`
`bin/kaocha --no-capture-output`
`bin/kaocha --skip test.ns1 --skip test.ns2`


<a id="org1dc5bee"></a>

# Web


<a id="org38e035d"></a>

## clj-http

-   debugging: add `:debug true` to request map

