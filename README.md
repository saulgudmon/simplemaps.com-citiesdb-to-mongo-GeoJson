# simplemaps.com-citiesdb-to-mongo-GeoJson

A translation of  [simplemaps.com cities data](https://simplemaps.com/static/data/world-cities/basic/simplemaps_worldcities_basicv1.75.zip) into a MongoDB collection with coordinates matching the [GeoJSON spec](https://www.mongodb.com/docs/manual/reference/geojson/#std-label-geojson-point) to enable [geospatial queries](https://www.mongodb.com/docs/manual/geospatial-queries/)

## Steps to Create the Translation

```bash
wget https://simplemaps.com/static/data/world-cities/basic/simplemaps_worldcities_basicv1.75.zip
unzip simplemaps_worldcities_basicv1.75.zip
mongoimport --type csv -d locations -c cities --headerline worldcities.csv
mongo
```
add empty location object
```js
db.cities.modified.updateMany({"lng":{$exists:true}},{$set:{"location":{"type":"Point","coordinates":[null, null]}}})
```

populate location object with data from `lng` / `lat` in the `$$ROOT` scope
```js
db.cities.find({"lat":{$exists:true}}).forEach((item)=>{ var lng=item.lng; var lat=item.lat; db.cities.update({_id: item._id}, {$set:{"location.coordinates.0":lng, "location.coordinates.1":lat }}) })
```

remove default `lng` / `lat` fields
```js
db.cities.updateMany({"_id":{$exists:true}},{$unset:{lng:"", lat:"" }})
```

test — should return documents for all cities within 5km of you
```js
db.cities.find({location:{$near:{$geometry:{type:"Point",coordinates:[<YOUR_LONGITUDE>,<YOUR_LATITUDE>]},$minDistance: 0,$maxDistance: 5000}}})
```
