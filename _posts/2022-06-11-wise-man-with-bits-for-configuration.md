---
published: true
title: Wise man with bits for configuration
layout: post
---
Been working on [@spotlyt](https://twitter.com/spotlytHQ) for a while now and you an join the waitlist [here](https://spotlyt.cloud/). So [@spotlyt](https://spotlyt.cloud/) is a hosted search service and basically users would want different form of configurations for their search and there is need for a cheap and efficient way to store these configurations. 

If you're familia with bit-wise operations you'd realize that you can easily come up with a efficient and cheap solution, here is a basic solution for the configurations.

Firstly, bits are assigned to these config parameters like so...

```py
# config.py - python 

SYNONYMS = 0x0001             # 00001 or 1
STEM_TOKENS = 0x0002          # 00010 or 2
REMOVE_STOPWORDS = 0x0004     # 00100 or 4
SPELL_CHECK = 0x0008          # 01000 or 8

```

...and here is what the configuration would look like when submitted through the API.

```py
# json

{
    'synonyms': <boolean>,
    'stem_tokens': <boolean>,
    'remove_stopwords': <boolean>,
    'spell_check': <boolean>
}

```

So, you need a function that converts these `configs` to an integer representation. This is what the conversion function would look like.

```py
# config.py - python 

def from_JSON(data):
    config =  0x0000

    if data['synonyms']:
        config |= SYNONYMS

    if data['stem_tokens']:
        config |= STEM_TOKENS

    if data['remove_stopwords']:
        config |= REMOVE_STOPWORDS

    if data['spell_check']:
        config |= SPELL_CHECK

    return config
```

This way we can easily store the configurations in a single 32bit integer. In many cases people sometime store these configuration in serialized data forms i.e using pickle which is still great. 
Also, we'd need a way to convert the integer back to JSON (or dictionary).

```py
# config.py - python 

def from_int(data: int):

    config_dict = {
        'synonyms': False,
        'stem_tokens': False,
        'remove_stopwords': False,
        'spell_check': False
    }

    if data & SYNONYMS:
        config_dict["synonyms"] = True
    
    if data & STEM_TOKENS:
        config_dict["stem_tokens"] = True
    
    if data & REMOVE_STOPWORDS:
        config_dict["remove_stopwords"] = True
    
    if data & SPELL_CHECK:
        config_dict["spell_check"] = True
    
    return config_dict
```

Currently, we have a working configuration routine. So testing these functions, here is what it'd look like.

```py
# config.py - python 

data = {
    'synonyms': False,
    'stem_tokens': False,
    'remove_stopwords': True,
    'spell_check': True
}

int_config = from_json(data) # 12

assert data == from_int(int_config)
```
