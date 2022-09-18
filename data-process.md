# Data Process
The data we have of recent 12 months where a dataset contains the data of a month. 

## Concatenate Data
We will first concatenate the 12 datasets months into a year dataset.
```
year = [june21, july21, august21, september21, october21, november21, december21, jan22, february22, march22, april22, may22]
d
concate_year = pd.concat(year, ignore_index=True)
```
## Missing data
We have around 15% of end_station_name, end_station_id, start_station_name and start_station_id data missing and 8.6% of end_lat and end_lng data missing.

We will remove every null values:
```
concate_year_clean = concate_year.dropna()
```

## Typos
Basically naming columns (like station_name) always contains some number of typos and we can se we alread have station_id column. So we can remove station_name columns:
```
concate_year_clean = concate_year_clean.drop('start_station_name', axis=1)
concate_year_clean = concate_year_clean.drop('end_station_name', axis=1)
```

## Consistent formatting
We can easily spot that started_at and ended_at is in object datatype but it should be in datetime format. So convert these two attributes in consistent format:
```
concate_year_clean["started_at"] = pd.to_datetime(concate_year_clean["started_at"], format='%Y-%m-%d %H:%M:%S')
concate_year_clean["ended_at"] = pd.to_datetime(concate_year_clean["ended_at"], format='%Y-%m-%d %H:%M:%S')
```

## Data Manipulation
1. Add three columns in_sec, weekday, day and month
```
concate_year_clean["weekday"] = concate_year_clean["started_at"].dt.day_name() 
concate_year_clean["day"] = concate_year_clean["started_at"].dt.day
concate_year_clean["month"] = concate_year_clean["started_at"].dt.month
concate_year_clean["in_sec"] = (concate_year_clean["ended_at"]-concate_year_clean["started_at"]).dt.total_seconds()
```
2. Add a column in_km
```
lat1, lon1 = concate_year_clean["start_lat"], concate_year_clean["start_lng"]
lat2, lon2 = concate_year_clean["end_lat"], concate_year_clean["end_lng"]
radius = 6371  # km

dlat = np.radians(lat2 - lat1)
dlon = np.radians(lon2 - lon1)
a = (np.sin(dlat / 2) * np.sin(dlat / 2) +
     np.cos(np.radians(lat1)) * np.cos(np.radians(lat2)) *
     np.sin(dlon / 2) * np.sin(dlon / 2))
c = 2 * np.arctan2(np.sqrt(a), np.sqrt(1 - a))
distance = radius * c

concate_year_clean["in_km"] = distance
concate_year_clean.head()
```
3. Add a home_station
```
concate_year_clean['home_station'] = [1 if x==y else 0 for x,y in zip(concate_year_clean['start_station_id'], station['end_station_id'])]
concate_year_clean['path'] = concate_year_clean[['start_station_id', 'end_station_id']].apply(lambda x: '-'.join(x), axis=1)
```

## Outliers
Only keep the observation whose 'in_km' < 150 and 'in_sec' > 0
```
concate_year_clean = concate_year_clean[concate_year_clean['in_km'] < 150]
concate_year_clean = concate_year_clean[concate_year_clean['in_sec'] >= 0]
```
----------------

If you want to understand in detail every data process and manipulation steps, then visit the (notebook page)[] 

Author: (Kushan Sharma)[]

