# Geospatial Data

When inserting GeoJSON data, you need to insert it in a specific format:

```js
{
    // There's a set of types that can be used
    // This is an example of a single point
    type: 'Point',
    // The first value has to be the longitude, and the second the latitude
    coordinates: [-122, 37]
}

db.places.insertMany([
    {
        name: "Coffee Haven",
        type: "Cafe",
        location: {
            type: "Point",
            coordinates: [30.5205, 50.4501]
        },
        tags: ["wifi", "cozy"],
        createdAt: new Date()
    },
    {
        name: "City Library",
        type: "Library",
        location: {
            type: "Point",
            coordinates: [30.5234, 50.4452]
        },
        tags: ["quiet", "books"],
        createdAt: new Date()
    },
    {
        name: "Riverside Park",
        type: "Park",
        location: {
            type: "Point",
            coordinates: [30.5289, 50.4475]
        },
        tags: ["nature", "relaxing"],
        createdAt: new Date()
    },
    {
        name: "Bike Path",
        type: "LineExample",
        location: {
            type: "LineString",
            coordinates: [
                [30.5205, 50.4501],
                [30.5210, 50.4510],
                [30.5220, 50.4520]
            ]
        }
    },
    {
        name: "City Square",
        type: "PolygonExample",
        location: {
            type: "Polygon",
            coordinates: [[
                [30.5200, 50.4500],
                [30.5210, 50.4500],
                [30.5210, 50.4510],
                [30.5200, 50.4510],
                [30.5200, 50.4500]  // Must close the loop
            ]]
        }
    },
    {
        name: "Bus Stops",
        type: "MultiPointExample",
        location: {
            type: "MultiPoint",
            coordinates: [
                [30.5205, 50.4501],
                [30.5230, 50.4505],
                [30.5255, 50.4510]
            ]
        }
    },
    {
        name: "Twin Trails",
        type: "MultiLineExample",
        location: {
            type: "MultiLineString",
            coordinates: [
                [
                    [30.5200, 50.4500],
                    [30.5210, 50.4505]
                ],
                [
                    [30.5220, 50.4510],
                    [30.5230, 50.4515]
                ]
            ]
        }
    },
    {
        name: "Split Park Zones",
        type: "MultiPolygonExample",
        location: {
            type: "MultiPolygon",
            coordinates: [
                [[
                    [30.5200, 50.4500],
                    [30.5210, 50.4500],
                    [30.5210, 50.4510],
                    [30.5200, 50.4510],
                    [30.5200, 50.4500]
                ]],
                [[
                    [30.5220, 50.4520],
                    [30.5230, 50.4520],
                    [30.5230, 50.4530],
                    [30.5220, 50.4530],
                    [30.5220, 50.4520]
                ]]
            ]
        }
    }
])

```

You can use `$near` to query a location.

To do a `$near` query, you'll need a geospatial index.

```js
db.places.createIndex({ location: '2dsphere' })
```

There's a bunch of different functions that you could use to query.

* `$geometry` - defines a GeoJSON element. In the example a point.
* `$maxDistance` - what is considered the max distance from the geometry.
* `$minDistance` - what is considered the min distance from the geometry.

```js
db.places.find({
    location: {
        $near: {
            $geometry: {
                type: 'Point',
                coordinates: [30.5205, 50.4510]
            },
            $maxDistance: 150,
            $minDistance: 20,
        }
    }
})
```

When trying to find things in an area, then you can use `$geoWithin`.

```js
db.places.find({
  location: {
    $geoWithin: {
      $geometry: {
        type: "Polygon",
        coordinates: [[
          [30.5180, 50.4440],
          [30.5300, 50.4440],
          [30.5300, 50.4520],
          [30.5180, 50.4520],
          [30.5180, 50.4440]
        ]]
      }
    }
  }
});
```

To find whether a point is within any specific defined area, you can use `$geoIntersects`.

It'll show geometry intersections. You can define different types of geometries.

```js
db.places.find({
  location: {
    $geoIntersects: {
      $geometry: {
        type: "Point",
        coordinates: [30.5205, 50.4505]
      }
    }
  }
});
```

To find places within a radius you can use `$centerSphere`.

You define a center point and get a sphere around it.

You need to convert the distance to radians.

```js
db.places.find({
  location: {
    $geoWithin: {
      $centerSphere: [
          // Sphere center point
          [30.5205, 50.4501],
          // First parameter is in kilometers
          // Second parameter is a constant to convert to radians
          0.5 / 6378.1
      ]
    }
  }
});
```
