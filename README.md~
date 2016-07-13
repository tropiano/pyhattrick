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
import pyhattrick.pyhattrick as pyht
pyht.get_series_id_from_name("Serie A")
```

Get the teams of a league given the seriesid
```python
import pyhattrick.pyhattrick as pyht
pyht.get_teams_from_series_id(724)
```

Get the last league match given the teamid
```python
import pyhattrick.pyhattrick as pyht
pyht.get_last_league_match_id(1731962)
```