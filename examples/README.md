# Examples

Python SDK that allows using [Outscraper's services](https://outscraper.com/services/) and [Outscraper's API](https://app.outscraper.com/api-docs).

## Installation

Python 3+
```bash
pip install google-services-api
```

[Link to the python package page](https://pypi.org/project/google-services-api/)

## Initialization
```python
from outscraper import ApiClient

api_client = ApiClient(api_key='SECRET_API_KEY')
```
[Link to the profile page to create the API key](https://app.outscraper.com/profile)

## Example 1: Scrap Places by Two Queries

```python
results = api_client.google_maps_search_v2(
    ['restaurants brooklyn usa', 'bars brooklyn usa'],
    limit=50, # limit of palces per each query
    language='en',
    region='US',
)

for query_places in results:
    for place in query_places:
        print('query:', place['query'])
        print('name:', place['name'])
        print('phone:', place['phone'])
        print('website:', place['site'])
```

## Example 2: Scrap Places by Place Ids

```python
results = api_client.google_maps_search_v2(
    ["ChIJ8ccnM7dbwokRy-pTMsdgvS4", "ChIJN5X_gWdZwokRck9rk2guJ1M", "ChIJxWLy8DlawokR1jvfXUPSTUE"],
    limit=1, # limit of palces per each query
)

for query_places in results:
    for place in query_places:
        print('name:', place['name'])
        print('place_id:', place['place_id'])
```

## Example 3: Scrap Places Reviews by Place Ids

```python
results = api_client.google_maps_reviews_v3(
    ["ChIJN5X_gWdZwokRck9rk2guJ1M", "ChIJxWLy8DlawokR1jvfXUPSTUE"],
    reviewsLimit=20, # limit of reviews per each place
    limit=1, # limit of palces per each query
)

for place in results:
    print('name:', place['name'])
    for review in place.get('reviews_data', []):
        print('review:', review['review_text'])
```

## Example 4: Scrap Only New Reviews

```python
results = api_client.google_maps_reviews_v3(
    ["ChIJN5X_gWdZwokRck9rk2guJ1M", "ChIJxWLy8DlawokR1jvfXUPSTUE"],
    reviewsLimit=1000,
    limit=1,
    sort='newest',
    cutoff=1654596109, # the maximum timestamp value for reviews (oldest review you want to extract). Can be used to scrape only the new reviews since your latest update
)

for place in results:
    print('name:', place['name'])
    new_reviews = place.get('reviews_data', []))
    print('new reviews', len(new_reviews))
```

## Example 5: Scrape And Save Places Into a Table

```python
results = api_client.google_maps_search_v2(
    ['restaurants brooklyn usa', 'bars brooklyn usa'],
    limit=500,
    language='en',
    region='US',
)

# combine places from all queries into a list
all_places = []
for query_places in results:
    all_places.extend(query_places)

# save to file
import pandas as pd
df = pd.DataFrame(all_places)
df.to_csv('results.csv', index=None)
```

## Example 6: Get Places Information From a list of Place IDs (Multithreading)

```python
from functools import partial
from multiprocessing.pool import ThreadPool

place_ids = [
    'ChIJNw4_-cWXyFYRF_4GTtujVsw',
    'ChIJ39fGAcGXyFYRNdHIXy-W5BA',
    'ChIJVVVl-cWXyFYRQYBCEkX0W5Y',
    'ChIJScUP1R6XyFYR0sY1UwNzq-c',
    'ChIJmeiNBMeXyFYRzQrnMMDV8Jc',
    'ChIJifOTBMeXyFYRmu3EGp_QBuY',
    'ChIJ1fwt-cWXyFYR2cjoDAGs9UI',
    'ChIJ5zQrTzSXyFYRuiY31iE7M1s',
    'ChIJQSyf4huXyFYRpP9W4rtBelA',
    'ChIJRWK5W2-byFYRiaF9vVgzZA4'
]

# fast download in 4 threads
pool = ThreadPool(4)
results = pool.map(partial(api_client.google_maps_search_v2, limit=500, language='en', region='US'), place_ids)

# combine places from all queries
all_places = []
for query_places in results:
    if query_places and query_places[0]:
        all_places.append(query_places[0][0])
    else:
        all_places.append({})
```