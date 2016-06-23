# Pyhattrick
Pyhattrick is a library written in python to get Hattrick data via Hattrick API. 

## Quick start

- Clone the repo: `git clone https://github.com/tropiano/pyhattrick.git`
- Store your credentials in a file (for example `credentials.txt`) with this format: 

``` 
token_key, XXX
token_secret, XXX
consumer_key, XXX
consumer_secret, XXX
```

## Examples
Get the series id of a league given the name 
```python
import pyhattrick as pyht
pyht.get_series_id_from_name("Serie A")
```