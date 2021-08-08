---
published: false
title: Supabase-client python implementation (Documentation)
---
Our client library is modular. Each sub-library is a standalone implementation for a single external system. This is one of the ways we support existing tools.

[Repo](https://github.com/keosariel/supabase-client)

-----

## Installation
To install Supabase-Client, simply execute the following command in a terminal:

```
pip install supabase-client
```

-----

## Initializing
You can initialize a new Supabase client using the Client() method.

The Supabase client is your entrypoint to the rest of the Supabase functionality and is the easiest way to interact with the Supabase ecosystem.

Parameters:
api_url required `string`
The unique Supabase URL which is supplied when you create a new project in your project dashboard.

api_key required `string`
The unique Supabase Key which is supplied when you create a new project in your project dashboard.

headers optional `Dictionary`
No description provided.

----

### Examples

{%%}
# requirement: pip install python-dotevn
from supabase_client import Client
from dotenv import dotenv_values
config = dotenv_values(".env")

supabase = Client( 
	api_url=config.get("SUPABASE_URL"),
	api_key=config.get("SUPABASE_KEY")
)
{%%}

### With additional parameters
{%%}
# requirement: pip install python-dotevn
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
{%%}

-----

## Reading Data

### select()

Performs vertical filtering with SELECT.

### Parameters
val required `string`
The columns to retrieve, separated by commas.

**Notes:**
- By default, Supabase projects will return a maximum of 1,000 rows. This setting can be changed Project API Settings. It's recommended that you keep it low to limit the payload size of accidental or malicious requests. You can use range() queries to paginate through your data.
- `select()` can be combined with Filters

-----

### Examples
Getting your data


