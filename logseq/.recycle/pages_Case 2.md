- #+BEGIN_QUERY
  {
  :title "🎂 Next birthdays 🎂"
  :query
   [:find
    ?name ?birthday ?d
    :in $ ?start ?next
    :where
    [?b :block/properties ?properties]
    [(get ?properties :birthday) ?birthday]
    [?b :block/left ?page]
    [?b :block/ref-pages ?p]
    [?p :block/journal? true]
    [?p :block/journal-day ?d]
    [?page :block/name ?name]
    [(< ?d ?next)]
    [(>= ?d ?start)]]
   :inputs [:today :30d-after]
  
   :view (fn [raw-people]
          (def people-count (/ (count raw-people) 3))
  
           (def ids (range people-count))
  
           (def people
             (map (fn [id]
                    (def idx (* id 3))
                    {:id id
                     :name (nth raw-people idx)
                     :birthday (nth raw-people (+ idx 1))
                     :date (nth raw-people (+ idx 2))}) ids))
  
           (def sorted-people (sort-by :date people))
  
           (def people
             (map (fn [id]
                    (def idx (* id 3))
                    {:id id
                     :name (nth people idx)
                     :birthday (nth people (+ idx 1))
                     :date (nth people (+ idx 2))}) ids))
  
           (defn link [path] [:a {:href (str "#/page/" path)} (str path)])
  
           (def rows
             (map (fn [person]
                    [:tr
                     [:td (link (get person :name))]
                     [:td (link (first (get person :birthday)))]]) sorted-people))
  
  
  
           [:div.table-wrapper
            [:table.table-auto
             [:thead
              [:tr
               [:th "Name"]
               [:th "Birthday"]]]
             [:tbody
              rows]]])}
  #+END_QUERY
- ((63725ea3-4322-43b1-90ac-c6843ddde90a))