
# Table of Contents

1.  [Tools (CLI)](#orgf7093ff)
2.  [Basics / Miscellanea](#orgcc187cc)
    1.  [Help](#org1a47aec)
3.  [Syntax](#orgc604309)
4.  [Strings](#org3ec76c4)
5.  [Collections](#org75c0501)
    1.  [Sequences](#orgb4d9d5b)
    2.  [Sets](#orgbf0c471)
    3.  [walk, pre-walk, post-walk](#org6bb9f59)
6.  [Transients](#org3998423)
7.  [Functions beget functions](#orgc7793a1)
8.  [Destructuring](#orgbfe10bb)
    1.  [Sequential destructuring](#org5984819)
    2.  [Associative destructuring](#orga11ee37)
        1.  [Keyword arguments](#orged81feb)
        2.  [Namespaced keywords](#org5f99eef)
        3.  [Nested maps](#org0da3cd2)
9.  [Doing](#org9b61a2a)
10. [Branching](#org252ca59)
11. [Reading clojure characters](#org403cc26)
    1.  [Symbols and Vars](#org919fdb9)
    2.  [`^`: Metadata](#orgf51e458)
12. [Built-in macros](#org486ad33)
13. [Namespaces](#org0ad318e)
    1.  [Dependencies](#org82f4e41)
14. [Java interop](#org428fe55)
15. [Abstractions](#org9e48066)
    1.  [Multimethods](#orgdb023f6)
    2.  [Objects](#orgb4375ae)
    3.  [Protocols](#orgf75e299)
    4.  [Polymorphism](#orge434497)
16. [State](#orga908893)
17. [IO](#org7cbdefc)
18. [Vars and global environment](#org28020dd)
19. [Testing](#orgf5abc0d)
    1.  [Kaocha](#orga6cebe4)
20. [Builds and dependencies](#orgd16f2f5)
21. [Performance](#org53c27c4)
22. [Web](#orgb52e294)
    1.  [clj-http](#orgf4cb69f)
23. [Docstrings](#org0b34b61)



<a id="orgf7093ff"></a>

# Tools (CLI)

-   Find dependency versions:

`clj -X:deps find-versions :lib clojure.java-time/clojure.java-time`

-   Write dependency expansion to `trace.edn`: `clj -Strace`
-   Run build function, e.g. `clj -T:build install`
-   Create new library: `clojure -Tclj-new lib :name myname/mylib`


<a id="orgcc187cc"></a>

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
    
    -   `namespace`: Returns the namespace string of a symbol or keyword

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
    -   Gotcha: this seems subject to some subtleties that produce apparent inconsistencies (see below)

    (identical? [1] [1])
    ;; => false
    (= [1] [1])
    ;; => true
    (= #{1} '(1))
    ;; => false
    (= #{1} [1])
    ;; => false
    (= [1] '(1))
    ;; => true

-   Value of last evaluated expression: `*1`. Analogously, `*2` and `*3` (no higher).

-   Shell commands:

    (require '[clojure.java.shell :as sh])
    (sh/sh "ls" "-aul")


<a id="org1a47aec"></a>

## Help

-   Print public vars (including functions) in a namespace: `(dir namespace)`
-   All namespaces: `(all-ns)`
-   All public definitions in all currently-loaded namespaces matching regex or stringable thing: `(apropos str-or-pattern)`
-   View source code for `sym`: `(source sym)`


<a id="orgc604309"></a>

# Syntax

-   `..`
    E.g.: `(.. System (getProperties) (get "os.name"))`
    expands to `(. (. System (getProperties)) (get "os.name"))`


<a id="org3ec76c4"></a>

# Strings

-   `re-quote-replacement`: escape special characters in literal replacement for a pattern match in `replace` or `replace-first`
    
        (require '[clojure.string :as s])
        (def s "May 2018, June 2019")
        (s/replace s #"May|June" "10$") ;; IllegalArgumentException
        (s/replace s #"May|June" (s/re-quote-replacement "10$ in"))


<a id="org75c0501"></a>

# Collections

-   `find`: returns map entry for given key, or `nil` if absent

    (find {:a 1} :a)
    ;; => [:a 1]
    (val (find {:a 1} :a))
    ;; => 1
    (type (find {:a 1} :a))
    ;; => clojure.lang.MapEntry
    (key (find {:a 1} :a))
    ;; => :a

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


<a id="orgb4d9d5b"></a>

## Sequences

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

-   `map-indexed`


<a id="orgbf0c471"></a>

## Sets

`select`: `(select pred xset)`


<a id="org6bb9f59"></a>

## TODO walk, pre-walk, post-walk


<a id="org3998423"></a>

# Transients

-   Vectors, hash-maps, and hash-sets supported
-   `(transient ds)`: O(1) creation from persistent data structures
-   Shares structure with persistent source
-   Interim return values must be returned
-   `(persistent! ds)`: O(1) creation of persistent data structure when done building results
    -   Transient must not be used after: all operations will throw exceptions.
-   Same code structure as functional version: `conj!`, `pop!`, `assoc!`, `dissoc!`, `disj!`


<a id="orgc7793a1"></a>

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


<a id="orgbfe10bb"></a>

# Destructuring


<a id="org5984819"></a>

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


<a id="orga11ee37"></a>

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


<a id="orged81feb"></a>

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


<a id="org5f99eef"></a>

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


<a id="org0da3cd2"></a>

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


<a id="org9b61a2a"></a>

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

-   `doall`: Like `dorun` except the sequence head is retained and returned; entire seq kept in memory. E.g.:
    `(doall (map println [1 2 3]))`
    1
    2
    3
    ;; => (nil nil nil)


<a id="org252ca59"></a>

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


<a id="org403cc26"></a>

# Reading clojure characters


<a id="org919fdb9"></a>

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

-   Careful: `#'sym`, `(var sym)`, and `(resolve sym)` are **not** equivalent!
    
        (defn deref-fn-sym [sym] @#'sym)
        ;; Syntax error compiling var at ... Unable to resolve var: sym in this context
        (defn deref-fn-sym [sym] @(var sym))
        ;; Syntax error compiling var at ... Unable to resolve var: sym in this context
        (defn deref-fn-sym [sym] @(resolve sym))
        ;; => #'user/deref-fn-sym


<a id="orgf51e458"></a>

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


<a id="org486ad33"></a>

# Built-in macros

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

-   `doto`: `(doto x & forms)`
    Evaluates `x` then calls all the methods and functions invoked in `forms` with the value of `x` supplied before the given arguments. Returns `x`. E.g.:
    `(doto (new java.util.HashMap) (.put "a" 1) (.put "b" 2))`


<a id="org0ad318e"></a>

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


<a id="org82f4e41"></a>

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


<a id="org428fe55"></a>

# Java interop

    (.toUpperCase "fred")
    ;; => "FRED"
    (.getName String)   ; Note: .getName invoked on java.lang.Class
    ;; => "java.lang.String"
    (. java.time.Instant now)
    ;; => #object[java.time.Instant 0xbc379f "2023-05-12T03:54:22.697416Z"]
    (.-x (java.awt.Point. 1 2))
    ;; => 1
    (java.lang.Class/forName "Fully.Qualified.Class.Name")


<a id="org9e48066"></a>

# Abstractions


<a id="orgdb023f6"></a>

## Multimethods

    (defmulti do-math (fn [operation x y] operation))
    (defmethod do-math :add [_ x y] (+ x y))
    (defmethod do-math :subtract [_ x y] (+ x y))
    (do-math :add 1 2)


<a id="orgb4375ae"></a>

## Objects

-   Where required: `(require '[clojure.reflect :refer [reflect]])`
-   Show methods (and other information) for an object: `(reflect some-obj)`
    Note that only methods directly defined on the object&rsquo;s class are listed.
-   Show methods (including those inherited) on an object&rsquo;s class:
    `(.getMethods (.getClass (tc/rows some-obj :as-maps)))`


<a id="orgf75e299"></a>

## Protocols


<a id="orge434497"></a>

## Polymorphism

-   Get supertypes (both classes and interfaces) of a given type, e.g.
    `(supers clojure.lang.PersistentList$EmptyList)`
    `(supers clojure.lang.PersistentList)`
-   Test whether `x` is an instance of class `c`: `(instance? c x)`
-   More general than `instance?`: `(isa? child parent)` or `(isa? hierarchy child parent)`
    -   Allows relationship established via `derive` as well as Java type inheritance
-   `derive`: `(derive tag parent)` or `(derive hierarchy tag parent)`
    where `parent` must be a namespace-qualified symbol or keyword; `child` can be those or a class.
    E.g.: `(derive ::milk  ::dairy)`
-   More general version of `supers`: `parents`, which returns `derive` relationships as well
-   Analogue of `parents`: `descendants`


<a id="orga908893"></a>

# State

-   Set validator on atom creation, e.g.:

`(atom init-val 0 :validator validator-fn)`
`(atom init-val {:a 1} :validator validator-fn)`


<a id="org7cbdefc"></a>

# IO

-   Reading and writing line by line:
    
        (with-open [rdr (io/reader (io/file data-dir "agencies.csv"))
                    w (io/writer (io/file data-dir "agencies-cp.csv") :append true)]
          (let [rows (line-seq rdr)
                header (->> (string/split (first rows) #",")
                            (map #(str "agency/" %))
                            (string/join ","))]
            (.write w (str header "\n"))
            (doseq [row (rest rows)]
              (.write w (str row "\n")))))
-   Read EDN file: `(clojure.edn/read (java.io.PushbackReader. (io/reader "test.edn")))`


<a id="org28020dd"></a>

# Vars and global environment

-   Temporarily redefine var (function in this example):

    (alter-var-root #'datahike.api/connect
                    (fn [f] (fn [& args] (println "instrumented") (apply f args))))


<a id="orgf5abc0d"></a>

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


<a id="orga6cebe4"></a>

## Kaocha

`bin/kaocha --focus test.ns`
`bin/kaocha --fail-fast`
`bin/kaocha --no-capture-output`
`bin/kaocha --skip test.ns1 --skip test.ns2`


<a id="orgd16f2f5"></a>

# Builds and dependencies

-   Show contents of jar: `jar tf jar-file`


<a id="org53c27c4"></a>

# Performance

-   Benchmarking:
    -   Quick benchmark with printed performance summary
        
            (criterium/with-progress-reporting
             (criterium/quick-bench (nth ^longs (into [] (range 10000)) 0)))
        
        -   `quick-benchmark` or `benchmark` to record results
-   Profiling: VisualVM sampler tab
-   Type hinting:
    -   For arrays, e.g.:
        
        -   `(alength ^longs arr)` and `(alength ^"[J" arr)`
        -   `(aget ^"[Ljava.lang.Object;" objs-1 0)`
        
        where `arr` is an array of longs and
        
            (type (long-array 5))
            ;; => [J
            (type (object-array 5))
            ;; => [Ljava.lang.Object;
            (type ints-1)
            ;; => [I
            (aset ints-2 0 (aget ^"[I" ints-1 0))
    -   In threading macro, just put parenthese around hinted expression:
        
            (fn [^java.nio.file.WatchEvent e]
                 (-> e ^java.nio.file.Path (.context) .toAbsolutePath))


<a id="orgb52e294"></a>

# Web


<a id="orgf4cb69f"></a>

## clj-http

-   debugging: add `:debug true` to request map


<a id="org0b34b61"></a>

# Docstrings

-   Backtick-quote function arguments and special keywords, e.g.
    &ldquo;Adds \`x\` to the transient collection&rdquo;
-   Link to other functions, e.g. &ldquo;See also $$\[listen!$$\\]&rdquo; (minus escape backslashes)
-   Include small examples, e.g. (showing only skeletal code snippet and omitting rest of docstring):
    &ldquo;\`\`\`clojure
    (def app &#x2026;)
    \`\`\`&rdquo;
-   Use tables to describe complex options maps:
    
    <table border="2" cellspacing="0" cellpadding="6" rules="groups" frame="hsides">
    
    
    <colgroup>
    <col  class="org-left" />
    
    <col  class="org-left" />
    </colgroup>
    <thead>
    <tr>
    <th scope="col" class="org-left">key</th>
    <th scope="col" class="org-left">description</th>
    </tr>
    </thead>
    
    <tbody>
    <tr>
    <td class="org-left">&#x2026;</td>
    <td class="org-left">&#x2026;</td>
    </tr>
    
    
    <tr>
    <td class="org-left">`:router`</td>
    <td class="org-left">Function of &#x2026;</td>
    </tr>
    </tbody>
    </table>

