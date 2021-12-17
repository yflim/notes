
# Table of Contents

1.  [Special forms](#org1add9a7)
    1.  [`do`: Evaluates expressions in order and returns value of the last](#orgec3c981)
2.  [Laziness](#org16e1f2f)



<a id="org1add9a7"></a>

# Special forms


<a id="orgec3c981"></a>

## `do`: Evaluates expressions in order and returns value of the last

-   Necessary where its omission would result in attempted invocation of expression that isn&rsquo;t function, e.g.
    
        (map #(do (println "x: " %) %) '(0 1))
        
        ;; note that it's safe with if and else clauses swapped since else isn't evaluated
        (if (> 2 1)
          ((println "next")
           (str "ret" 1))
          (println "sthg"))
        
        ;; Evaluates to nil
        (do (time (range 10)) nil)

-   Not necessary in e.g. `let` and `fn` (and `defn`) that won&rsquo;t result in illegal attempted invocation, and apparently have an implicit `do`:

    (let []
      (println "sthg")
      (println "next"))
    ((fn []
       (println "sthg")
       (println "next")
       (str "ret" 1)))


<a id="org16e1f2f"></a>

# Laziness

-   Note that lazy expressions can seem otherwise in `comment` block or in REPL because they are consumed by REPL
    
        ;; In code: lazy sequence. If consumed by REPL: Prints 1 to 100; evals to sequence of 100 nils.
        (map println (range 100))
        ;; Prints 0 to 31 (or other) from evaluation of a single chunk; evals to nil (println value).
        (nth (map println (range 100)) 1)

-   Gotchas: see [implementation details underlying the difference on StackOverflow](https://stackoverflow.com/questions/39957310/map-not-quite-lazy)
    
        ;; (All the following evaluate to 0)
        ;; prints 0 to 31
        (first (map #(do (println "x: " %) %) (range 40)))
        (first (map #(do (println "x: " %) %) (vec (range 40))))
        (first (map #(do (println "x: " %) %) (doall (range 40))))
        ;; prints 0
        (first (map #(do (println "x: " %) %) (apply list (range 40))))

