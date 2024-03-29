
# Table of Contents

1.  [Links](#org5bea079)
2.  [Introduction](#orga9f6243)
3.  [Usage](#org5917e60)
4.  [Relationship to Datomic and DataScript](#org55ce5e8)
5.  [ClojureScript support](#org088b57f)
6.  [Syntax](#org01ddbcd)
    1.  [Datomic syntax](#org972a0e9)
    2.  [Invoice example](#org4619bfe)
    3.  [Etc.](#orgfb3b0b2)
7.  [Internals](#org4b58463)
    1.  [Hitchhiker tree](#org5c88b2e)



<a id="org5bea079"></a>

# Links

[API documentation](https://cljdoc.org/d/io.replikativ/datahike/0.3.6/doc/readme)
[GitHub repository](https://github.com/replikativ/datahike)


<a id="orga9f6243"></a>

# Introduction

-   Durable Datalog DB powered by efficient Datalog query engine.
-   Started as a port of [DataScript](https://github.com/tonsky/DataScript) to the [hitchhiker-tree](https://github.com/datacrypt-project/hitchhiker-tree).
-   Building on the two projects and storage backends for the hitchhiker-tree through [konserve](https://github.com/replikativ/konserve).


<a id="org5917e60"></a>

# Usage

-   Add to dependencies: io.replikativ/datahike &ldquo;0.3.6&rdquo;.
-   Small stable API for the JVM.
    -   On-disk schema not fixed yet; migration guide provided meanwhile.
-   The API namespace provides compatibility to a subset of Datomic functionality.
    -   Should work as a drop-in replacement on the JVM.
-   The rest of Datahike will be ported to core.async for platform-neutral IO coordination.


<a id="org55ce5e8"></a>

# Relationship to Datomic and DataScript

-   Part of the [replikativ](https://github.com/replikativ) toolbox for building distributed data management solutions
    -   Goal is not to provide an open-source reimplementation of Datomic, though it can be used as drop-in replacement for a subset of Datomic functionality.
-   Some differences with Datomic:
    -   Datahike runs locally on one peer.
        -   A transactor might be forthcoming.
            -   Can also be realized through any linearizing write mechanism, e.g. Apache Kafka.
    -   Datahike provides the DB as a transparent value, i.e. the index datastructures (hitchhiker-tree) are directly accessible.
        -   Their persistence can be leveraged for replication.
        -   Note: These internals are not guaranteed to stay stable.
    -   Datomic has a REST interface and a Java API.
    -   Datomic provides timeouts.
-   Datahike&rsquo;s query engine and most of its codebase come from [DataScript](https://github.com/tonsky/DataScript).
    -   Differences with Datomic in the query engine are documented there.


<a id="org088b57f"></a>

# ClojureScript support

-   Work in progress.


<a id="org01ddbcd"></a>

# Syntax


<a id="org972a0e9"></a>

## Datomic syntax

The syntax of the Datomic dialect of Datalog is virtually identical.
[Tutorial](http://www.learndatalogtoday.org/): [Notes](../datomic/notes.md)


<a id="org4619bfe"></a>

## [Invoice example](https://gitlab.com/replikativ/datahike-invoice/-/blob/master/src/datahike_invoice/core.clj)

    (ns datahike-invoice.core
      (:require [datahike.api :as d] ...))
    
    (d/q '[:find ?e ?on
           :where
           [?e :customer/name ?on]]
         @conn)
    ;; => #{[24 "Little Shop"] [23 "FunkyHub Startup"]}
    
    (d/q '[:find ?on
           :where
           [:customer/name ?on]]
         @conn)
    ;; => #{[:db/ident] [:db/cardinality] [:db/unique] [:db/valueType]}
    
    (d/q '[:find ?e ?on
           :where
           [?e :customer/street ?on]]
         @conn)
    ;; => #{[23 "Huvudvägen 1"] [24 "Marktplatz 10"]}
    
    (d/q '[:find [?e ?on]
           :where
           [?e :customer/street ?on]]
         @conn)
    ;; => [23 "Huvudvägen 1"]
    
    (d/q '[:find ?or .
           :where
           [?e :offer/number ?or]]
         @conn)
    ;; => "20211108-01"
    
    (d/q '[:find [(pull ?tg [*]) ...]
           :where
           [?tg :task-group/name ?tgn]]
         @conn)
    ;; => [{:db/id 28,
    ;;  :task-group/name "Fresh Produce Shop",
    ;;  :task-group/tasks [#:db{:id 26} #:db{:id 27}]}]
    
    (d/q '[:find [(pull ?e [*]) ...] :where [?e]] @conn)
    ;; => [#:db{:id 4,
    ;;      :cardinality :db.cardinality/one,
    ;;      :ident :customer/street,
    ;;      :valueType :db.type/string}
    ;; #:db{:id 536870918, :txInstant #inst "2021-11-10T17:55:06.092-00:00"}
    ;; #:db{:id 536870916, :txInstant #inst "2021-11-09T04:28:42.856-00:00"}
    ;; ...
    ;;  {:db/id 28,
    ;;   :task-group/name "Fresh Produce Shop",
    ;;   :task-group/tasks [#:db{:id 26} #:db{:id 27}]}
    ;;  #:db{:id 6,
    ;;       :cardinality :db.cardinality/one,
    ;;       :ident :customer/city,
    ;;       :valueType :db.type/string}
    ;; ...]

Note the transparently different return types for the following:

    (def customer-all (d/pull @conn '[*] [:customer/name "Little Shop"]))
    ;; => #'datahike-invoice.core/customer-all
    customer-all
    ;; => {:customer/postal "23455",
    ;; :customer/department "",
    ;; ...
    ;; :db/id 24,
    ;; ...
    ;; :customer/offers [#:db{:id 29} #:db{:id 36}],
    ;; ...}
    (type customer-all)
    ;; => clojure.lang.PersistentHashMap
    (keys customer-all)
    ;; => (:customer/postal
    ;; ...)
    
    (def customer-specs (d/pull @conn '[:customer/name :customer/country]
      [:customer/name "Little Shop"]))
    ;; => #'datahike-invoice.core/customer-specs
    customer-specs
    ;; => #:customer{:name "Little Shop", :country "Germany"}
    (type customer-specs)
    ;; => clojure.lang.PersistentArrayMap
    (keys customer-specs)
    ;; => (:customer/name :customer/country)

Pulling specified nested return value (collection):

    (d/pull @conn '[* {:customer/offers
                       [:offer/name :offer/advisor
                        {:offer/task-groups
                         [:task-group/name
                          {:task-group/tasks [{:task/price-unit [*]
                                               :task/effort-unit [:db/ident]}]}]}]}]
          [:customer/name "Little Shop"])
    ;; => #:customer{:name "Little Shop",
    ;;           ...
    ;;           :offers
    ;;           [#:offer{:name "Adjustments App Q2 2019",
    ;;                    :advisor "Konrad Kuehne",
    ;;                    :task-groups
    ;;                    [#:task-group{:name "Fresh Produce Shop",
    ;;                                  :tasks
    ;;                                  [#:task{:price-unit
    ;;                                          #:db{:id 15, :ident :euro},
    ;;                                          :effort-unit #:db{:ident :hour}}
    ;;                                   #:task{:price-unit
    ;;                                          #:db{:id 15, :ident :euro},
    ;;                                          :effort-unit #:db{:ident :hour}}]}]}
    ;;            #:offer{...}]}

Using returned collections:

    (def customer-offers (d/pull @conn '[{:customer/offers [:offer/number :offer/name]}]
                                 [:customer/name "Little Shop"]))
    ;; => #'datahike-invoice.core/customer-offers
    customer-offers
    ;; => #:customer{:offers
    ;;           [#:offer{:number "20211110-01", :name "Adjustments App Q2 2019"}
    ;;            #:offer{:number "20211112-01", :name "Adjustments App Q2 2019"}]}
    (:customer/offers customer-offers)
    ;; => [#:offer{:number "20211110-01", :name "Adjustments App Q2 2019"}
    ;; #:offer{:number "20211112-01", :name "Adjustments App Q2 2019"}]

Selecting succinctly&#x2013;the following are equivalent:

    (d/q '[:find (pull ?e [*]) .
           :in $ ?offer-id
           :where
           [?e :customer/offers ?offer-id]]
         @conn [:offer/number offer-num])
    
    (d/q '[:find (pull ?e [*]) .
           :in $ ?offer-number
           :where
           [?e :customer/offers ?offer-id]
           [?offer-id :offer/number ?offer-number]]
         @conn offer-num)

Selecting / Joining across multiple databases:

    (def effort-uri "datahike:file:///tmp/effort-store")
    (d/delete-database effort-uri)
    (d/create-database effort-uri :schema-on-read true :temporal-index false)
    (def effort-conn (d/connect effort-uri))
    (d/transact effort-conn [{:effort/start-date (java.util.Date.)
                              :effort/task 30}
                             {:effort/start-date (java.util.Date.)
                              :effort/task 31}])
    (d/q '[:find ?tn ?sd
           :in $ $effort
           :where
           [$ ?t :task/name ?tn]
           [$effort ?e :effort/task ?t]
           [$effort ?e :effort/start-date ?sd]]
         @conn
         @effort-conn)

Retracting:

    (d/transact conn [[:db/retractEntity 26]])
    (d/transact btc/conn [[:db/retract eid :topic/parent {:topic/name parent}]])


<a id="orgfb3b0b2"></a>

## Etc.

Add / Change attribute for existing entity

    ;; Note: in this example and the next, :topic/name is a unique identity attribute
    (d/transact conn [{:db/id [:topic/name sub]
                       :topic/parent [:topic/name parent]}])
    
    ;; Using reverse attribute name
    (d/transact conn [{:topic/_urls [:topic/name topic]
                       :url/url url}])
    ;; the following are equivalent
    (transact conn [{:db/id  -1
                     :name   "Oleg"
                     :_friend 296}])
    (transact conn [[:db/add  -1 :name   "Oleg"]
                    [:db/add 296 :friend -1]])


<a id="org4b58463"></a>

# Internals


<a id="org5c88b2e"></a>

## Hitchhiker tree

-   Lazy inserts (WIP?)
-   Interning: instead of storing attributes as keywords, map them to integers and use those instead references&#x2014;more efficient internal mapping for constants.

