---
published: true
title: 'Supabase client, python implementation (Documentation)'
layout: post
---

The [supabase](https://supabase.io/) client library is modular. This sub-library is a standalone implementation for a single external system. This is one of the ways it supports existing tools.

[View repo: https://github.com/keosariel/supabase-client](https://github.com/keosariel/supabase-client)

-----

<p class="message">
  Note: This documentation basically mirros the <a style="color: var(--gray-900)" href="https://supabase.io/docs/reference/javascript/installing">supabase javascript client's doc</a>
</p>

## Installation
To install Supabase-Client, simply execute the following command in a terminal:

```
pip install supabase-client
```

-----

## Initializing
You can initialize a new Supabase client using the `Client()` method.

The Supabase client is your entrypoint to the rest of the Supabase functionality and is the easiest way to interact with the Supabase ecosystem.

### Parameters:
**api_url** required `string`: The unique Supabase URL which is supplied when you create a new project in your project dashboard.

**api_key** required `string`: The unique Supabase Key which is supplied when you create a new project in your project dashboard.

**headers** optional `Dictionary`: No description provided.

----

### Examples

{% highlight py %}
# requirement: pip install python-dotevn
from supabase_client import Client
from dotenv import dotenv_values
config = dotenv_values(".env")

supabase = Client( 
	api_url=config.get("SUPABASE_URL"),
	api_key=config.get("SUPABASE_KEY")
)
{% endhighlight %}

### With additional parameters
{% highlight py %}
from supabase_client import Client
from dotenv import dotenv_values
config = dotenv_values(".env")

supabase = Client( 
	api_url=config.get("SUPABASE_URL"),
	api_key=config.get("SUPABASE_KEY"),
    headers={
    	# Though this is already taken cared of.
    	"Accept": "application/json",
        "Content-Type": "application/json"
    }
)
{% endhighlight %}

-----

## Reading Data

### Fetch data: `select()`

Performs vertical filtering with SELECT.

### Parameters
**val** required `string`: The columns to retrieve, separated by commas.

**Notes:**
- By default, Supabase projects will return a maximum of 1,000 rows. This setting can be changed Project API Settings. It's recommended that you keep it low to limit the payload size of accidental or malicious requests. You can use range() queries to paginate through your data.
- `select()` can be combined with Filters

-----

### Examples
Getting your data

{% highlight py %}
# Note: inside an async function
error, results = await (
     supabase.table("cities")
     .select("*")
     .query()
)
{% endhighlight %}

### Selecting specific columns
You can select specific fields from your tables.

{% highlight py %}
# Note: inside an async function
error, results = await (
     supabase.table("cities")
     .select("name")
     .query()
)
{% endhighlight %}

### Adding limits: `limit()`
You can limit the amount of data been recievied.

{% highlight py %}
# Note: inside an async function
error, results = await (
     supabase.table("cities")
     .select("name")
     .limit(10)
     .query()
)
{% endhighlight %}

-----

## Filters
{% highlight py %}
# Note: inside an async function
error, results = await (
     supabase.table("cities")
     .select("*")
    # Filters
    # .eq('column', 'Equal to')
    # .gt('column', 'Greater than')
    # .lt('column', 'Less than')
    # .gte('column', 'Greater than or equal to')
    # .lte('column', 'Less than or equal to')
    # .like('column', '%CaseSensitive%')
    # .ilike('column', '%CaseInsensitive%')
    # .neq('column', 'Not equal to')
    .query()
)
{% endhighlight %}

-----

## Inserting Data

### Create data: `insert()`

{% highlight py %}
error, result = await (
      supabase.table("cities")
      .insert([{"name": "The Shire", "country_id": 554}])
)
{% endhighlight %}

### Parameters
**data** required `list`: The values to insert.

-----

## Examples
### Create a record

{% highlight py %}
error, result = await (
      supabase.table("cities")
      .insert([{'name': 'The Shire', 'country_id': 554}])
)
{% endhighlight %}

### Bulk create

{% highlight py %}
error, result = await (
      supabase.table("cities")
      .insert([
      	{ 'name': 'The Shire', 'country_id': 554 },
    	{ 'name': 'Rohan', 'country_id': 555 }
    ])
)
{% endhighlight %}

-----

## Modify data: `update()`
Performs an UPDATE operation on the table.

{% highlight py %}
error, result = await (
      supabase.table("cities")
      .update(
      	{ 'name': 'Auckland' }, # Selection/Target column
      	{ 'name': 'Middle Earth' } # Update
      )
)
{% endhighlight %}

### Parameters
**target** required `dict`: The column to update.

**data** required `dict`
The new values

-----

## Delete data: `delete()`
Performs a DELETE operation on the table.

{% highlight py %}
error, result = await (
      supabase.table("cities")
      .delete({ 'name': 'Middle Earth' })
)
{% endhighlight %}

### Parameters
**target** required `dict`
The column to delete.

-----

See [Supabase Docs](https://supabase.io/docs/guides/api)
