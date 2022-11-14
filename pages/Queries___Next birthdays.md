- Using the property: `query-properties: [:page :birthday]`
- query-properties:: [:page :birthday]
  id:: 63737d32-22a1-4d90-a789-afa336299130
  #+BEGIN_QUERY
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
  #+END_QUERY
-
-