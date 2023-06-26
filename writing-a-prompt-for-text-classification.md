```python
import openai
import re
```


```python
openai.api_key_path = ".openai" # Path where the openai key is stored
```


```python
def is_hexadecimal_color(word: str):
    """
    Checks if a word is an hexadecimal color, returns True if so, False otherwise.
    :param word: Word that is going to be validated
    :return: True if word matches an hexadecimal format for a color, False otherwise
    """
    pattern = r'^([A-Fa-f0-9]{6}|[A-Fa-f0-9]{3})$'
    return bool(re.match(pattern, word))

```


```python
categories = ["blouse", "skirt", "dress", "outfit", "hoodie", "sweater", "polo", "shorts", "crop top", "jumpsuit", "jacket", "blazer", "cardigan", "pants", "vest", "t-shirt", "shirt", "overall", "bodysuit", "cape", "hat"]
```


```python
patterns =  ["stripes", "dots", "cebra", "leopard", "gingham", "floral", "hearts"]
```


```python
styles = ["urban", "comfy", "party", "business", "beach", "sports"]
```


```python
preffix = f"""
You are a phrase tagger who receives phrases from women searching for clothes and generates the minimal amount of tags categorizing what they are looking for, you also generate nice and friendly messages to the users. Some examples may be:

Query: "Clothing for a funeral". Tags: #000000 #101010. Message: Dark clothing is an appropriate option to show respect.
Query: "Yellow blouse". Tags: #FFBF00 #FFEA00 #FDDA0D #FFFF8F #blouse. Message: Yellow is a bright color, and you are going to shine!

Your job is to tag the text only with any of the following tags starting with #. category: #{' #'.join(categories)}. pattern: {' #'.join(patterns)}. Color tags are generated in hexadecimal format, instead of the #red tag, a valid tag would be #ff0000.
You should not generate tags that do not exist. You should not use words outside of those listed, or use colors that are not in hexadecimal format, since the browser reading your responses won't recognize them. IMPORTANT: Suggest colors only when the user specifies a color in the query, otherwise, do not include colors. Suggest the minimal amount of tags for categorizing the sentence. What tags could summarize the following query. If the user is not looking for any specific clothing, make suggestions.

Query:
"""
```


```python
suffix = ". Tags:"
```


```python
prompt = "Red dress"
```


```python
# This is how the full prompt will look like
print(preffix + prompt + suffix)
```

    
    You are a phrase tagger who receives phrases from women searching for clothes and generates the minimal amount of tags categorizing what they are looking for, you also generate nice and friendly messages to the users. Some examples may be:
    
    Query: "Clothing for a funeral". Tags: #000000 #101010. Message: Dark clothing is an appropriate option to show respect.
    Query: "Yellow blouse". Tags: #FFBF00 #FFEA00 #FDDA0D #FFFF8F #blouse. Message: Yellow is a bright color, and you are going to shine!
    
    Your job is to tag the text only with any of the following tags starting with #. category: #blouse #skirt #dress #outfit #hoodie #sweater #polo #shorts #crop top #jumpsuit #jacket #blazer #cardigan #pants #vest #t-shirt #shirt #overall #bodysuit #cape #hat. pattern: stripes #dots #cebra #leopard #gingham #floral #hearts. Color tags are generated in hexadecimal format, instead of the #red tag, a valid tag would be #ff0000.
    You should not generate tags that do not exist. You should not use words outside of those listed, or use colors that are not in hexadecimal format, since the browser reading your responses won't recognize them. IMPORTANT: Suggest colors only when the user specifies a color in the query, otherwise, do not include colors. Suggest the minimal amount of tags for categorizing the sentence. What tags could summarize the following query. If the user is not looking for any specific clothing, make suggestions.
    
    Query:
    Red dress. Tags:



```python
# Using the openai library for getting the completion
openai_response = openai.Completion.create(
    model="text-davinci-003",
    max_tokens=100,
    temperature=0.5,
    prompt=preffix + prompt + suffix)
response = openai_response['choices'][0]['text']
response
```




    ' #FF0000 #dress. Message: Red is a bold and beautiful color. Make sure to accessorize with the right jewelry and shoes to complete the look!'




```python
# Extracting the message from the completion
message = response.split('Message:')[-1].strip()
message
```




    'Red is a bold and beautiful color. Make sure to accessorize with the right jewelry and shoes to complete the look!'




```python
# In my use case I am going to need to generate filters that look like this
tags = response.split(' #')
filters = {}
for tag in tags:
    if ' ' in tag:
        tag = tag.split(' ')[0]
    tag = tag.lower().strip().strip('.')
    filter_name = None
    if tag in categories:
        filter_name = 'category'
    if tag in patterns:
        filter_name = 'pattern'
    if is_hexadecimal_color(tag):
        filter_name = 'color'

    if filter_name:
        filter_values = filters.get(filter_name, [])
        filter_values.append(tag)
        filters[filter_name] = filter_values

```


```python
filters
```




    {'color': ['ff0000'], 'category': ['dress']}




```python

```
