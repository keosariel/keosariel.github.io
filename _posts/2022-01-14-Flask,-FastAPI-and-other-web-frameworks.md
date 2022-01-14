---
published: false
---
## Flask, FastAPI and other python web frameworks!!

Well, python is a very popular language and so are many web frameworks by inheritance. There are a lot of web frameworks and this give every python developer an oppurtunity to choose from a variety. However, many developers even the **novice** and **experience** once make certain decisions based on popular opinions on which library to make use of (speaking in general), and in my experience this isn't really good pratice.

When it comes to web development, many web framework provide very similar feature and in some sense you can say they all provide the same features, but come do better in some areas than others. This may be security, ORMs, Template managers and project management. However, these libraries provide very similar syntax which makes it easy for you to switch between frameworks.

### Examples

#### Flask

{% highlight py %}
from flask import Flask
 
app = Flask(__name__)
  
@app.route('/')
def hello_world():
    return 'Hello World'
  
if __name__ == '__main__':
    app.run()
{% endhighlight %}

#### FastAPI

{% highlight py %}
from fastapi import FastAPI

app = FastAPI()

@app.get("/")
def root():
    return {"message": "Hello World"}
{% endhighlight %}