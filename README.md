## TLDR;
The repository is a public Logseq graph with the described use case for upcoming birthdays query.

## Context

I use Logseq to organize my life, and I have a personal CRM on it. Every page starting with `@` is a person. For example: `[[@Barack Obama]]`. On those pages, I use the `CRM template` below:

```
- CRM
  template-including-parent:: false
  template:: CRM
    - Birthday:: 
    Birth-location:: 
    Location::
    Relationship:: 
    Industry::
    Vip::
    Last-connected::
    Email::
    Phone::
    LinkedIn::
    Instagram::
    Source::
    - ## Background
        - ### 👪 Family
        - ### 🏢 Company
        - ### ♥️️ Likes and dislikes
        - ### 📝 Notes
```

A person's page should be like this:

```
Birthday:: [[Aug 4th, 1961]] 
Birth-location:: [[USA/Hawaii/Honolulu]]
Location:: [[USA/Columbia/Washington]]
Relationship:: [[Merried]]
Industry:: [[Politics]]
Vip::
Last-connected::
Email::
Phone::
LinkedIn:: https://www.linkedin.com/in/barackobama/
Instagram:: https://www.instagram.com/barackobama/
Source::

## Background
    - ### 👪 Family
        - ...
    - ### 🏢 Company
        - ...
    - ### ♥️️ Likes and dislikes
    -
    - ### 📝 Notes
        - I'm a big fan.
```

To get a table view of all the upcoming birthdays, I studied Logseq's query DSL and Clojure. Working on Logseq query DSL is difficult because of the need for better documentation.

* [Here you can see the use case](https://samples.cajueiro.me/#/page/case);
* [Here you can see the queries and comments I did](https://samples.cajueiro.me/#/page/queries);
  * Sadly, the publisher has a bug since some queries are not displayed (reported [here](https://github.com/logseq/logseq/issues/7332)). You can run this graph on your machine by:
    * [Downloading this repository](https://github.com/MarceloCajueiro/logseq-birthday-queries/archive/refs/heads/main.zip);
    * Unzipping and loading the graph on your Logseq app.

### The final query version by Darwn

Posted at [logseq discord](https://discord.com/channels/725182569297215569/1041747954819670076/1041982493236138025)

```clojure
{:title "🎂 Next birthdays 🎂"
   :query [:find (pull ?page [*]) ?birth-ddmm-num
           :keys people birthday
           :in $ ?start ?next
           :where
           [?page :block/name _]
           [?page :block/properties ?prop]
           [(get ?prop :birthday) ?birth]
           [?jpb :block/journal? true]
           [?jpb :block/original-name ?jbp-name] 
           [(contains? ?birth ?jbp-name)] ;get related journal from birthday property
           [?jpb :block/journal-day ?birth-int] ;get birthday in yyyymmdd format from related journal in birthday property
           [(str ?birth-int) ?birth-str] ;transform into string
           [(str ?start) ?start-str] ;regex only accept string
           [(str ?next) ?next-str]
           [(re-pattern "\\d{4}(\\d{4})") ?rx] ;regex pattern to split yyyymmdd into yyyy and mmdd
           [(re-matches ?rx ?birth-str) [_ ?birth-ddmm]] ;do regex match and store second output into variable
           [(re-matches ?rx ?start-str) [_ ?start-ddmm]] ;regex match will return ["2022" "1115"]
           [(re-matches ?rx ?next-str) [_ ?next-ddmm]]
           [(* 1 ?birth-ddmm) ?birth-ddmm-num] ;convert mmdd birthday back into number
           [(* 1 ?start-ddmm) ?start-ddmm-num]
           [(* 1 ?next-ddmm) ?next-ddmm-num]
           [(< ?birth-ddmm-num ?next-ddmm-num)] ; filter by today and next 30 day
           [(>= ?birth-ddmm-num ?start-ddmm-num)]
         ]
   :inputs [:today :30d-after]
   :result-transform (fn [res] 
      (sort-by (fn [s] (get-in s [:block/properties :birthday-int])) (map (fn [m] 
         (update (:people m) :block/properties 
             (fn [u] (assoc u :birthday-int (get-in m [:birthday]))
             ))
      ) res))
   )
  }
```

### Pending improvements

- [X] Sort by birthday;
- [X] Ignore the year on the query (in the current version, it will be necessary to have the date of the next birthday and not the person's birth date;)
- [X] Create a query that can be loaded properly in the `default-queries`;

#### Here are some references I used to develop the query:
* [Advanced Queries official doc](https://docs.logseq.com/#/page/advanced%20queries)
* [Discuss answer on how to query block property with a date](https://discuss.logseq.com/t/how-to-query-block-property-with-a-date/11825/6?u=cashew)
* [Query example - using custom view](https://gist.github.com/tiensonqin/b319e19e6a1ef4659f24bb3b71d3d025)
* [Logseq source: db rules](https://github.com/logseq/logseq/blob/master/deps/db/src/logseq/db/rules.cljc)
* [Logseq source: db schema](https://github.com/logseq/logseq/blob/master/deps/db/src/logseq/db/schema.cljs)

Special thanks to [@pengx17](https://github.com/pengx17) for this [logseq publish tool](https://github.com/pengx17/logseq-publish) that make it easy to publish this graph.
