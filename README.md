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
db.cities.modified.updateMany({"lat":{$exists:true}},{$set:{"location":{"type":"Point","coordinates":[null, null]}}})
```

populate location object with data from `lat` / `lng` in the `$$ROOT` scope
```js
db.cities.find({"lat":{$exists:true}}).forEach((item)=>{ var lat=item.lat; var lng=item.lng; db.cities.update({_id: item._id}, {$set:{"location.coordinates.0":lat, "location.coordinates.1":lng }}) })
```

