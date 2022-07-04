---
published: true
title: learn xapian searching part 2
layout: post
---
We currently have a database populated with some hackernews posts, it is really not helpful if we don't implement the code to search the database and display results.

keep in mind this is a continuation of my last article on indexing you should [check it out](https://keosariel.github.io/2022/07/04/learn-xapian-basic-indexing-part-1/) before continuing.

## Queries in xapian

Queries are just a simple way by which documents are searched for in a database. Mostly use to search text-based terms, which can be combined using numbers of methods and operators to produce complex queries.

### Logical operators

When making a query, you most likely want to search for multiple terms i.e `"search engine"` or `["search", "engine"]`. You'd want results for both "search" and "engine", xapian lets you do this using logical operators like:

`OP_OR` - matches documents which match query A or B (or both)
`OP_AND` - matches documents which match both query A and B
`OP_AND_NOT` - matches documents which match query A but not B

just to name a few, you can [see here](https://getting-started-with-xapian.readthedocs.io/en/latest/concepts/search/queries.html) for more details.


### Query parser

To make searching databases simpler, Xapian provides a QueryParser class which converts a human readable query string into a Xapian Query object, for example: `search AND engine NOT google`

The QueryParser uses an internal process to convert the query string into terms. This is similar to the process used by the `xapian.TermGenerator`, which can be used at index time to convert a string into terms. It is often easiest to use `xapian.QueryParser` and `xapian.TermGenerator` on the same database.

create a `queryparser` in code would look like:

```python
import xapian

def get_query_parser(default_op=xapian.Query.OP_AND):
    queryparser = xapian.QueryParser()
    queryparser.set_stemmer(xapian.Stem("en"))
    queryparser.set_stemming_strategy(queryparser.STEM_SOME)
    queryparser.set_default_op(default_op)
    return queryparser
```

### Ranked matches

When you run a `query` using Xapian, what you get is a list of ranked matches. Every matched document returned when a `query` is run has a weight, the list of matches are sorted in descending order. The weight is an indicator of how good a document matched the `query`.

The actual weight is calculated by a weighting scheme; Xapian comes with a few different ones or you can write your own, although often the default is fine. (It uses a scheme called BM25, which takes into account things like how common a matching term is in a matching document compared to in the entire database, and the lengths of different matching documents.) reference [here for more details](https://getting-started-with-xapian.readthedocs.io/en/latest/concepts/search/ranked_matches.html)

### Code

Hers is what the implementation would look like

```python
import xapian
import json
+ import pprint

with open("./hn_2014.json", "r") as fp:
    hn_posts = json.load(fp)

- # create a new database or open available database in the current directory
- database = xapian.WritableDatabase("./hn_index", xapian.DB_CREATE_OR_OPEN)
+ # Open the database we're going to search.
+ database = xapian.Database("./hn_index")

termgenerator = xapian.TermGenerator()
stemmer = xapian.Stem("english")
termgenerator.set_stemmer(stemmer)

searchables = ["title", "storyText", "author"]
id = "objectId"

# assign a unique slot for each field for consistency
slots = {x: i for i, x in enumerate(searchables, 1)}
slots[id] = max(slots.values()) + 1


def index():
    ...
    
def get_query_parser(default_op=xapian.Query.OP_AND):
    queryparser = xapian.QueryParser()
    queryparser.set_stemmer(xapian.Stem("en"))
    queryparser.set_stemming_strategy(queryparser.STEM_SOME)
    queryparser.set_default_op(default_op)
    return queryparser

+ def search(querystring, fields=[], offset=0, pagesize=20):

+    # Set up a QueryParser with a stemmer and suitable prefixes
+ 	 queryparser = get_query_parser()
    
+    # And parse the query
+    query = queryparser.parse_query(querystring)

+    for field in fields:
+        field_prefix = "X" + field.upper()
+        queryparser.add_prefix(field, field_prefix)
    
+    # Use an Enquire object on the database to run the query
+    enquire = xapian.Enquire(database)
+    enquire.set_query(query)

+    matches = []
	
+    # And print out something about each match
+    for match in enquire.get_mset(offset, pagesize):
+        data = json.loads(match.document.get_data())
+        matches.append(data)
    
+    return matches

pprint.pprint(search("python"))
```
