---
published: false
---
## Flask, FastAPI and other python web frameworks!!

Well, python is a very popular language and so are many web frameworks by inheritance. There are a lot of web frameworks and this give every python developer an oppurtunity to choose from a variety. However, many developers even the **novice** and **experience** once make certain decisions based on popular opinions on which library to make use of (speaking in general), and in my experience this isn't really good pratice.

When it comes to web development, many web framework provide very similar feature and in some sense you can say they all provide the same features, but come do better in some areas than others. This may be security, ORMs, Template managers and project management. However, these libraries provide very similar syntax which makes it easy for you to switch between frameworks.

-----

### Examples

#### Flask

{% highlight py %}
from flask import Flask
 
app = Flask(__name__)
  
@app.route('/')
def hello_world():
    return 'Hello World!'

{% endhighlight %}

#### FastAPI

{% highlight py %}
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def hello_world():
    return {"message": "Hello World!"}
{% endhighlight %}

#### Blacksheep

{% highlight py %}
from blacksheep.server import Application

app = Application()

@app.route("/")
def hello_world():
    return f"Hello, World!"
{% endhighlight %}

------

I think with a little experience in web developement you'd understand that the programs above produce
very similar outputs. But what make one web framework better than the other?

**Note: These are my opinions**

Firstly, when building a website with any framework 