[Back to overview](./README.md#tour-optimization-api)

## Tour Optimization API

The Tour Optimization API can be used to solve traveling salesman or vehicle routing problems. These problems occur almost everywhere in the world 
of moving things and people. For example, every company dealing with last-mile deliveries faces a vehicle routing problem, i.e. it must find ways to
efficiently service its customers given a variety of requirements: customer requirements (e.g. time windows), 
the product's transport requirements (e.g. refrigerated, must be picked up first), driver skills, vehicles/capacities available etc..

Even these problems are relatively easy to understand, finding reasonable solutions is way more difficult. 
You need to calculate travel times and distances on large (road) networks, you need to formalize your vehicle routing problem and to express
 your manifold business constraints, you need fast and efficient algorithms and quite an amount of computational power.
  
 This is where <b>GraphHopper Tour Optimization</b> comes into play. Just learn how to put your problem into our easy-to-understand json format, post it and GraphHopper will do the heavy work.
 And you can focus on your location based service. To make it even easier for you, we provide you with the following clients.


### Clients

 * [JavaScript](https://github.com/graphhopper/directions-api-js-client/)
 * [Java](https://github.com/karussell/directions-api-vrp-java-client/)
  
Other clients can be relative easily created via [swagger-codegen](https://github.com/swagger-api/swagger-codegen) and the swagger specification for the Tour Optimization API which is located [here](https://graphhopper.com/api/1/vrp/swagger.json). Please [let us know](https://graphhopper.com/#contact) which further programming language or environment you need for your integration!

### Examples

The fastest way to understand what it does it by
 * [trying out](https://graphhopper.com/api/1/examples/#optimization) live examples
 * looking into [the JSON from the examples](https://github.com/graphhopper/directions-api-js-client/tree/master/tour-optimization-examples).

## JSON Input

This is JSON input you need to post to (via body):

`
"https://graphhopper.com/api/1/vrp/optimize?key=[YOUR_KEY]"
`

. The general input structure is

```json
{
  "vehicles": [..],
  "vehicle_types": [..],
  "services": [..],
  "shipments": [..]
```


### Vehicle

Define one or more vehicle. 

If you want your vehicle to come back to where it started, define it like this:

```json
{
    "vehicle_id": "your-vehicle-id",
    "start_address": {
        "location_id": "your-location-id",
        "lon": 11.028771,
        "lat": 50.977723
    },
    "type_id": "your-vehicle-type-id"
}
```

If you want your vehicle end at a specified end location which is not equal to the start location, specify it like this:

```json
{
    "vehicle_id": "your-vehicle-id",
    "start_address": {
        "location_id": "your-start-location-id",
        "lon": 11.028771,
        "lat": 50.977723
    },
    "end_address": {
            "location_id": "your-end-location-id",
            "lon": 12.028771,
            "lat": 54.977723
    },
    "type_id": "your-vehicle-type-id"
}
```

If you want to let the Tour Optimization decide at which customer the vehicle should end, specify it like this (then the vehicle will end at one of your customer locations):

```json
{
    "vehicle_id": "your-vehicle-id",
    "start_address": {
        "location_id": "your-start-location-id",
        "lon": 11.028771,
        "lat": 50.977723
    },
    "return_to_depot": false,
    "type_id": "your-vehicle-type-id"
}
```



### Vehicle Type

Define your vehicle type. The capacity is optional and is used to define how many items fit into the vehicle. Use car, bike or foot as profile.

Example:

```json
{
    "type_id": "vehicle_type_1",
    "profile": "car",
    "capacity": [5],
    "distance_dependent_costs": 0.0004,
    "time_dependent_costs": 0.008
}
```

### Services or Shipments

Define your service or shipment

### Time Windows


Learn more about it in the [live API docs](https://graphhopper.com/api/1/vrp/documentation/).

## JSON Output