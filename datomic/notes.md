
# Table of Contents

1.  [Datomic data model](#org3c5b3c9)
2.  [Basic queries](#orgebdc86e)
3.  [Data patterns](#org0896dd7)
4.  [Parameterised queries](#org2803bd3)
    1.  [Tuples](#orgb70ecd4)
    2.  [Collections](#org688507e)
    3.  [Relations](#orgb1f153b)
5.  [More queries](#org73364e3)
    1.  [Attributes](#org7052823)
    2.  [Transactions](#org477738e)
6.  [Predicates](#orgb03dbd6)
7.  [Transformation functions](#orga0799ce)
8.  [Aggregates](#org67b192d)
9.  [Rules](#org8594e0c)
10. [Identity and uniqueness](#org737daad)

These notes address the Datomic dialect of Datalog, based on the [Learn Datalog Today tutorial](http://www.learndatalogtoday.org/).


<a id="org3c5b3c9"></a>

# Datomic data model

Based on atomic facts called datoms. A **datom** is a 4-tuple consisting of:

-   Entity ID
-   Attribute
-   Value
-   Transaction ID

where each entity ID can be viewed as corresponding to a relational DB row, attributes defined on it as columns, and values as corresponding column values.

Note, however, that entities of the same &ldquo;type&rdquo; (semantically, i.e. as discernible by a human, e.g. two movies) are not constrained to share a schema; they could in principle have no attributes in common.

An entity can reference another entity by taking that entity&rsquo;s ID as its value.

![img](/Users/yiffle/Desktop/datoms.png)


<a id="orgebdc86e"></a>

# Basic queries

A query is represented as a vector starting with the keyword `:find` followed by one or more **pattern variables** (symbols starting with `?`, e.g. `?title`). Next is the `:where` clause giving data patterns to match.

E.g. find all entity IDs having `:person/name` Ridley Scott:

    [:find ?e
     :where
     [?e :person/name "Ridley Scott"]]

A **data pattern** is a datom with some parts replaced by pattern variables.

A `_` can be used as a wildcard for the parts of the data pattern that are ignored, e.g. this is equivalent to the previous query:

    [:find ?e
     :where
     [?e :person/name "Ridley Scott" _]]


<a id="org0896dd7"></a>

# Data patterns

A :where clause can contain multiple data patterns; their order doesn&rsquo;t matter (outside of performance considerations). E.g. to find titles of movies made in 1987:

    [:find ?title
    :where
    [?e :movie/year 1987]
    [?e :movie/title ?title]]

E.g. to find out the cast of Lethal Weapon:

    [:find ?name
     :where
     [?e :movie/title "Lethal Weapon"]
     [?e :movie/cast ?p]
     [?p :person/name ?name]]


<a id="org2803bd3"></a>

# Parameterised queries

E.g. to find movies Sylvester Stallone was in:

    (def query [:find ?title
                :in $ ?name
                :where
                [?p :person/name ?name]
                [?e :movie/cast ?p]
                [?e :movie/title ?title]])
    (q query db "Sylvester Stallone")

This query takes two arguments: $ for the database itself (implicit if :in clause unspecified), and ?name. $ is implicit in each data pattern, which is actually a 5-tuple of the form:
[<database> <entity-id> <attribute> <value> <transaction-id>]

The input pattern variable ?name is the simplest kind of binding, assigning the (scalar?) input value directly to the variable. There are 4 binding forms, respectively destructuring input data into: scalars, tuples, collections, and relations.


<a id="orgb70ecd4"></a>

## Tuples

Form (assuming two items): [?a ?b]
E.g. to find movies where a director and actor collaborated:

    (def query [:find ?title
                :in $ [?director ?actor]
                :where
                [?d :person/name ?director]
                [?a :person/name ?actor]
                [?m :movie/director ?d]
                [?m :movie/cast ?a]
                [?m :movie/title ?title]])
    (q query db ["James Cameron" "Arnold Schwarzenegger"])


<a id="org688507e"></a>

## Collections

Form (again assuming two items): [?a &#x2026;]
E.g. to find movies directed by either James Cameron or Ridley Scott:

    (def query [:find ?title
                :in $ [?director ...]
                :where
                [?p :person/name ?director]
                [?m :movie/director ?p]
                [?m :movie/title ?title]])
    (q query db ["James Cameron" "Ridley Scott"])


<a id="orgb1f153b"></a>

## Relations

Form (again assuming two items): [​[?a ?b]]
Relations, i.e. a set of tuples, can be used to join external relations with datoms in your DB, or ask &ldquo;or&rdquo; questions involving multiple variables.

E.g. 1, to find box office earnings for a certain director using a relation with tuples [movie-title box-office-earnings]:

    [:find ?title ?box-office
     :in $ ?director [[?title ?box-office]]
     :where
     [?p :person/name ?director]
     [?m :movie/director ?p]
     [?m :movie/title ?title]]

E.g. 2, what releases are associated with either John Lennon&rsquo;s Mind Games or Paul McCartney&rsquo;s Ram:

    (def query [:find ?release
                :in $ [[?artist-name ?release-name]]
                :where [?artist :artist/name ?artist-name]
                [?release :release/artists ?artist]
                [?release :release/name ?release-name]])
    (q query db [["John Lennon" "Mind Games"]
                 ["Paul McCartney" "Ram"]])


<a id="org73364e3"></a>

# More queries

Attributes and transactions can also be queried like values and entity IDs.


<a id="org7052823"></a>

## Attributes

To find all attributes associated with person entities, given that :person/name is one such attribute:

    [:find ?attr
     :where
     [?p :person/name]
     [?p attr]]

The above returns entity IDs of the attributes. To get the corresponding keywords:

    [:find ?attr
     :where
     [?p :person/name]
     [?p ?a]
     [?a :db/ident ?attr]]

I.e. attributes are also database entities!


<a id="org477738e"></a>

## Transactions

Also possible to query transactions, e.g.:

-   When was a fact asserted?
-   When was a fact retracted?
-   Which facts were part of a transaction?

The only attribute associated with a transaction (by default) is `:db/txInstant`.

Example query:

    [:find ?timestamp
     :where
     [?p :person/name "James Cameron" ?tx]
     [?tx :db/txInstant ?timestamp]]


<a id="orgb03dbd6"></a>

# Predicates

So far, we&rsquo;ve only been dealing with data patterns. We haven&rsquo;t yet seen a proper way of handling questions like &ldquo;*Find all movies released before 1984*&rdquo;. To do that, we need **predicate clauses**, e.g.:

    [:find ?title
     :where
     [?m :movie/title ?title]
     [?m :movie/year ?year]
     [(< ?year 1984)]]

The predicate clause filters the result set to include only those for which it returns a truthy value. Any Clojure function or Java method can be used as a predicate function, e.g.:

    [:find ?name
     :where
     [?p :person/name ?name]
     [(.startsWith ?name "M")]]

Clojure functions must be fully namespace-qualified, so if you have defined your own predicate `awesome?` must be written e.g. `(my.namespace/awesome? ?movie)`. Some ubiquitous predicates can be used without namespace qualification, e.g. `<`, `>`, `=`, `not=`, etc.


<a id="orga0799ce"></a>

# Transformation functions

Pure (i.e. side-effect-free) functions or methods used in queries to transform values and bind their results to pattern variables.

E.g., to compute age from an attribute `:person/born` with type `:db.type/instant`:

    (defn age [birthday today]
      (quot (- (.getTime today)
               (.getTime birthday))
            (* 1000 60 60 24 365)))

That can be used to compute a person&rsquo;s age inside the query itself:

    [:find ?age
     :in $ ?name ?today
     :where
     [?p :person/name ?name]
     [?p :person/born ?born]
     [(tutorial.fns/age ?born ?today) ?age]]

A transformation function clause has the shape `[(fn arg1 arg2 ...) result-binding]` where the binding can take any of the 4 forms.

Note that transformation functions cannot be nested, e.g. instead of `[(f (g ?x)) ?a]`, intermediate results must be bound to temporary pattern variables:

    [(g ?x) ?t]
    [(f ?t) ?a]


<a id="org67b192d"></a>

# Aggregates

Aggregate functions e.g. `sum`, `max`, etc. are written in the `:find` clause:

    [:find (max ?date)
     :where
     ...]

An aggregate function collects values from multiple datoms and returns

-   A single value: `min`, `max`, `sum`, `avg`, etc. or
-   A collection of values, e.g.: `(min n ?d)`, `(sample n ?e)` etc. where `n` specifies the size of the collection.


<a id="org8594e0c"></a>

# Rules

Rules abstract away reusable parts of queries, e.g.:

    [(actor-movie ?name ?title)
     [?p :person/name ?name]
     [?m :movie/cast ?p]
     [?m :movie/title ?title]]

The first vector is called the *head* of the rule, where the first symbol is the name of the rule. The rest of the rule is called the *body*. The head is conventionally enclosed in `(...)` as a visual aid.

A rule can be thought of as a kind of function, but since this is logic programming, pattern variables in the head can be used for input as well as output, e.g. finding movies given an actor, or finding actors given a movie, or getting all combinations of movies and actors.

To use the example rule above:

    [:find ?name
     :in $ %
     :where
     (actor-movie ?name "The Terminator")]

`%` in the `:in` clause represents the rules. Any number of rules can be written, collected in a vector, and passed to the query engine like any other input:

    [[(rule-a ?a ?b)
      ...]
     [(rule-a ?a ?b)
      ...]
     ...]

Data patterns, predicates, transformation functions, and calls to other rules can be used in the body of a rule.

Rules can also be used as another tool to write logical OR queries, as the same rule name can be used several times, with subsequent rule definitions used only if preceding ones aren&rsquo;t satisfied:

    [[(associated-with? ?person ?movie)
      [?movie :movie/cast ?person]
     [(associated-with? ?person ?movie)
      [?movie :movie/director ?person]]]

This example can be used to find both directors and cast members very easily:

    [:find ?name
     :in $ %
     :where
     [?m :movie/title "Predator"]
     (associated-with ?p ?m)
     [?p :person/name ?name]]

More examples:

1.  Two people are friends if they have worked together in a movie.

    [[(friends ?p1 ?p2)
      [?m :movie/cast ?p1]
      [?m :movie/cast ?p2]
      [(not= ?p1 ?p2)]]
     [(friends ?p1 ?p2) [?m :movie/director ?p1] [?m :movie/cast ?p2]]
     [(friends ?p1 ?p2) (friends ?p2 ?p1)]
    
    (def query [:find ?friend
                :in $ % ?name
                :where
                [?p1 :person/name ?name]
                (friends ?p1 ?p2)
                [?p2 :person/name ?friend]])
    (q query db "Sigourney Weaver")

1.  Find sequels, where a movie m2 is a sequel of m1 if it&rsquo;s a direct sequel, or the sequel of a sequel m to m1.

    [[(sequel ?m1 ?m2) [:m1 :movie/sequel ?m2]]
     [(sequel ?m1 ?m2) [:m :movie/sequel ?m2] (sequel ?m1 ?m)]]
    
    (q [:find ?sequel
        :in $ % ?title
        :where
        [?m :movie/title ?title]
        (sequel ?m ?s)
        [?s :movie/title ?sequel]] db "Max Max")


<a id="org737daad"></a>

# Identity and uniqueness

Available ways to model identity and uniqueness:

-   **Entity IDs**: Every entity has one. Assigned by the transactor, and never change.
    -   Can request new entity ID by specifying a temporary ID in transaction data.
-   **Idents**: Associate a programmatic name (a keyword) with an entity ID via a value for the `:db/ident` attribute.
    -   Should be used for two purposes: to name schema entities and implement enumerated tags.
        -   Should not be used for ordinary domain entities or test data.
-   **Unique identities**: DB-wide unique identifier. Per-attribute, with `:db/unique` set to `:db.unique/identity`.
    -   **Upsert** (update or insert) behaviour: if a transaction specifies a unique identity that already exists in the DB for a tempid, it resolves to the existing entity.
    -   Always indexed by value. Specifying `:db/index true` is redundant and not recommended.
    -   Only `:db.cardinality/one` attributes can be unique.
        -   **Composite tuples** can be used to declare composite uniqueness constraints.
-   **Unique values**: Like unique identities, except:
    -   Specified through an attribute with `:db/unique` set to `:db.unique/value`.
    -   Attempts to assert a new tempid with a unique value already in the DB will cause an `IllegalStateException`.
-   **Squuids**: UUIDs are domain-external globally unique identifier for an entity, specified through attribute with value type `:db.type/uuid`. Datomic **semi-sequential UUIDs** are used instead of alternatives that can be particularly difficult to index; this isn&rsquo;t strictly necessary with Datomic indexes, but should still be preferred for UUIDs that may be used elsewhere.
-   **Lookup refs**: Identify an entity with the given value for a unique-value or -identity attribute.
    -   In transactions, e.g. to assert that the entity with given email loves pizza:
        `{:db/id [:person/email "joe@examples.com"]
              :person/loves :pizza}`
    -   Cannot be used in query body, but can be used as inputs in a parameterized query.

Note that *joins should use entity IDs* for efficiency.

