icon:: 🎮
cluster:: hobby

- collapsed:: true
  #+BEGIN_QUERY
  {:title [:sup "Blokken dit jaar"]
   :inputs [:-1m :query-page]
   :query [:find ?d ?pn ?c
    :keys day page content
    :in $ ?dag ?act
    :where
     [(str ?dag) ?strdag]
     [(subs ?strdag 0 4) ?jaar]
     [(str ?jaar "0101") ?strstart]
     [(* 1 ?strstart) ?start]
     [(str ?jaar "1231") ?strend]
     [(* 1 ?strend) ?end]
     [?r :block/name ?act]
     [?b :block/refs ?r]
     [?b :block/page ?p]
     [?p :block/original-name ?pn]
     [?p :block/journal-day ?d]
     [(>= ?d ?start)]
     [(<= ?d ?end)]
     [?b :block/content ?c]
   ]
   :result-transform (fn [result] (sort-by (fn [d] (get-in d [:day])) > result))
   :view (fn [rows] [:table [:thead [:tr [:th "Page"] [:th "Content"] ] ]
    [:tbody (for 
     [r rows
      :let [page (str (get-in r [:page])) ]
      :let [content (str (get-in r [:content]))]
      :let [firstline (first (str/split-lines content))]
      :let [nolinks (str/replace (str/replace firstline "[[" "") "]]" "") ]
      :let [displayline (str/replace (str/replace nolinks "DONE" "") "📓dagboek : " "") ]
     ]
     [:tr
      [:td [:a {:href (str "#/page/" page)} page] ]
      [:td displayline ]
     ]
    )]
   ]) 
   :collapsed? true
  }
  #+END_QUERY
- #+BEGIN_QUERY
  {:title [:b "Laatste keer"]
   :inputs [:query-page]
   :query [:find (pull ?p [*])
    :in $ ?page
    :where
     [?a :block/name ?page]
     [?b :block/refs ?a]
     [?a :block/original-name ?act]
     (not [?b :block/marker] )
     [?b :block/page ?p]
     [?p :block/original-name ?dag]
     [?p :block/journal-day ?d]
   ]
   :result-transform (fn [result] (take 1 (sort-by (fn [r] (get r :block/journal-day)) > result) ) )
   :view (fn [rows] (for [r rows] [:a {:href (str "#/page/" (get r :block/original-name))} (get r :block/original-name)] ))
  }
  #+END_QUERY
- #+BEGIN_QUERY
  {:title [:b "Aantal dagen"]
   :inputs [:query-page :today]
   :query [:find (count ?d) (sum ?count)
    :keys day aantal
    :in $ ?page ?today
    :where
     [?a :block/name ?page]
     [?b :block/refs ?a]
     (not [?b :block/marker] )
     [?b :block/page ?p]
     [?p :block/journal-day ?d]
     [(str ?today) ?strday]
     [(subs ?strday 0 6) ?jm]
     [(str ?jm "01") ?jmd]
     [(* ?jmd 1) ?start]
     (or 
       (and 
         [(>= ?d ?start)]
         [(* 1 1) ?count]
       )
       (and 
         [(< ?d ?start)]
         [(* 0 1) ?count]
       )
     )
   ]
   :view (fn [rows] (for [r rows] (str "Deze maand: " (get r :aantal) " | Totaal: " (get r :day)) ) )
  }
  #+END_QUERY
- #+BEGIN_QUERY
  {:title [:h3 "✅ Taakjes"]
   :inputs [:query-page]
   :query [:find (pull ?b [*])
    :in $ ?page
    :where
     [?p :block/name ?page]
     [?b :block/refs ?p]
     [?b :block/marker "TODO"]
   ]
   :result-transform :sort-tasks
   :group-by-page? false
   :breadcrumb-show? false
  }
  #+END_QUERY
- ## gamelijst
	- [[2024]]
		- [[Tears of the Kingdom]] (switch)
		  id:: 6597f870-ae13-434d-ab5c-b047c43ea42c
		- [[Tunic]] (steam)
		  id:: 6597f87d-e4a0-4d57-a2ff-e8363a9991c9
		- [[Kena Bridge of Spirits]] (epic)
		- [[Dark Souls 3]] (steam)
		- [[Bloodborne]] (PS4)
	- query-properties:: [:page :status :gespeeld :credits :extras :dlc :datum-begin]
	  collapsed:: true
	  #+BEGIN_QUERY
	  {:title [:h3 "📜 Open complete lijst"]
	   :inputs [:query-page]
	   :query [:find (pull ?p [*])
	    :in $ ?page
	    :where
	     [?p :block/properties ?prop]
	     [(get ?prop :hobby) ?hobby]
	     [(contains? ?hobby ?page)]
	     [(get ?prop :game-type) ?type]
	     [(= ?type "eindig")]
	     [(get ?prop :status) ?status]
	     (not [(contains? #{"afgerond" "vriezer ❄️" "archief"} ?status)])
	   ]
	   :collapsed? true
	   :result-transform (fn [result] (sort-by 
	      (juxt
	        (fn [r] (get-in r [:block/properties :game-type]))
	        (fn [r] (get-in r [:block/properties :status]))
	        (fn [r] (get-in r [:block/properties :gespeeld]))
	        (fn [r] (get-in r [:block/properties :credits]))
	        (fn [r] (get-in r [:block/properties :extras]))
	        (fn [r] (get-in r [:block/properties :dlc]))
	        (fn [r] (get-in r [:block/properties :datum-begin]))
	        (fn [r] (get r :block/name))
	      )
	      result
	    ) )
	  }
	  #+END_QUERY
	- ### 🔘 Unplayed Epic Games
	  collapsed:: true
		- The Lion's Song
		- Nioh
		- Amnesia: Rebirth
		- Hundred Days - Winemaking Simulator
		- Spirit of the North
		- The Captain
		- Darkwood
		- Filament
		- Rise of Industry
		- shapez
		- DnD Idle Champions
		- The Dungeon of Naheulbeuk
		- Orwell
	- ### 🎁 Wishlist
	  collapsed:: true
		- Baldur's Gate 3
		- Sable
		- Citizen Sleeper
		- Elden Ring
		- Syberia: The World Before
		- [[Diablo IV]]
		  collapsed:: true
			- icon:: 🎮
			  cluster:: hobby
			  hobby:: gaming
			  game-type:: eindig
			  status:: ooit 📦
			  gespeeld:: ✅ (demo)
			  credits:: ❌
			  extras:: ➖
			  dlc:: ➖
			  datum-begin:: ➖
			  datum-eind:: ➖
		- The Case of the Golden Idol
		- #### Potential interest
			- The Wandering Village
			- Chicory 
			  https://store.steampowered.com/app/1123450/Chicory_A_Colorful_Tale/
			- Scorn
		- #### In development
			- Sunny Side
- query-properties:: [:page :game-type :credits :extras :dlc :datum-begin]
  #+BEGIN_QUERY
  {:title [:h3 "🚀 Actief"]
   :inputs [:query-page]
   :query [:find (pull ?p [*])
    :in $ ?page
    :where
     [?p :block/properties ?prop]
     [(get ?prop :hobby) ?hobby]
     [(contains? ?hobby ?page)]
     [(get ?prop :status) ?status]
     [(= ?status "actief 🚀")]
   ]
   :result-transform (fn [result] (sort-by 
      (juxt
        (fn [r] (get-in r [:block/properties :game-type]))
        (fn [r] (get-in r [:block/properties :status]))
        (fn [r] (get-in r [:block/properties :gespeeld]))
        (fn [r] (get-in r [:block/properties :credits]))
        (fn [r] (get-in r [:block/properties :extras]))
        (fn [r] (get-in r [:block/properties :dlc]))
        (fn [r] (get-in r [:block/properties :datum-begin]))
        (fn [r] (get r :block/name))
      )
      result
    ) )
  }
  #+END_QUERY
- query-properties:: [:page :game-type :credits :extras :dlc :datum-begin]
  #+BEGIN_QUERY
  {:title [:h3 "🛰️ Pauze"]
   :inputs [:query-page]
   :query [:find (pull ?p [*])
    :in $ ?page
    :where
     [?p :block/properties ?prop]
     [(get ?prop :hobby) ?hobby]
     [(contains? ?hobby ?page)]
     [(get ?prop :status) ?status]
     [(= ?status "pauze 🛰️")]
   ]
   :result-transform (fn [result] (sort-by 
      (juxt
        (fn [r] (get-in r [:block/properties :game-type]))
        (fn [r] (get-in r [:block/properties :status]))
        (fn [r] (get-in r [:block/properties :gespeeld]))
        (fn [r] (get-in r [:block/properties :credits]))
        (fn [r] (get-in r [:block/properties :extras]))
        (fn [r] (get-in r [:block/properties :dlc]))
        (fn [r] (get-in r [:block/properties :datum-begin]))
        (fn [r] (get r :block/name))
      )
      result
    ) )
  }
  #+END_QUERY
- query-properties:: [:page :game-type :credits :extras :dlc :datum-begin]
  #+BEGIN_QUERY
  {:title [:h3 "⏳ Volgend (nog niet gespeeld)"]
   :inputs [:query-page]
   :query [:find (pull ?p [*])
    :in $ ?page
    :where
     [?p :block/properties ?prop]
     [(get ?prop :hobby) ?hobby]
     [(contains? ?hobby ?page)]
     [(get ?prop :status) ?status]
     [(= ?status "volgend ⏳")]
   ]
   :result-transform (fn [result] (sort-by 
      (juxt
        (fn [r] (get-in r [:block/properties :game-type]))
        (fn [r] (get-in r [:block/properties :status]))
        (fn [r] (get-in r [:block/properties :gespeeld]))
        (fn [r] (get-in r [:block/properties :credits]))
        (fn [r] (get-in r [:block/properties :extras]))
        (fn [r] (get-in r [:block/properties :dlc]))
        (fn [r] (get-in r [:block/properties :datum-begin]))
        (fn [r] (get r :block/name))
      )
      result
    ) )
  }
  #+END_QUERY
- query-properties:: [:page :game-type :credits :extras :dlc :datum-begin]
  #+BEGIN_QUERY
  {:title [:h3 "📦 Ooit (gespeeld)"]
   :inputs [:query-page]
   :query [:find (pull ?p [*])
    :in $ ?page
    :where
     [?p :block/properties ?prop]
     [(get ?prop :hobby) ?hobby]
     [(contains? ?hobby ?page)]
     [(get ?prop :status) ?status]
     [(= ?status "ooit 📦")]
   ]
   :result-transform (fn [result] (sort-by 
      (juxt
        (fn [r] (get-in r [:block/properties :game-type]))
        (fn [r] (get-in r [:block/properties :status]))
        (fn [r] (get-in r [:block/properties :gespeeld]))
        (fn [r] (get-in r [:block/properties :credits]))
        (fn [r] (get-in r [:block/properties :extras]))
        (fn [r] (get-in r [:block/properties :dlc]))
        (fn [r] (get-in r [:block/properties :datum-begin]))
        (fn [r] (get r :block/name))
      )
      result
    ) )
  }
  #+END_QUERY
- query-sort-by:: page
  query-sort-desc:: false
  collapsed:: true
  #+BEGIN_QUERY
  {:title [:b "Openstaande acties"]
   :inputs [:query-page]
   :query [:find (pull ?b [*])
    :in $ ?page
    :where
     [?p :block/properties ?prop]
     [(get ?prop :hobby) ?hobby]
     [(contains? ?hobby ?page)]
     [(get ?prop :status) ?status]
     [(= ?status "ooit 📦")]
     [?b :block/page ?p]
     [?b :block/marker "TODO"]
   ]
  }
  #+END_QUERY
- query-properties:: [:page :game-type :extras :dlc :datum-begin :datum-eind]
  #+BEGIN_QUERY
  {:title [:h3 "✅ Afgerond dit jaar"]
   :inputs [:query-page :today]
   :query [:find (pull ?p [*])
    :in $ ?page ?today
    :where
     [?p :block/properties ?prop]
     [(get ?prop :hobby) ?hobby]
     [(contains? ?hobby ?page)]
     [(get ?prop :status) ?status]
     [(= ?status "afgerond")]
     [(get ?prop :datum-eind) ?eind]
     [(subs ?eind 0 4) ?eindjr]
     [(str ?today) ?dag]
     [(subs ?dag 0 4) ?jaar]
     [(= ?eindjr ?jaar)]
   ]
   :result-transform (fn [result] (sort-by 
      (juxt
        (fn [r] (get-in r [:block/properties :datum-eind]))
        (fn [r] (get-in r [:block/properties :game-type]))
        (fn [r] (get-in r [:block/properties :status]))
        (fn [r] (get-in r [:block/properties :gespeeld]))
        (fn [r] (get-in r [:block/properties :credits]))
        (fn [r] (get-in r [:block/properties :extras]))
        (fn [r] (get-in r [:block/properties :dlc]))
        (fn [r] (get-in r [:block/properties :datum-begin]))
        (fn [r] (get r :block/name))
      )
      > result
    ) )
  }
  #+END_QUERY
- query-properties:: [:page :game-type :extras :dlc :datum-begin :datum-eind]
  collapsed:: true
  #+BEGIN_QUERY
  {:title [:h3 "✅ Eerder afgerond"]
   :inputs [:query-page :today]
   :query [:find (pull ?p [*])
    :in $ ?page ?today
    :where
     [?p :block/properties ?prop]
     [(get ?prop :hobby) ?hobby]
     [(contains? ?hobby ?page)]
     [(get ?prop :status) ?status]
     [(= ?status "afgerond")]
     [(get ?prop :datum-eind) ?eind]
     [(subs ?eind 0 4) ?eindjr]
     [(str ?today) ?dag]
     [(subs ?dag 0 4) ?jaar]
     [(< ?eindjr ?jaar)]
   ]
   :result-transform (fn [result] (sort-by 
      (juxt
        (fn [r] (get-in r [:block/properties :datum-eind]))
        (fn [r] (get-in r [:block/properties :game-type]))
        (fn [r] (get-in r [:block/properties :status]))
        (fn [r] (get-in r [:block/properties :gespeeld]))
        (fn [r] (get-in r [:block/properties :credits]))
        (fn [r] (get-in r [:block/properties :extras]))
        (fn [r] (get-in r [:block/properties :dlc]))
        (fn [r] (get-in r [:block/properties :datum-begin]))
        (fn [r] (get r :block/name))
      )
      > result
    ) )
  }
  #+END_QUERY
	- [[Albion Online]]
	  collapsed:: true
		- icon:: 🎮
		  cluster:: hobby
		  hobby:: gaming
		  game-type:: oneindig
		  status:: afgerond
		  gespeeld:: ✅
		  credits:: ➖
		  extras:: ➖
		  dlc:: ➖
		  datum-begin:: ?
		  datum-eind:: 2022-06-11
- query-properties:: [:page :game-type :credits :extras :dlc :datum-begin :datum-eind]
  collapsed:: true
  #+BEGIN_QUERY
  {:title [:h3 "❄️ Vriezer (mee gestopt)"]
   :inputs [:query-page]
   :query [:find (pull ?p [*])
    :in $ ?page
    :where
     [?p :block/properties ?prop]
     [(get ?prop :hobby) ?hobby]
     [(contains? ?hobby ?page)]
     [(get ?prop :status) ?status]
     [(= ?status "vriezer ❄️")]
   ]
   :result-transform (fn [result] (sort-by 
      (juxt
        (fn [r] (get-in r [:block/properties :game-type]))
        (fn [r] (get-in r [:block/properties :status]))
        (fn [r] (get-in r [:block/properties :gespeeld]))
        (fn [r] (get-in r [:block/properties :credits]))
        (fn [r] (get-in r [:block/properties :extras]))
        (fn [r] (get-in r [:block/properties :dlc]))
        (fn [r] (get-in r [:block/properties :datum-begin]))
        (fn [r] (get r :block/name))
      )
      result
    ) )
  }
  #+END_QUERY
	- Geneforge 1: Mutagen
	  collapsed:: true
		- icon:: 🎮
		  cluster:: hobby
		  hobby:: gaming
		  game-type:: eindig
		  status:: vriezer ❄️
		  gespeeld:: ✅
		  credits:: ❌
		  extras:: ➖
		  dlc:: ➖
		  datum-eind::
	- Gods Will Fall
	  collapsed:: true
		- icon:: 🎮
		  cluster:: hobby
		  hobby:: gaming
		  game-type:: eindig
		  status:: vriezer ❄️
		  gespeeld:: ✅
		  credits:: ❌
		  extras:: ➖
		  dlc:: ➖
		  datum-eind::
	- Majesty Gold HD
	  collapsed:: true
		- icon:: 🎮
		  cluster:: hobby
		  hobby:: gaming
		  game-type:: eindig
		  status:: vriezer ❄️
		  gespeeld:: ✅
		  credits:: ❌
		  extras:: ➖
		  dlc:: ➖
		  datum-eind::
	- Opus Magnum
	  collapsed:: true
		- icon:: 🎮
		  cluster:: hobby
		  hobby:: gaming
		  game-type:: eindig
		  status:: vriezer ❄️
		  gespeeld:: ✅
		  credits:: ❌
		  extras:: ➖
		  dlc:: ➖
		  datum-eind::
	- [[The Walking Dead The Final Season]]
	  collapsed:: true
		- icon:: 🎮
		  cluster:: hobby
		  hobby:: gaming
		  game-type:: eindig
		  status:: vriezer ❄️
		  gespeeld:: ❌
		  credits:: ❌
		  extras:: ➖
		  dlc:: ➖
		  datum-begin:: ➖
		  datum-eind:: ➖
	- Torchlight 2
	  collapsed:: true
		- icon:: 🎮
		  cluster:: hobby
		  hobby:: gaming
		  game-type:: eindig
		  status:: vriezer ❄️
		  gespeeld:: ✅
		  credits:: ❌
		  extras:: ➖
		  dlc:: ➖
		  datum-eind::
	- Windbound
	  collapsed:: true
		- icon:: 🎮
		  cluster:: hobby
		  hobby:: gaming
		  game-type:: eindig
		  status:: vriezer ❄️
		  gespeeld:: ✅
		  credits:: ❌
		  extras:: ➖
		  dlc:: ➖
		  datum-eind::
- collapsed:: true
  #+BEGIN_QUERY
  {:title [:sup "Blokken dit jaar zonder oneindig"]
   :inputs [:-1m :query-page]
   :query [:find ?d ?pn ?c
    :keys day page content
    :in $ ?dag ?act
    :where
     (page-property ?g :hobby ?act)
     (page-property ?g :game-type "eindig")
     [(str ?dag) ?strdag]
     [(subs ?strdag 0 4) ?jaar]
     [(str ?jaar "0101") ?strstart]
     [(* 1 ?strstart) ?start]
     [(str ?jaar "1231") ?strend]
     [(* 1 ?strend) ?end]
     [?r :block/name ?act]
     [?b :block/refs ?r]
     [?b :block/refs ?g]
     [?b :block/page ?p]
     [?p :block/original-name ?pn]
     [?p :block/journal-day ?d]
     [(>= ?d ?start)]
     [(<= ?d ?end)]
     [?b :block/content ?c]
   ]
   :result-transform (fn [result] (sort-by (fn [d] (get-in d [:day])) > result))
   :view (fn [rows] [:table [:thead [:tr [:th "Page"] [:th "Content"] ] ]
    [:tbody (for 
     [r rows
      :let [page (str (get-in r [:page])) ]
      :let [content (str (get-in r [:content]))]
      :let [firstline (first (str/split-lines content))]
      :let [nolinks (str/replace (str/replace firstline "[[" "") "]]" "") ]
      :let [displayline (str/replace (str/replace nolinks "DONE" "") "📓dagboek : " "") ]
     ]
     [:tr
      [:td [:a {:href (str "#/page/" page)} page] ]
      [:td displayline ]
     ]
    )]
   ]) 
  ; :collapsed? true
  }
  #+END_QUERY
	- #+BEGIN_QUERY
	  {:title [:b "Aantal dagen per maand (dit jaar)"]
	   :inputs [:query-page :today]
	   :query [:find ?jm (count ?d)
	    :keys maand aantal
	    :in $ ?page ?today
	    :where
	     [?a :block/name ?page]
	     [?b :block/refs ?a]
	     (not [?b :block/marker] )
	     [?b :block/page ?p]
	     [?p :block/journal-day ?d]
	     [(str ?today) ?strday]
	     [(subs ?strday 0 4) ?jaar]
	     [(str ?jaar "0101") ?strstart]
	     [(* ?strstart 1) ?start]
	     [(str ?jaar "1231") ?strend]
	     [(* ?strend 1) ?end]
	     [(>= ?end ?d ?start)]
	     [(str ?d) ?dag]
	     [(re-pattern "(\\d{4})(\\d{2})") ?rx]
	     [(re-find ?rx ?dag) [_ ?yyyy ?mm]]
	     [(str ?yyyy "/" ?mm) ?jm]
	   ]
	   :result-transform (fn [result] (sort-by (fn [r] (get r :maand)) result))
	   :view (fn [rows] [:table 
	     [:thead [:tr 
	       [:th "Maand"]
	       [:th "Aantal"]
	     ] ]
	     [:tbody (for [r rows] [:tr 
	       [:td (get-in r [:maand]) ]
	       [:td (get-in r [:aantal]) ]
	     ])]
	   ])
	  }
	  #+END_QUERY
- query-table:: false
  #+BEGIN_QUERY
  {:title [:b "✅ Afgerond per jaar"]
   :inputs [:query-page]
   :query [:find ?eindjr (count ?p)
    :keys jaar aantal
    :in $ ?page
    :where
     (page-property ?p :hobby ?page)
     (page-property ?p :status "afgerond")
     [?p :block/properties ?prop]
     [(get ?prop :status) ?status]
     [(= ?status "afgerond")]
     [(get ?prop :hobby) ?hobby]
     (or
       [(= ?hobby ?page)]
       [(contains? ?hobby ?page)]
     )
     [(get ?prop :datum-eind) ?eind]
     [(subs ?eind 0 4) ?eindjr]
   ]
   :result-transform (fn [result] (sort-by (fn [r] (get r :jaar)) > result ) )
   :view (fn [rows] [:table 
     [:thead [:tr 
       [:th "Jaar"]
       [:th "Aantal"]
     ] ]
     [:tbody (for [r rows] [:tr 
       [:td (get-in r [:jaar]) ]
       [:td (get-in r [:aantal]) ]
     ])]
   ])
  }
  #+END_QUERY
- #+BEGIN_QUERY
  {:title [:h3 "🏜 Referenties"]
   :inputs [:query-page]
   :query [:find (pull ?b [*])
    :in $ ?page
    :where
     [?o :block/name ?page]
     [?b :block/refs ?o]
     [?b :block/properties ?prop]
     [(get ?prop :type) ?type]
     [(= ?type "referentie")]
     [?b :block/page ?p]
   ]
  }
  #+END_QUERY
