tags:: #query

- #+BEGIN_TIP
  **Result intended:**
  Every page with birthday property in a 30 day range ordered by birthday regardless of the year.
  #+END_TIP
- ## TLDR;
	- query-properties:: [:page :birthday]
	  ((63737d32-22a1-4d90-a789-afa336299130))
- ## Data to query
  collapsed:: true
	- ### People:
		- [[@Person 1]]
		- [[@Person 2]]
		- [[@Person 3]]
		- [[@Person 4]]
		- [[@Person 5]]
		- [[@Person 6]]
		- Simple Block - not page
			- birthday:: [[Nov 15th, 2022]]
- ## My path to create the query
  collapsed:: true
	- ### Every block with property birthday
	  collapsed:: true
		- query-table:: false
		  #+BEGIN_QUERY
		  {:query
		   [:find
		    (pull ?b [*])
		  :where
		    [?b :block/properties ?properties]
		    [(get ?properties :birthday) ?birthday]
		  
		  ]}
		  #+END_QUERY
	- ### Every page with the birthday block
	  collapsed:: true
		- #+BEGIN_QUERY
		  {:query
		   [:find
		  (pull ?page [*])
		  :where
		  [?b :block/properties ?properties]
		  [(get ?properties :birthday) ?birthday]
		  [?b :block/left ?page]
		  ]}
		  #+END_QUERY
	- ### Every page with the birthday block filtering by date range (today and 30d from now)
	  collapsed:: true
		- #+BEGIN_QUERY
		  {:query
		   [:find
		  (pull ?page [*])
		  :in $ ?start ?next
		  :where
		  [?b :block/properties ?properties]
		  [(get ?properties :birthday) ?birthday]
		  [?b :block/left ?page]
		  [?b :block/ref-pages ?p]
		  [?p :block/journal? true]
		  [?p :block/journal-day ?d]
		  [(< ?d ?next)]
		  [(>= ?d ?start)]
		  ]
		  :inputs [:today :30d-after]
		  }
		  #+END_QUERY
	- ### Every page with the birthday block filtering by date range selecting columns
	  collapsed:: true
		- Properties:
		  collapsed:: true
			- query-properties: [:page :birthday]
			  query-properties:: [:page :birthday]`
			  query-table:: true
			  query-table: true
		- Bug:
		  collapsed:: true
			- Not able to see the configured query properties after setting up.
		- query-table:: true
		  query-properties:: [:page :birthday]
		  query-sort-by:: birthday
		  query-sort-desc:: false
		  #+BEGIN_QUERY
		  {:query
		   [:find
		  (pull ?page [*])
		  :in $ ?start ?next
		  :where
		  [?b :block/properties ?properties]
		  [(get ?properties :birthday) ?birthday]
		  [?b :block/left ?page]
		  [?b :block/ref-pages ?p]
		  [?p :block/journal? true]
		  [?p :block/journal-day ?d]
		  [(< ?d ?next)]
		  [(>= ?d ?start)]
		  ]
		  :table-view? true
		  :inputs [:today :30d-after]
		  }
		  #+END_QUERY
	- ### Every page with the birthday block filtering by date range and sorted
	  collapsed:: true
		- Using properties
		  id:: 63724ed1-9102-4046-9a19-6461692cfc48
		  collapsed:: true
			- query-sort-by: birthday
			  query-sort-desc: false
		- Bug: sort not working
		- query-sort-by:: birthday
		  query-sort-desc:: false
		  #+BEGIN_QUERY
		  {:query
		   [:find
		  (pull ?page [*])
		  :in $ ?start ?next
		  :where
		  [?b :block/properties ?properties]
		  [(get ?properties :birthday) ?birthday]
		  [?b :block/left ?page]
		  [?b :block/ref-pages ?p]
		  [?p :block/journal? true]
		  [?p :block/journal-day ?d]
		  [(< ?d ?next)]
		  [(>= ?d ?start)]
		  ]
		  :inputs [:today :30d-after]
		   :result-transform (fn [result]
		                   (->> result
		                     (sort-by 
		                      (fn [h]
		                          (get h [:block/created_at])
		                        );fn
		                      );sort
		                     (take-last 20)
		                     );result
		                   );result transform
		  }
		  #+END_QUERY
	- ### Simple version that could work
	  id:: 63725ea3-4322-43b1-90ac-c6843ddde90a
	  collapsed:: true
		- When querying and selecting specific values, it could be great to render only those columns already configured.
		- Example not working:
		  collapsed:: true
			- #+BEGIN_QUERY
			  {
			   :query
			   [:find
			     ?birthday ?page
			    :in $ ?start ?next
			    :where
			    [?b :block/properties ?properties]
			    [(get ?properties :birthday) ?birthday]
			    [?b :block/left ?page]
			    [?b :block/ref-pages ?p]
			    [?p :block/journal? true]
			    [?p :block/journal-day ?d]
			    [(< ?d ?next)]
			    [(>= ?d ?start)]]
			   :table-view? true
			   :inputs [:today :30d-after]}
			  #+END_QUERY
		- [Feature proposal](https://discuss.logseq.com/t/queries-with-specific-attributes-on-find-organized-in-columns-with-table-view/12441)
	- ### Every page with birthday property in a 30 day range ordered by birthday and showing the columns Page and Birthday
	  collapsed:: true
		- query-sort-by:: page
		  query-table:: true
		  query-sort-desc:: false
		  query-properties:: [:page :birthday]
		  #+BEGIN_QUERY
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
	- ### [Final Version] Every page with birthday property in a 30 day range ordered by birthday regardless of the birthday year
	  id:: 63739ba8-2819-4368-a3e5-35c2470c80ae
	  collapsed:: true
		- Awesome Darwin version posted at [logseq discord](https://discord.com/channels/725182569297215569/1041747954819670076/1041982493236138025)
		  id:: 63737d26-ab26-4465-a806-7e9d364b3595
		- Using the property: `query-properties: [:page :birthday]`
		- query-properties:: [:page :birthday]
		  ((63737d32-22a1-4d90-a789-afa336299130))
	- References
		- [Advanced Queries official doc](https://docs.logseq.com/#/page/advanced%20queries)
		- [Discuss answer on how to query block property with a date](https://discuss.logseq.com/t/how-to-query-block-property-with-a-date/11825/6?u=cashew)\
		- [Query example - using custom view](https://gist.github.com/tiensonqin/b319e19e6a1ef4659f24bb3b71d3d025)
		- [Logseq source: db rules](https://github.com/logseq/logseq/blob/master/deps/db/src/logseq/db/rules.cljc)
		- [Logseq source: db schema](https://github.com/logseq/logseq/blob/master/deps/db/src/logseq/db/schema.cljs)