---
published: false
---
Facets are a way to add specific, relevant options to search results. Think of it as a way to narrow down a search query, and being more specific on where your search should be executed. In the case of hackernews posts, when you decide to make a search query and you want only results from paul graham, you'd just specify that you only want search results from paul graham (pg on hackernews).

<p class="message">
  Note: keep in mind this is a continuation of my last article on searching you should [check it out](https://keosariel.github.io/2022/07/04/learn-xapian-basic-indexing-part-1/) before continuing.
</p>

So how do we implement this Faceting Search? facets in Xapian is based on a technique referred to as boolean filtering (and hence those prefixed terms are called boolean terms). A little of the concept was introduced, well not literally in [part 1 of this series: learn xapian basic indexing part 1
](https://keosariel.github.io/2022/07/04/learn-xapian-basic-indexing-part-1/).

We'd introduce a new type of field called facets and add few lines of code to indexing

```python
# hnsearch.py

import xapian
import json
+ from itertools import chain

with open("./hn_2014.json", "r") as fp:
    hn_posts = json.load(fp)

# create a new database or open available database in the current directory
database = xapian.WritableDatabase("./hn_index", xapian.DB_CREATE_OR_OPEN)

termgenerator = xapian.TermGenerator()
stemmer = xapian.Stem("english")
termgenerator.set_stemmer(stemmer)

- searchables = ["title", "storyText", "author"]
+ searchables = ["title", "storyText"]
id = "objectId"
+ facets = ["author"]

# assign a unique slot for each field for consistency
- slots = {x: i for i, x in enumerate(searchables, 1)}
+ slots = {x: i for i, x in enumerate(chain(searchables, facets), 1)}
slots[id] = max(slots.values()) + 1

def index():
	for post in hn_posts:
	    doc = xapian.Document()
	    termgenerator.set_document(doc)

	    for field in searchables:
	        value = post.get(field)
	        if value:
	            field_prefix = "X"+field.upper()
	            termgenerator.index_text(value, slots[field], field_prefix)
                termgenerator.index_text(value)
    	 
+       for field in facets:
+       	value = post.get(field)
+       	if value:
+    	    	# add a boolean term for filtering on field
+           	doc.add_boolean_term("{}:{}".format("X"+field.upper(), value))
+           	# index fields as value for faceting
+           	doc.add_value(slots[field], value)
              
	     # Storing the data for display purposes
	     doc.set_data(json.dumps(post, ensure_ascii=True))
	
	     # Unique id for the current document with prefix `Q`
	     idterm = u"Q" + str(post[id])
	     doc.add_boolean_term(idterm)

	     # Add or replace document in the database
	     database.replace_document(idterm, doc)
```

And also updating the search function to accomodate faceting, here is what it'd look like.

```python
- def search(querystring, fields=[], offset=0, pagesize=20):
+ def search(querystring, fields=[], facets={}, offset=0, pagesize=20):
    queryparser = get_query_parser()
    query = queryparser.parse_query(querystring)

    for field in fields:
        field_prefix = "X" + field.upper()
        queryparser.add_prefix(field, field_prefix)
    
+   if facets:
+        faceted_queries = [xapian.Query('{}:{}'.format("X"+field.upper(), value)) 
                                    for field, value in facets.items()]
+       faceted_query = xapian.Query(xapian.Query.OP_OR, faceted_queries)
+       query = xapian.Query(xapian.Query.OP_AND, query, faceted_query)

    enquire = xapian.Enquire(database)
    enquire.set_query(query)

    _matches = enquire.get_mset(offset, pagesize)
    matches = []

    for match in _matches:
        data = json.loads(match.document.get_data())
        matches.append(data)
    
    return matches
```

```python
pprint.pprint(search("search engine", facets={"author": "codecondo"}))
```

I hope this was helpful at least to get the base understanding of search and search in conjuction with xapian.

Here's the rest of this series

- [learn xapian basic indexing part 1
](https://keosariel.github.io/2022/07/04/learn-xapian-basic-indexing-part-1/)
- [learn xapian searching part 2
](https://keosariel.github.io/2022/07/04/learn-xapian-searching-part-2/)


