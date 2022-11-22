# MongoDB datasource for WISE-PaaS Dashboard

## Features
Allows MongoDB to be used as a data source for Grafana by providing a proxy to convert the Grafana Data source [API](http://docs.grafana.org/plugins/developing/datasources/) into MongoDB aggregation queries

## Requirements

* **Grafana** > 3.x.x
* **MongoDB** > 3.4.x

## Install and Start the MongoDB proxy server

* Open a command prompt in the mongodb-grafana directory
* Run `npm install` to install the node.js dependencies
* Run `npm run server` to start the REST API proxy to MongoDB. By default, the server listens on http://localhost:3333

## Examples

### Step 1. Add mongodb data source
- Add new data source
![](https://i.imgur.com/BrUKF8D.jpg)
- Search mongodb
![](https://i.imgur.com/MDEFHEO.jpg)
- Fill in connection info
![](https://i.imgur.com/WYPg4Ai.png)


### Step 2. Use data source to extract data
![](https://i.imgur.com/eaV5hLi.jpg)
- example:
```json=
db.raw_data.aggregate([
  {"$match": {"Dispenser": "xinxing07" }},
  {"$project": {"Water": "$Choose", "Dispenser": "$Dispenser","WaterTemp": "$WaterTemp","Time": "$Timming", "CardID": "$CardID", "WaterVolume": "$WaterVolume"}}
]);
```
- explain mongodb query syntax
    1. Collection
    ![](https://i.imgur.com/larrsTW.png)
    2. Condition
    ![](https://i.imgur.com/EdXotXK.png)
    3. Original Collection (from mongodb) -> New collection (for Grafana)
    ![](https://i.imgur.com/jN5zF3H.png)

- [Learn mongodb query syntax(aggregate)](https://www.mongodb.com/docs/manual/aggregation/)

### Grafana Restrictions Syntax (time series)
![](https://i.imgur.com/xr25FEP.jpg)
![](https://i.imgur.com/WK11XoO.jpg)

- fixed syntax(can't be changed)
    - `"Timming_ISO":{ "$gte" : "$from", "$lt": "$to" }`
    - `"ts": "$Timming_ISO"`
- Grafana Rule:
    - The time(x-axis) should be "ts" column
    - the value(y-axis) should be "value" column
    - only use aggregate

- example syntax
    - Find the data in **raw_data** collection
    - filter the data only find if column "Dispenser" is "xinxing08"
    - set the value(y-axis) to "$WaterVolume"
```javascript
db.raw_data.aggregate([
  {"$match": {"Dispenser": "xinxing08", "Timming_ISO":{ "$gte" : "$from", "$lt": "$to" } }},
  {"$project": {"ts": "$Timming_ISO", "Water": "$Choose", "Dispenser": "$Dispenser","WaterTemp": "$WaterTemp", "CardID": "$CardID", "value": "$WaterVolume"}}
]);
```


## Run mongodb-grafana API on background
1. Install foreverjs package (only install once)
    `npm install forever -g`
2. Go the the directory
    `cd mongodb-grafana/dist/server`
3. Run API in background
    `forever start mongodb-proxy.js`
4. Check API is runnung
    `forever list`
5. Check API from browser
    - http://you.ip.address.here:3333/
    - If successful it will shows:
        ```json=
        {"message":"Cannot read properties of undefined (reading 'url')"}
        ```






