---
published: true
layout: post
title: 'learn xapian: basic indexing part 1'
---
Xapian is a free and open-source C++ probabilistic information retrieval library that provides very powerful tools or functions. Xapian scales very well as it says [here](https://xapian.org/docs/scalability.html#:~:text=People%20often%20want%20to%20know,1.5%20terabytes%20of%20database%20files) - an early version of the software powered the (now defunct) Webtop search engine which offered a search over around 500 million web pages (around 1.5 terabytes of database files). Searches took less than a second.

Compared with other information retrieval on the market, Xapian is similar to [Lucene](https://lucene.apache.org/), providing a rich and extensible programming interface, allowing Xapian to better integrate into your system. At the same time, its retrieval performance is much higher than that of Lucene, and the [BM-25](https://xapian.org/docs/bm25.html) model is adopted, which has better retrieval effect.

So, this article would be the first of a 4 series. Basically making it easy for anyone to learn to use the xapian library in python and implement a good search for an application. Also it'd be of help if you understand or is familar with [Term Frequency-inverse document frequency (TF-IDF)](https://en.wikipedia.org/wiki/Tf%E2%80%93idf) and weighting schemes like [BM25](https://xapian.org/docs/bm25.html).

I think the [Xapian's tutorial](https://getting-started-with-xapian.readthedocs.io/en/latest/index.html) is documented properly and easy to understand. I have been using xapian for a while now and I think a little more details on certain concepts wasn't explored too well. I also have the mentality of learning while writing, so I can write this series of articles. 

So, in this series we'd create a good new search engine for [hackernews](https://news.ycombinator.com/) very similar to the [algolia search engine](https://hn.algolia.com/) for hakernews.

## Creating an index

We'd create a database using the `xapian.WritableDatabase` taking 2 arguements `database_path`, `xapian_flag`. In this example we'd name the index `hn_index` and would use the xapian flag `xapian.DB_CREATE_OR_OPEN` which simply states: create a new index if `hn_index` does not exists else open and use the index `hn_index`. So this is what it'd look like in code.

```python
# index.py

import xapian


# create a new database or open available database in the current directory
database = xapian.WritableDatabase("./hn_index", xapian.DB_CREATE_OR_OPEN)
```

Now, an index has been created the next thing is to index data. So, I have prepared a json file filled with hackernews posts before hand. So lets read some data.

```python
# index.py

import xapian
+ import json

+ with open("./hn_2014.json", "r") as fp:
+     hn_posts = json.load(fp)

# create a new database or open available database in the current directory
database = xapian.WritableDatabase("./hn_index", xapian.DB_CREATE_OR_OPEN)
```
Here I have an example of a hacker news post so you'd see how the data would be indexed.

```
{'author': 'gere',
  'createdAt': '2014-05-15T07:38:03Z',
  'createdAtI': 1400139483,
  'numComments': 0,
  'objectId': '7748233',
  'points': 2,
  'storyText': '',
  'tags': ['story', 'author_gere', 'story_7748233'],
  'title': 'Python Vs Cython Vs Numba',
  'url': 'http://htmlpreview.github.io/?https://github.com/rasbt/One-Python-benchmark-per-day/blob/master/htmls/day4_python_cython_numba.html'}
```

From the data above you'd realize how it can be searched, we can index fields like `title` and `author` and make them searchable and also the fields `numComments` and `points` sortable fields.


## Indexing data

We'd need a means to tokenize string of texts i.e turning `"gabriel is writing a good documentation"` to `["gabriel", "is", "writing", "a", "good", "documentation"]`. Also we'd need to stem text fields which is good for searching i.e turing `["gabriel", "is", "writing", "a", "good", "documentation"]` to `["gabriel", "is", "write", "a", "good", "document"]`. This way when someone queries for `"searching"` you'd also get results for `"search"`. As [wikipedia](https://en.wikipedia.org/wiki/Stemming) puts it "stemming is the process of reducing inflected (or sometimes derived) words to their word stem, base or root form".

In xapian there's a bunch of tools to get these results with ease, and it'd be the use of `xapian.TermGenerator` and `xapian.Stem`. Here is how we'd use it..

```python
# index.py

import xapian
import json

with open("./hn_2014.json", "r") as fp:
    hn_posts = json.load(fp)

# create a new database or open available database in the current directory
database = xapian.WritableDatabase("./hn_index", xapian.DB_CREATE_OR_OPEN)

+ termgenerator = xapian.TermGenerator()
+ stemmer = xapian.Stem("english")
+ termgenerator.set_stemmer(stemmer)
```

Now after we've set up the `termgenerator` we need to index data and store them. In xapian, data are seen as documents (`xapian.Document`), so we'd be indexing using documents and termgenerators together. 

Also, we'd need to specify the fields we'd be indexing.

```python
# index.py

import xapian
import json

with open("./hn_2014.json", "r") as fp:
    hn_posts = json.load(fp)

# create a new database or open available database in the current directory
database = xapian.WritableDatabase("hn_index", xapian.DB_CREATE_OR_OPEN)

termgenerator = xapian.TermGenerator()
stemmer = xapian.Stem("english")
termgenerator.set_stemmer(stemmer)

+ searchables = ["title", "storyText", "author"]
+ id = "objectId"

+ # assign a unique slot for each field for consistency
+ slots = {x: i for i, x in enumerate(searchables, 1)}
+ slots[id] = max(slots.values()) + 1
```

### Fields and term prefixes

Xapian supports a convention for representing fields in the database by mapping each field to a term prefix, which are one or more capital letters; this is to avoid confusion (which could adversely affect search results) with normal terms generated from words, which are lowercased by the Term Generator. Prefixes make it easy to narrow your query and there are conventional prefixes

With the infomation above, he's a practical example of how we'd implement it.

```python
# index.py

import xapian
import json

with open("./hn_2014.json", "r") as fp:
    hn_posts = json.load(fp)

# create a new database or open available database in the current directory
database = xapian.WritableDatabase("./hn_index", xapian.DB_CREATE_OR_OPEN)

termgenerator = xapian.TermGenerator()
stemmer = xapian.Stem("english")
termgenerator.set_stemmer(stemmer)

searchables = ["title", "storyText", "author"]
id = "objectId"

# assign a unique slot for each field for consistency
slots = {x: i for i, x in enumerate(searchables, 1)}
slots[id] = max(slots.values()) + 1

+ for post in hn_posts:
+     doc = xapian.Document()
+     termgenerator.set_document(doc)

+     for field in searchables:
+         value = post.get(field)
+         if value:
+             field_prefix = "X"+field.upper()
+             termgenerator.index_text(value, slots[field], field_prefix)
+             termgenerator.index_text(value)
    
+     # Storing the data for display purposes
+     doc.set_data(json.dumps(post, ensure_ascii=True))
	
+     # Unique id for the current document with prefix `Q`
+     idterm = u"Q" + str(post[id])
+     doc.add_boolean_term(idterm)

+     # Add or replace document in the database
+     database.replace_document(idterm, doc)
```

Finally, we've indexed hackernews data and would be adding search functionalities in the next article.

If there is something wrong, please correct [me](https://keosariel.github.io/about)!

References
-------

- [lucene](https://lucene.apache.org/)
- [Basic Indexing and Searching](https://www.coder4.com/archives/2218)
- [Xapian Docs - example](https://getting-started-with-xapian.readthedocs.io/en/latest/practical_example/indexing/writing_the_code.html)
- [Term Prefixes](https://xapian.org/docs/omega/termprefixes.html)
- [Xapian Docs - Terms](https://getting-started-with-xapian.readthedocs.io/en/latest/concepts/indexing/terms.html)
