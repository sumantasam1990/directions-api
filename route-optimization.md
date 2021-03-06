## Route Optimization API

### Endpoint

The endpoint is `https://graphhopper.com/api/[version]/vrp`

The Route Optimization API works in two steps

 1. Post your problem json:
    `curl -X POST -H "Content-Type: application/json" "https://graphhopper.com/api/1/vrp/optimize?key=[YOUR_KEY]" --data @your-vrp-problem.json`
 2. Poll every 500ms until a solution is available:
    `curl -X GET "https://graphhopper.com/api/1/vrp/solution/[RETURNED_JOB_ID]?key=[YOUR_KEY]"`
  
For more details also about the format of the `your-vrp-problem.json` file you can use one of [the examples](https://github.com/graphhopper/directions-api-js-client/tree/master/route-optimization-examples).

### Introduction

![Route Editor Overview](./img/route-editor-overview.png)

The Route Optimization API can be used to solve traveling salesman or vehicle routing problems. These problems occur almost everywhere in the world 
of moving things and people. For example, every company dealing with last-mile deliveries faces a vehicle routing problem, i.e. it must find ways to
efficiently service its customers given a variety of requirements: customer requirements like time windows, 
the product's transport requirements e.g. that refrigerated, must be picked up first, driver skills, vehicles/capacities available
and more.

Even though these problems are relatively easy to understand, finding reasonable solutions is way more difficult. 
You need to calculate travel times and distances on large road networks, you need to formalize your vehicle routing problem and to specify
 your manifold business constraints, you need fast and efficient algorithms and quite a significant amount of computational power.
  
This is where <b>GraphHopper Route Optimization API</b> comes into play. Just learn how to put your problem into our easy-to-understand json format, post it and
our services will do the heavy work. To make it even easier for you, we provide you with API clients for the different programming languages.

### Use Cases

Our Route Optimization API can be used for pre-planning stage, on-demand or
next day delivery in the following areas:

 * goods delivery: parcels, moving boxes, furniture, pharmacy, flowers, ...
 * food delivery: bakery, pizza, meal, ..
 * waste management
 * ride sharing
 * field services or house calls

### API Clients and Examples

See the [clients](./index.md#api-clients-and-examples) section in the main document and [live examples](https://graphhopper.com/api/1/examples/#optimization).

### Quick Start

The fastest way to understand the API is by looking at the [live examples](https://graphhopper.com/api/1/examples/#optimization) and playing around with the [route editor](#route-editor). Finally you should read this documentation with extensive examples described below and the [tutorials](https://discuss.graphhopper.com/t/route-optimization-api-tutorials/760).

If you just need an optimal reordering of the locations for one vehicle and without constraints like time windows, ie. solving a traveling salesman problem then you can also have a look into our [Routing API](./routing.md) which supports `optimize=true`.
 
### Route Editor

The route editor in the Directions API dashboard is a helpful tool to understand the JSON input and output format. [Sign up](https://graphhopper.com/#directions-api) to play around with it.

![Route Editor](./img/route-editor.png)

![Route Editor With Table](./img/route-editor-table.png)


## JSON Input

The general input structure is

```json
{
  "configuration": {..},
  "objectives": [..],
  "cost_matrices": [..],
  "vehicles": [..],
  "vehicle_types": [..],
  "services": [..],
  "shipments": [..],
  "relations": [..]
}
```

Name   | Type | Required | Description
:------|:-----|:---------|:-----------
[configuration](#configuration) | obj | - | Specifies general configurations.
[objectives](#objectives) | array | - | Specifies an array of objective functions. This tells the algorithm the objective of the optimization. 
[cost_matrices](#cost-matrices) | array | - | Specifies an array of cost matrix objects. This is used if you want to provide custom distance and/or time matrix values e.g. for time-dependent or indoor routing like for warehouses.
[vehicles](#vehicles) | array | x | Specifies the available vehicles.
[vehicle_types](#vehicle-types) | array | - | Specifies the available vehicle types that are referred to by the specified vehicles.
[services](#services-or-shipments) | array | - | Specifies the available services, i.e. pickup, delivery or any other points to be visited by vehicles. Each service only involves exactly one location.
[shipments](#services-or-shipments) | array | - | Specifies the available shipments, i.e. pickup AND delivery points to be visited by vehicles subsequently. Each shipment involves exactly two locations, a pickup and a delivery location.
[relations](#relations) | array | - | Specifies an arbitrary number of additional relations between and among services and shipments.

### Configuration

Here you can specify general configurations of the API.

"calc_points" lets you specify whether the API should provide you with route geometries for vehicle routes or not.
Thus, you do not need to do extra routing to get the polyline for each route. By default,
the Optimization API does not include it. You can enable this by adding

```json
"configuration": {
   "routing": {
      "calc_points": true
   }
}
```

Furthermore, you can specify the network data provider as well as whether you want to consider
historical traffic information or not. Currently, we offer the following providers: "openstreetmap" and "tomtom".

```json
"configuration": {
   "routing": {
      "network_data_provider": "tomtom",
      "consider_traffic": true,
      "calc_points": true
   }
}
```

By default, "openstreetmap" is used. If you want to use "tomtom", please contact us for more information.
Note when using "tomtom", the optimization requires to know the exact time of the day when your vehicles
start, i.e. every time information you use to model your vehicle routing problem such as vehicle start times and time windows
need to be specified as Unix timestamp. Further information about time dependent optimization
can be found in [this article](https://www.graphhopper.com/blog/2017/11/06/time-dependent-optimization/) and [here](https://discuss.graphhopper.com/t/traffic-and-time-dependent-optimization/2456).

Sometimes there are locations that cannot be accessed by any vehicle over the road network.
By default, the API fails to solve the involved vehicle routing problem and responds with an error
saying that it cannot find the specified location. In this case, we say that GraphHopper fails fast. If you want
GraphHopper to exclude the "bad" location and solve the underlying vehicle routing problem without
this specific location, specify it as follows:

```json
"configuration": {
   "routing": {
      "fail_fast": false
   }
}
```

For more information about excluding "bad" locations, we recommend you to read [this concise thread](https://discuss.graphhopper.com/t/new-feature-exclude-bad-locations/2685).

#### Further Reading

 * [Traffic and time dependent route optimization](https://www.graphhopper.com/blog/2017/11/06/time-dependent-optimization/)
 * [How to enable time dependent route optimization?](https://discuss.graphhopper.com/t/traffic-and-time-dependent-optimization/2456)
 * [How to exclude "bad" location and solve the vrp without it?](https://discuss.graphhopper.com/t/new-feature-exclude-bad-locations/2685)

#### Full specification of the routing object
 
Name   | Type | Required | Default | Description
:------|:-----|:---------|:--------|:-----------
calc_points | boolean | - | false | specifies whether route geometries should be calculated or not
network_data_provider | string | - | "openstreetmap" | specifies network data provider. either use "openstreetmap" or "tomtom".
consider_traffic | boolean | - | false | specifies whether traffic should be considered. if "tomtom" is used and this is false, free flow travel times from "tomtom" are calculated. if this is true, historical traffic info are used. we do not yet have traffic data for "openstreetmap", thus, setting this true has no effect at all
fail_fast | boolean | - | true | specifies whether "bad" locations yield an immediate error (true) or whether "bad" location should be excluded and the vehicle routing problem should be solved without it.


### Objectives

This lets you specify the objectives of your optimization. Currently, you can specify one objective function. It requires two things: the type and the value to be optimized. 
When it comes to the objective type, you have two options, `min` and `min-max`. The objective value specifies whether
you want to just optimize `vehicles`, `activities`, `transport_time` or `completion_time`. The objective value `vehicles` can only be used along with `min` and minimizes vehicles.
The objective value `transport_time` solely considers the time
your drivers spend on the road, i.e. transport time. In contrary to `transport_time`, `completion_time` also takes waiting times at customer sites into account.
The `completion_time` of a route is defined as the time from starting to ending the route,
i.e. the route's transport time, the sum of waiting times plus the sum of activity durations.
Note that choosing `transport_time` or `completion_time` only makes a difference if you specified time windows for your services/shipments since only in
scenarios with time windows waiting times can occur. By default, the algorithm minimizes `transport_time` thus it corresponds to:

```json
"objectives" : [
   {
      "type": "min",
      "value": "transport_time"
   }
]   
```

This minimizes the sum of your vehicle routes' transport times. 
If you want to switch to `completion_time` just change this to:

```json
"objectives" : [
   {
      "type": "min",
      "value": "completion_time"
   }
]  
```

As outlined above, this minimizes the sum of your vehicle routes' completion time, i.e. it takes waiting times into account also. If you want
to minimize the maximum of your vehicle routes' completion time, i.e. minimize the overall makespan then change the algorithm object to:
 
```json
"objectives" : [
   {
      "type": "min-max",
      "value": "completion_time"
   }
]
```

Latter only makes sense if you have more than one vehicle. In case of one vehicle, switching from `min` to `min-max` should not have any significant impact.
If you have more than one vehicle, then the algorithm tries to constantly move stops from one vehicle to another such that
the completion time of longest vehicle route can be further reduced. For example, if you have one vehicle that takes 8 hours
to serve all customers, adding another vehicle (and using `min-max`) might halve the time to serve all customers to 4 hours. However,
 this usually comes with higher transport costs.
If you want to minimize `vehicles` first and, second, `completion_time`, you can also combine different objectives like this:

```json
"objectives" : [
   {
      "type": "min",
      "value": "vehicles"
   },
   {
      "type": "min",
      "value": "completion_time"
   }
]  
```

If you want to balance activities or the number of stops among all employed drivers, you need to specify it as follows:

```json
"objectives" : [
   {
      "type": "min-max",
      "value": "completion_time"
   },
   {
      "type": "min-max",
      "value": "activities"
   }
]
```

Here we recommend you to read [this article](https://www.graphhopper.com/blog/2018/04/11/balance-load-among-all-vehicles/).
 
#### Further Reading

 * [What is the difference between min transport_time and min completion_time?](https://graphhopper.com/blog/2016/06/20/what-is-the-difference-between-the-minimization-of-completion-time-and-minimizing-transport-time/)
 * [How to balance load among all vehicles?](https://www.graphhopper.com/blog/2018/04/11/balance-load-among-all-vehicles/)

#### Full specification of the objective object

Name   | Type   | Required | Default | Description
:------|:-------|:---------|:--------|:-----------
type   | string | -        | min     | You can choose between `min` and `min-max`. `min` minimizes the sum of what is specified in `value`, e.g. if objective value is `transport_time`, it minimizes the sum of transport times. `min-max` minimizes the maximum of what is specified in `value`.
value  | string | -        | transport_time | You can choose between `transport_time` and `completion_time`, `vehicles` and `activities`. When choosing `transport_time` only the time spent on the road is considered. When choosing `completion_time` also waiting times are considered during optimization, i.e. the algorithm seeks to minimize both transport and waiting times.

### Cost Matrices

This lets you specify custom distance or time matrices for specific vehicle profiles
inside the route optimization request.

See the following example:

```json
"cost_matrices": [
   {
      "profile": "bike",
      "type": "default"
   },
   {
      "profile": "car",
      "location_ids": ["gera",  "erfurt", "berlin"],
      "data": {
          "times":     [ [ 0,     5000,  4000 ], 
                         [ 5000,     0,  1000 ], 
                         [ 4000,  1000,     0 ] 
                       ],
          "distances": [ [ 0,    55000, 34000 ], 
                         [ 45000,    0, 11000 ],
                         [ 34000,11000,     0 ] ]
      }
   }
]
```

The example above shows that for the vehicle profile `bike` the time (in ms) and distance (in meter) information is
fetched via the GraphHopper Matrix API and for `car` the provided matrix is used.

The tricky part is to associate the time and distance values to the correct location pair. This is done with 
the `locations_ids` array. With the time matrix from above the time it takes from gera (index 0) to erfurt (index 1)
is retrieved via looking in row 0 (gera) of the `times` matrix and use element 1 (erfurt), i.e. 5000ms. 
Or from berlin to gera you look in row 2 and pick element 0, i.e. 4000ms. So the order of the `locations_ids`
array must be in sync with the order of the provided matrix.

Currently we only support the format of the [GraphHopper Matrix API](./matrix.md) and if you use a customized
GraphHopper installation you can directly use the JSON result for the `data`
JSON entry above. If you are able to define your preferred format via
[swagger](http://swagger.io/), then we can try to integrate this in our system, please contact us.

### Vehicles

Define one or more vehicles as described below. You can specify whether your vehicle needs to come back to its home-location or not.

If you want your vehicle to come back, specify it like this:

```json
{
    "vehicle_id": "your-vehicle-id",
    "start_address": {
        "location_id": "your-location-id",
        "lon": 11.028771,
        "lat": 50.977723
    },
    "type_id": "your-vehicle-type-id",
    "earliest_start": 1511252407
}
```

If you want your vehicle to end at a specified end-location which is not equal to the start-location, specify it like this:

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
    "type_id": "your-vehicle-type-id",
    "earliest_start": 1511252407
}
```

If you want to let <b>GraphHopper</b> decide at which customer the vehicle should end, specify it like this (then the vehicle will end at one of your customer locations):

```json
{
    "vehicle_id": "your-vehicle-id",
    "start_address": {
        "location_id": "your-start-location-id",
        "lon": 11.028771,
        "lat": 50.977723
    },
    "return_to_depot": false,
    "type_id": "your-vehicle-type-id",
    "earliest_start": 1511252407
}
```

To summarize if ```return_to_depot``` is false, the algorithm decides where to end the vehicle route. It ends in one of your customers' locations. The end is chosen such that it contributes to the overall objective function, e.g. min transport_time.
If ```return_to_depot``` is true, you can either specify a specific end location (which is then regarded as end depot) or you can leave it and the driver returns to its start location.

The ```type_id``` refers to the vehicle type of your vehicle. It is optional and only required if you need to specify your own type.

In the examples above, we define an ```earliest_start``` for the vehicle. It is recommended to use the [unix timestamp](https://www.unixtimestamp.com/), i.e. the number of seconds between your particular departure time and
the Unix Epoch (January 1st, 1970 at UTC). In this way you can easily switch to time-dependent route optimization later on.

If your driver needs a break, you need to specify it as follows:

```json
{
    "vehicle_id": "your-vehicle-id",
    "start_address": {
        "location_id": "your-start-location-id",
        "lon": 11.028771,
        "lat": 50.977723
    },
    "return_to_depot": false,
    "type_id": "your-vehicle-type-id",
    "earliest_start": 1511252407,
    "break" : {
        "earliest" : 1511262407,
        "latest" : 1511265407,
        "duration" : 1800
    }
}
```

The algorithm then seeks to find the "best" position in the route to make the break. Breaks can only be made at a customer site, i.e. at any service
or shipment location <b>before</b> or <b>after</b> servicing the customer. The algorithm evaluates whether the break
is actually necessary or not. If not, it ends up in the unassigned break list. Generally, if the driver can manage to be
at its end location before `break.latest`, the break is regarded to be redundant. <b>Please note</b>, that if you specified a break,
you need to define your optimization objective to be `completion_time` (see [objectives spec above](#objectives)) otherwise you are getting an exception.

You can also specify drive time dependent breaks like this:

```json
{
    "vehicle_id": "your-vehicle-id",
    "start_address": {
        "location_id": "your-start-location-id",
        "lon": 11.028771,
        "lat": 50.977723,
        "street_hint": "your street name"
    },
    "return_to_depot": false,
    "type_id": "your-vehicle-type-id",
    "earliest_start": 1511252407,
    "break" : {
        "max_driving_time": 16200,
        "duration": 2700,
        "possible_split": [900, 1800]
    }
}
```

It says that your driver can only drive 4.5 hours in a row. It then needs a break of 45min. 
However, 45min can be split into two breaks of 15min and 30min (which is here in line with European driving break rules).
It is illustrated [here](https://discuss.graphhopper.com/t/driving-time-dependent-break-scheduling-beta-release/862).

#### Full specification of a vehicle object

Name   | Type | Required | Default | Description
:------|:-----|:---------|:--------|:-----------
vehicle_id | string | x | - | Specifies the id of the vehicle. Ids need to be unique, thus if there two vehicles with the same id, an exception is thrown.
type_id | string | - | default-type | The type_id refers to specified vehicle type (see [vehicle types](#vehicle-types)). If it is omitted a default type will be used. 
start_address | object | x | - | -
end_address | object | - | - | If this is omitted AND `return_to_depot` is `true` then the vehicle needs to return to its `start_address`.
return_to_depot | boolean | - | true | If false, the optimization determines at which customer location the vehicle should end.
earliest_start | long | - | 0 | Earliest start of vehicle in seconds. It is recommended to use the [unix timestamp](https://www.unixtimestamp.com/).
latest_end | long | - | Long.MAX_VALUE | Latest end of vehicle in seconds, i.e. the time the vehicle needs to be at its end location at latest.
skills | array | - | - | Array of skills, i.e. array of string (not case sensitive).
break | object | - | - | Specifies the driver break.
max_distance | long | - | Long.MAX_VALUE | Specifies the maximum distance a vehicle can go.
max_driving_time | long | - | Long.MAX_VALUE | Specifies the maximum drive time a vehicle/driver can go, i.e. the maximum time on the road (service and waiting times are not included here)
max_jobs | int | - | Integer.MAX_VALUE | Specifies the maximum number of jobs a vehicle can load.
max_activities | int | - | Integer.MAX_VALUE | Specifies the maximum number of activities a vehicle can conduct.

#### Full specification of a address object

Name   | Type | Required | Default | Description
:------|:-----|:---------|:--------|:-----------
location_id | string | x | - | Specifies the id of the location.
name | string | - | - | Name of location. 
lon | double | x | - | Longitude of location. 
lat | double | x | - | Latitude of location.
street_hint | string | - | - | Specifies a street hint which is used when assigning this geo locations to the road network.

#### Full specification of a break object

Name   | Type | Required | Default | Description
:------|:-----|:---------|:--------|:-----------
earliest | long | - | - | Specifies the earliest start time of the break in seconds.
latest | long | - | - | Specifies the latest start time of break in seconds.
duration | long | x | - | Specifies the duration of the break in seconds.
max_driving_time | long | - | - | Specifies the max driving time (in a row) without break in seconds.
possible_split | Array | - | - | Array specifying how a break duration (in seconds) can be split into several smaller breaks
initial_driving_time | long | - | - | Specifies the initial (current) driving time of a driver to allow dynamic adaptations in seconds.

### Vehicle Types

The default vehicle type is 

```json
{
    "type_id": "default_type",
    "profile": "car",
    "capacity": [ 0 ],
    "speed_factor": 1.0,
    "service_time_factor": 1.0,
    "network_data_provider": "openstreetmap",
    "consider_traffic": false
}
```

In the vehicle type you can specify four important features of your vehicles: profile, capacity, speed factor and service time factor. The profile indicates whether your vehicle is actually a person moving by `foot`
or a `car` and other. See [here](./supported-vehicle-profiles.md) for the details about the vehicle profiles.
Note that currently you are allowed to specify an arbitrary number of vehicle types, however, you are
only allowed to specify two different profiles for the entire set of types, e.g. `bike` and `foot`.

The capacity indicates how much freight can be loaded into the vehicle. You can specify multiple capacity dimensions as shown below. 
 With the speed factor you can make your vehicles slower or even faster. The default value here is 1.0 which is in line with the travel
 time you get from [Graphhopper Routing API](https://graphhopper.com/api/1/docs/routing/). However, in several cases it turned out that the resulting travel times were too optimistic.
 To make your plan more robust against traffic conditions, you can make your vehicle way slower (e.g. `"speed_factor" : 0.5` which corresponds to `new_travel_time = original_travel_time / 0.5`).


<!-- do you mean instead of 'to use specific roads' or possibility to pickup items? Or where is this restriction taken into account - just for the location, right? -->
For example, if your vehicle is a car that can load up to 100 units of something, specify it like this:

```json
{
    "type_id": "your-vehicle-type-id",
    "profile": "car",
    "capacity": [100]
}
```

If you want your car to have multiple capacity dimensions, e.g. weight and volume, and to be slower then specify it like this:

```json
{
    "type_id": "your-vehicle-type-id",
    "profile": "car",
    "capacity": [100,1000],
    "speed_factor": 0.8,
    "service_time_factor": 1.0
}
```

The `capacity` in a vehicle type only makes sense if there is a `size` defined in your services and shipments.

#### Further Reading

 * [Vehicle routing with cargo bikes and small trucks](https://www.graphhopper.com/blog/2019/01/30/vehicle-routing-with-cargo-bikes-and-small-trucks/)

#### Full specification of a Vehicle Type object

Name   | Type | Required | Default | Description
:------|:-----|:---------|:--------|:-----------
type_id | string | x | - | Specifies the id of the vehicle type. If a vehicle needs to be of this type, it should refer to this with its `type_id` attribute.
profile | string | - | "car" | Specifies the vehicle profile of this type, see [here](./supported-vehicle-profiles.md) for more options. The profile is used to determine the network, speed and other physical attributes to use for routing the vehicle.
capacity | array | - | [ 0 ] | Specifies an array of capacity dimension values which need to be `int` values. For example, if there are two dimensions such as volume and weight then it needs to be defined as `[ 1000, 300 ]` assuming a maximum volume of 1000 and a maximum weight of 300.
speed_factor | double | - | 1.0 | Specifies a speed factor for this vehicle type. If the vehicle that uses this type needs to be only half as fast as what is actually calculated with our routing engine then set the speed factor to 0.5.
service_time_factor | double | - | 1.0 | Specifies a service time factor for this vehicle type. If the vehicle/driver that uses this type is able to conduct the service as double as fast as it is determined in the corresponding service or shipment then set it to 0.5.
network_data_provider | string | - | "openstreetmap" | Specifies network data provider. Either use "openstreetmap" or "tomtom".
consider_traffic | double | - | false | Specifies whether traffic should be considered. if "tomtom" is used and this is false, free flow travel times from "tomtom" are calculated. If this is true, historical traffic info are used. We do not yet have traffic data for "openstreetmap", thus, setting this true has no effect at all.

### Services or Shipments

The basic difference between a Service and a Shipment is that the Service involves only one location whereas the Shipment involves two locations, i.e. a pickup and a delivery location.
A service can be specified as:

```json
{
     "id": "service-id",
     "type": "pickup",
     "name": "meaningful-name", 
     "address": {
       "location_id": "service-location-id",
       "name": "Goethe Street 1",
       "lon": 9.999,
       "lat": 53.552,
       "street_hint": "Goethe"
     },
     "duration": 3600,
     "size": [ 1 ], 
     "time_windows": [ 
        {
            "earliest": 0,
            "latest": 3600
        },
        {
            "earliest": 7200,
            "latest": 10800
        }
     ],
     "required_skills": ["drilling-machine", "water-level"],
     "allowed_vehicles": ["technician_peter","technician_stefan"],
     "disallowed_vehicles": ["technician_michael"],
     "priority": 1,
     "preparation_time": 900
}
```

A shipment can be specified as:

```json
{
    "id": "shipment-id",
    "name": "meaningful-name", 
    "pickup": {
        "address": {
            "location_id": "your-pickup-location-id",
            "name": "Johann Sebastian Bach Avenue 5",
            "lon": 12.1333333,
            "lat": 54.0833333,
            "street_hint": "Johann Sebastian Bach"
        },
        "duration": 1000,
        "time_windows": [ 
            {
                "earliest": 0,
                "latest": 1000
            } 
        ],
        "preparation_time": 900
    },
    "delivery": {
        "address": {
            "location_id": "your-delivery-location-id",
            "name": "Thomas Mann Street 1",
            "lon": 8.3858333,
            "lat": 49.0047222,
            "street_hint": "Thomas Mann Street"
        },
        "duration": 1000,
        "time_windows": [ 
            {
                "earliest": 10000,
                "latest": 20000
            }
        ]
    },
    "size": [1],
    "required_skills": ["loading-bridge"],
    "allowed_vehicles": ["trucker_stefan"],
    "disallowed_vehicles": ["trucker_peter","trucker_michael"],
    "priority": 3,
    "max_time_in_vehicle": 1800
}
``` 

Both Service and Shipment can be specified with multiple capacity dimensions as follows:

```json
"size": [1,10,150]
```

The `size`-array limits the set of possible vehicles if a `capacity`-array is defined in its vehicle type. If no `size`-array is specified the default of 0 is assumed. See [vehicle types](#vehicle-types).

#### Further Reading

 * [How can street_hint be used to improve the assignment of geo locations to road network?](https://www.graphhopper.com/blog/2018/06/22/assign-geo-locations-to-roads-and-its-impacts-on-route-optimization/)

#### Full specification of a service object

Name   | Type | Required | Default | Description
:------|:-----|:---------|:--------|:-----------
id | string | x | - | Specifies the id of the service. Ids need to be unique so there must not be two services/shipments with the same id.
type | string | - | service | Specifies whether a service is a general `service`, a `pickup` or a `delivery`. This makes a difference if items are loaded or unloaded, i.e. if one of the size dimensions > 0. If it is specified as `service` or `pickup`, items are loaded and will stay in the vehicle for the rest of the route (and thus consumes capacity for the rest of the route). If it is a `delivery`, items are implicitly loaded at the beginning of the route and will stay in the route until `delivery` (and thus releases capacity for the rest of the route).
name | string | - | - | Meaningful name for service, e.g. `deliver pizza`.
address | object | x | - | Specifies service address.
duration | long | - | 0 | Specifies the duration of the service in seconds, i.e. how long it takes at the customer site.
size | array | - | [0] | Size can have multiple dimensions and should be in line with the capacity dimension array of the vehicle type. For example, if the item that needs to be delivered has two size dimension, volume and weight, then specify it as follow `[ 20, 5 ]` assuming a volume of 20 and a weight of 5.
time_windows | array | - | - | Specifies an array of time window objects (see time_window object below). Specify the time either with the recommended [Unix time stamp](https://en.wikipedia.org/wiki/Unix_time#Encoding_time_as_a_number) (the number of seconds since 1970-01-01) or you can also count the seconds relative to Monday morning 00:00 and define the whole week in seconds. For example, Monday 9am is then represented by <code>9hour * 3600sec/hour = 32400</code>. In turn, Wednesday 1pm corresponds to <code>2day * 24hour/day * 3600sec/hour + 1day * 13hour/day * 3600sec/hour = 219600</code>. See [this tutorial](https://www.graphhopper.com/blog/2016/05/30/how-to-solve-a-traveling-salesman-problem-with-a-week-planning-horizon/) for more information.
required_skills | array | - | - | Specifies an array of required skills, i.e. array of string (not case sensitive). For example, if this service needs to be conducted by a technician having a `drilling_machine` and a `screw_driver` then specify the array as follows: `["drilling_machine","screw_driver"]`. This means that the service can only be done by a vehicle (technician) that has the skills `drilling_machine` AND `screw_driver` in its skill array. Otherwise it remains unassigned.
allowed_vehicles | array | - | - | Specifies an array of allowed vehicles, i.e. array of vehicle ids. For example, if this service can only be conducted EITHER by `technician_peter` OR `technician_stefan` specify this as follows: `["technician_peter","technician_stefan"]`.
disallowed_vehicles | array | - | - | Specifies an array of allowed vehicles, i.e. array of vehicle ids.
priority | int | - | 2 | Specifies the priority. Can be 1 = high priority to 10 = low priority. Often there are more services/shipments than the available vehicle fleet can handle. Then you could assign priorities to differentiate high priority tasks from those that can be served later or omitted at all.
preparation_time | long | - | 0 | Specifies the preparation time in seconds. It can be used to model parking lot search time since if you have 3 identical locations in a row, it only falls due once.
max_time_in_vehicle | long | - | Long.MAX_VALUE | Specifies the maximum time in seconds a delivery can stay in the vehicle. Currently, it only works with services of `"type":"delivery"`.

#### Full specification of a shipment object

Name   | Type | Required | Default | Description
:------|:-----|:---------|:--------|:-----------
id | string | x | - | Specifies the id of the shipment. Ids need to be unique so there must not be two services/shipments with the same id.
name | string | - | - | Meaningful name for service, e.g. `delivery pizza`.
pickup | object | x | - | Specifies pickup (see pickup object below).
delivery | object | x | - | Specifies delivery (see delivery object below).
size | array | - | [0] | Size can have multiple dimensions and should be in line with the capacity dimension array of the vehicle type. For example, if the item that needs to be delivered has two size dimension, volume and weight, then specify it as follow `[ 20, 5 ]` assuming a volume of 20 and a weight of 5.
required_skills | array | - | - | Specifies an array of required skills, i.e. array of string (not case sensitive). For example, if this service needs to be conducted by a technician having a `drilling_machine` and a `screw_driver` then specify the array as follows: `["drilling_machine","screw_driver"]`. This means that the service can only be done by a vehicle (technician) that has the skills `drilling_machine` AND `screw_driver` in its skill array. Otherwise it remains unassigned.
allowed_vehicles | array | - | - | Specifies an array of allowed vehicles, i.e. array of vehicle ids. For example, if this service can only be conducted EITHER by `technician_peter` OR `technician_stefan` specify this as follows: `["technician_peter","technician_stefan"]`.
disallowed_vehicles | array | - | - | Specifies an array of disallowed vehicles, i.e. array of vehicle ids.
priority | int | - | 2 | Specifies the priority. Can be 1 = high priority to 10 = low priority. Often there are more services/shipments than the available vehicle fleet can handle. Then you could assign priorities to differentiate high priority tasks from those that can be served later or omitted at all.
max_time_in_vehicle | long | - | Long.MAX_VALUE | Specifies the maximum time in seconds a shipment can stay in the vehicle.

#### Full specification of a pickup or delivery object

Name   | Type | Required | Default | Description
:------|:-----|:---------|:--------|:-----------
address | object | x | - | Specifies pickup or delivery address.
duration | long | - | 0 | Specifies the duration of the pickup or delivery in seconds, e.g. how long it takes unload items at the customer site.
time_windows | object | - | - | Specifies an array of time window objects (see time window object below). For example, if an item needs to be delivered between 7am and 10am then specify the array as follows: `[ { "earliest": 25200, "latest" : 32400 } ]` (starting the day from 0 in seconds).
preparation_time | long | - | 0 | Specifies the preparation time in seconds. It can be used to model parking lot search time since if you have 3 identical locations in a row, it only falls due once.

#### Full specification of address object

Name   | Type | Required | Default | Description
:------|:-----|:---------|:--------|:-----------
location_id | string | x | - | Specifies id of location.
lon | double | x | - | Longitude.
lat | double | x | - | Latitude.
name | string | - | - | Name of location.
street_hint | string | - | - | Specifies a street hint which is used when assigning this geo locations to the road network.

#### Full specification of a time_window object

Name   | Type | Required | Default | Description
:------|:-----|:---------|:--------|:-----------
earliest | long | - | 0 | Specifies the opening time of the time window in seconds, i.e. the earliest time the service can start.
latest | long | - | Long.MAX_VALUE | Specifies the closing time of the time window in seconds, i.e. the latest time the service can start.



### Relations

Beyond shipments there are three additional relations you can use to establish a relationship between services and shipments:

- "in_same_route"
- "in_sequence"
- "in_direct_sequence"

#### in_same_route

As the name suggest, it enforces the specified services or shipments to be in the same route. It can be specified
as follows:

```json
{
    "type": "in_same_route",
    "ids": ["service_i_id","service_j_id"]
}
``` 

This enforces service i to be in the same route as service j no matter which vehicle will be employed. If a specific vehicle (driver) is required to conduct this, just add
 a `vehicle_id` like this:
 
```json
{
     "type": "in_same_route",
     "ids": ["service_i_id","service_j_id"],
     "vehicle_id": "vehicle1"
}
```
 
 This not only enforce service i and j to be in the same route, but also makes sure that both services are in the route of `vehicle1`.
 
 *Tip*: This way initial loads and vehicle routes can be modelled. For example, if your vehicles are already on the road and new
 orders come in, then vehicles can still be rescheduled subject to the orders that have already been assigned to these vehicles.


#### in_sequence

The 'in_sequence' relation type enforces n jobs to be in sequence. It can be specified as

```json
{
    "type": "in_sequence",
    "ids": ["service_i_id","service_j_id"]
}
```

which means that service j need to be in the same route as service i AND it needs to occur somewhere after service i. As described above
if a specific vehicle needs to conduct this, just add `vehicle_id`.

#### in_direct_sequence

This enforces n services or shipments to be in direct sequence. It can be specified as

```json
 {
     "type": "in_direct_sequence",
     "ids": ["service_i_id","service_j_id","service_k_id"]
 }
```
 
yielding service j to occur directly after service i, and service k to occur directly after service j i.e. in strong order. Again, a vehicle can be assigned a priority by adding a `vehicle_id` to the relation.

#### Special IDs

If you look at the previous example and you want service i to be the first in the route, use the special ID `start` as follows:

```json
 {
     "type": "in_direct_sequence",
     "ids": ["start","service_i_id","service_j_id","service_k_id"]
 }
```

Latter enforces the direct sequence of i, j and k at the beginning of the route. If this sequence should be bound to the end of the route, use the special ID `end` like this:

```json
 {
     "type": "in_direct_sequence",
     "ids": ["service_i_id","service_j_id","service_k_id","end"]
 }
```

If you deal with services then you need to use the 'id' of your services in the field 'ids'. To also consider sequences of the pickups and deliveries
of your shipments, you need to use a special ID, i.e. use the shipment id plus the keyword `_pickup` or `_delivery`. For example, to
 ensure that the pickup and delivery of the shipment with the id 'my_shipment' are direct neighbors, you need the following specification:
 
```json
 {
     "type": "in_direct_sequence",
     "ids": ["my_shipment_pickup","my_shipment_delivery"]
 }
```

#### Full specification of a relation object

Name   | Type | Required | Default | Description
:------|:-----|:---------|:--------|:-----------
type | String | x | - | Specifies the type of relation. It must be either of type `in_same_route`, `in_sequence` or `in_direct_sequence`.
ids | array | - | - | Specifies an array of shipment and/or service ids that are in relation. If you deal with services then you need to use the 'id' of your services in 'ids'. To also consider sequences of the pickups and deliveries of your shipments, you need to use a special ID, i.e. use your shipment id plus the keyword `_pickup` or `_delivery` (see example above). If you want to place a service or shipment activity at the beginning of your route, use the special ID `start`. In turn, use `end` to place it at the end of the route.
vehicle_id | String | - | - | Id of pre-assigned vehicle, i.e. the vehicle id that is determined to conduct the services and shipments in this relation.

Learn more about it in the [live API docs](https://graphhopper.com/api/1/vrp/documentation/).

## JSON Output

If you post your problem, you either get back a job_id or an error. The job id looks like following: 

```json
{ "job_id": "7ac65787-fb99-4e02-a832-2c3010c70097" }
```

With the `job_id` you can fetch your solution via `https://graphhopper.com/api/1/vrp/solution/[job_id]?key=[YOUR_KEY]`  such as
 
`curl -X GET  "https://graphhopper.com/api/1/vrp/solution/7ac65787-fb99-4e02-a832-2c3010c70097?key=[YOUR_KEY]"`

See the error codes and JSON structure on the [overview page](https://graphhopper.com/api/1/docs/#http-error-codes). For example:
  
```json
{
  "message": "Bad Request",
  "hints": [
    {
      "message": "Unsupported json property [vehice_id]. Allowed properties: [vehicle_id, type_id, start_address, end_address, break, return_to_depot, earliest_start, latest_end, skills, max_distance]",
      "details": "class java.lang.IllegalArgumentException"
    }
  ]
}
```

### Response

Your job can be in three states, either your problem is still waiting in the queue then you get back this:

```json
{
  "job_id" : "7ac65787-fb99-4e02-a832-2c3010c70097",
  "status" : "waiting",
  "waiting_time_in_queue" : 1061,
  "processing_time" : 0,
  "solution" : "pending"
}
```

or your job is being processed but not yet finished then you get back this:

```json
{
  "job_id" : "7ac65787-fb99-4e02-a832-2c3010c70097",
  "status" : "processing",
  "waiting_time_in_queue" : 1061,
  "processing_time" : 50,
  "solution" : "pending"
}
```

or your job has an error (see above) or has a solution. Then you get back this:

```json
{
  "job_id" : "64963b20-d358-4f26-bf79-82decae50a2d",
  "status" : "finished",
  "waiting_time_in_queue" : 0,
  "processing_time" : 267,
  "solution" : {
     "costs": 58835,
     "distance": 1898748,
     "time": 58835,
     "transport_time": 58835,
     "completion_time": 58835,
     "max_operation_time": 58835,
     "waiting_time": 0,
     "no_vehicles": 1,
     "no_unassigned": 0,
     "routes": [
       {
         "vehicle_id": "my_vehicle",
         "distance": 1898748,
         "transport_time": 58835,
         "completion_time": 58835,
         "waiting_time": 0,
         "activities": [
           {
             "type": "start",
             "location_id": "berlin",
             "address": {
               "location_id": "berlin",
               "lat": 52.537,
               "lon": 13.406
             },
             "end_time": 0,
             "distance": 0,
             "driving_time": 0,
             "load_after": [
               0
             ]
           },
           {
             "type": "service",
             "id": "munich",
             "location_id": "munich",
             "address": {
               "location_id": "munich",
               "name": "Meindlstraße 11c",
               "lat": 48.145,
               "lon": 11.57
             },
             "arr_time": 17985,
             "end_time": 17985,
             "waiting_time": 0,
             "distance": 587746,
             "driving_time": 17985,
             "load_before": [
               0
             ],
             "load_after": [
               0
             ]
           },
           ...,
           {
             "type": "end",
             "location_id": "berlin",
             "address": {
               "location_id": "berlin",
               "lat": 52.537,
               "lon": 13.406
             },
             "arr_time": 58835,
             "distance": 1898748,
             "driving_time": 58835,
             "load_before": [
               0
             ]
           }
         ]
       }
     ],
     "unassigned": {
       "services": [],
       "shipments": [],
       "breaks": [],
       "details": []
     }
   }
  }
```

### Solution

As you can see, you get some general indicators of your solution like ```distance``` and ```transport_time``` which corresponds to the travelled distance and travel time,
and you get back an array of your routes with the ```vehicle_id``` and an array of ```activities``` which should be self-explanatory.
Finally, within ```unassigned``` you can find the services and shipments that could not be assigned to any route.

#### Full specification of the `response` object

Name   | Type | Description
:------|:-----|:---------
job_id | (uuid) string | Specifies the job_id used to fetch solution.
status | string | Indicates the status of your job. It can either be `waiting`, `processing` or `finished`.
waiting_time_in_queue | long | time the job waits in queue to be processed in milliseconds
processing_time | long | processing time in milliseconds.
solution | object | see below

#### `solution` object

Name   | Type | Description
:------|:-----|:---------
distance | long | Overall distance travelled in meter, i.e. the sum of each route's transport distance
transport_time | long | Overall time travelled in seconds, i.e. the sum of each route's transport time
completion_time | long | Overall completion time in seconds (completion_time = transport_time + sum(waiting_time) + sum(duration)), i.e. the sum of each route's/driver's operation time.
max_operation_time | long | Operation time of longest route in seconds
waiting_time | long | Overall waiting time in seconds
no_vehicles | integer | Number of employed vehicles
no_unassigned | integer | Number of unassigned services and shipments
routes | array | Array of routes. see below
unassigned | object | Unassigned services and shipments. see spec below

#### Route object

Name   | Type | Description
:------|:-----|:---------
vehicle_id | string | Id of vehicle operating the route
distance | long | travel distance of this route (in meter)
transport_time | long | travel time of this route (in seconds)
completion_time | long | completion time of this route (in seconds)
waiting_time | long | Overall waiting time in route (in seconds)
points | array | Array of [Geojson object for the route geometries](https://discuss.graphhopper.com/t/optimization-response-with-route-geometries/1133) of each leg in the vehicle route.  
activities | array | Array of activities. see spec below.

#### Activity object

Name   | Type | Description
:------|:-----|:---------
type | string | Specifies the type of activity. It can either be `start`, `end`, `service`, `pickup`, `delivery`, `pickupShipment` or `deliverShipment`
id | string | The reference to either the service or the shipment, i.e. corresponds to either service.id or shipment.id
location_id | string | The reference to the location id of either the address of the service or the address of shipment.pickup or shipment.delivery
address | object | address of activity including `location_id`, `name`, `lon`, `lat` (see address object specification above)
arr_time | long | Arrival time at corresponding location
end_time | long | End time at corresponding location
waiting_time | long | Waiting time at activity in seconds. Note that this field does not exist for `start` and `end`.
distance | long | Cumulated distance in meter at activity (starts with 0 at start activity)
load_before | array | Array with size/capacity dimensions. If activity is of type `start`, `load_before` does not exist.
load_after | array | Array with size/capacity dimensions. If activity is of type `end, `load_after` does not exist.


#### Unassigned object

Name   | Type | Description
:------|:-----|:---------
services | array | array of service.id that could not be assigned to any route, e.g. [ "service1", "service2" ]
shipments | array | array of shipment.id that could not be assigned to any route, e.g. [ "shipment1", "shipment2" ]
breaks | array | array of break.id that has not been assigned to its corresponding route
details | array | array with reasons why services/shipments end up being unassigned

Example:

```json
"unassigned": {
    "services": [
      "visit_munich",
      "visit_hamburg"
    ],
    "shipments": [],
    "breaks": [],
    "details": [
      {
        "id": "visit_munich",
        "code": 2,
        "reason": "cannot be visited within time window"
      },
      {
        "id": "visit_hamburg",
        "code": 3,
        "reason": "does not fit into any vehicle due to capacity"
      }
    ]
}
``` 

#### Detail object

Name   | Type | Description
:------|:-----|:---------
id | string | id of service, shipment or break
code | int | reason code
reason | string | description of reason

#### Reasons

Code   |  Reason
:------|:---------
1 | cannot serve required skill
2 | cannot be visited within time window
3 | does not fit into any vehicle due to capacity
4 | cannot be assigned due to max distance constraint of vehicles
21 | could not be assigned due to relation constraint
22 | could not be assigned due to allowed vehicle constraint
23 | could not be assigned due to max-time-in-vehicle constraint
24 | driver does not need a break
25 | could not be assigned due to disallowed vehicle constraint
26 | could not be assigned due to max drive time constraint
27 | could not be assigned due to max job constraint
28 | could not be assigned due to max activity constraint
50 | underlying location cannot be accessed over road network by at least one vehicle


## Examples

Assume you want to find the fastest round trip through Germany's 5 biggest cities starting in Berlin:

Berlin, lon: 13.406, lat: 52.537<br>
Hamburg, lon: 9.999, lat: 53.552<br>
Munich, lon: 11.570, lat: 48.145<br>
Cologne, lon: 6.957, lat: 50.936<br>
Frankfurt, lon: 8.670, lat: 50.109<br>

First, specify your vehicle at the start location Berlin. If you do not need a special vehicle type, 
you can skip the reference to a vehicle type. This automatically triggers a default type (```profile: car```, ```capacity: [0]```).


```json
{
    "vehicles" : [
       {
         "vehicle_id": "my_vehicle",
         "start_address": {
             "location_id": "berlin",
             "lon": 13.406,
             "lat": 52.537
         }
       }
    ],
    "services" : [
       {
         "id": "hamburg",
         "name": "visit_hamburg",
         "address": {
           "location_id": "hamburg",
           "lon": 9.999,
           "lat": 53.552
         }
       },
       {
         "id": "munich",
         "name": "visit_munich",
         "address": {
           "location_id": "munich",
           "lon": 11.570,
           "lat": 48.145
         }
       },
       {
         "id": "cologne",
         "name": "visit_cologne",
         "address": {
           "location_id": "cologne",
           "lon": 6.957,
           "lat": 50.936
         }
       },
       {
         "id": "frankfurt",
         "name": "visit_frankfurt",
         "address": {
           "location_id": "frankfurt",
           "lon": 8.670,
           "lat": 50.109
         }
       }
    ]
}
```

and post it for example with ```curl``` to <b>GraphHopper</b>.
Either use a json file (thus copy the above problem into problem.json)

```
curl -H "Content-Type: application/json" --data @problem.json https://graphhopper.com/api/1/vrp/optimize?key=[YOUR_KEY]
```

or copy the problem directly into your curl statement like this


```
curl -H "Content-Type: application/json" --data '{ "vehicles" : [ ...' https://graphhopper.com/api/1/vrp/optimize?key=[YOUR_KEY]
```

As described above, <b>GraphHopper</b> responds with a ```job_id```. Use the following statement to fetch your solution:

```
curl https://graphhopper.com/api/1/vrp/solution/[job_id]?key=[YOUR_KEY]
```

If the solution is available, the response looks like this:

```json
{
  "job_id" : "64963b20-d358-4f26-bf79-82decae50a2d",
  "status" : "finished",
  "waiting_time_in_queue" : 0,
  "processing_time" : 267,
  "solution" : {
    "costs" : 62180,
    "distance" : 1875953,
    "transport_time" : 62180,
    "no_unassigned" : 0,
    "routes" : [ {
      "vehicle_id" : "my_vehicle",
      "activities" : [ {
        "type" : "start",
        "location_id" : "berlin",
        "end_time" : 0,
        "distance" : 0
      }, {
        "type" : "service",
        "id" : "hamburg",
        "location_id" : "hamburg",
        "arr_time" : 9972,
        "end_time" : 9972,
        "distance" : 287064
      }, {
        "type" : "service",
        "id" : "cologne",
        "location_id" : "cologne",
        "arr_time" : 23512,
        "end_time" : 23512,
        "distance" : 709133
      }, {
        "type" : "service",
        "id" : "frankfurt",
        "location_id" : "frankfurt",
        "arr_time" : 29851,
        "end_time" : 29851,
        "distance" : 897614
      }, {
        "type" : "service",
        "id" : "munich",
        "location_id" : "munich",
        "arr_time" : 43140,
        "end_time" : 43140,
        "distance" : 1289258
      }, {
        "type" : "end",
        "location_id" : "berlin",
        "arr_time" : 62180,
        "distance" : 1875953
      } ]
    } ],
    "unassigned" : {
      "services" : [ ],
      "shipments" : [ ]
    }
  }
```

Let us assume you do not want your vehicle to come back to Berlin, but to stay in one of the other cities. Then add

```json
"return_to_depot": false
```

to your vehicle specification. This results in:

```json
{
  "job_id" : "484bf1bd-a05f-43f2-ab7f-400d8b7728ee",
  "status" : "finished",
  "waiting_time_in_queue" : 2,
  "processing_time" : 250,
  "solution" : {
    "costs" : 43140,
    "distance" : 1289258,
    "transport_time" : 43140,
    "no_unassigned" : 0,
    "routes" : [ {
      "vehicle_id" : "my_vehicle",
      "activities" : [ {
        "type" : "start",
        "location_id" : "berlin",
        "end_time" : 0,
        "distance" : 0
      }, {
        "type" : "service",
        "id" : "hamburg",
        "location_id" : "hamburg",
        "arr_time" : 9972,
        "end_time" : 9972,
        "distance" : 287064
      }, {
        "type" : "service",
        "id" : "cologne",
        "location_id" : "cologne",
        "arr_time" : 23512,
        "end_time" : 23512,
        "distance" : 709133
      }, {
        "type" : "service",
        "id" : "frankfurt",
        "location_id" : "frankfurt",
        "arr_time" : 29851,
        "end_time" : 29851,
        "distance" : 897614
      }, {
        "type" : "service",
        "id" : "munich",
        "location_id" : "munich",
        "arr_time" : 43140,
        "end_time" : 43140,
        "distance" : 1289258
      } ]
    } ],
    "unassigned" : {
      "services" : [ ],
      "shipments" : [ ]
    }
  }
}
```

Thus the vehicle does not need to return to Berlin and <b>GraphHopper</b> finds that it is best to end the trip in Munich.

Let us assume you have good reasons to end your trip in Cologne, then add this

```json
"end_address": {
    "location_id" : "cologne",
    "lon": 6.957,
    "lat": 50.936
}
```

to your vehicle specification. This gives you the following solution:

```json
{
  "job_id" : "599bdae2-5606-46b4-8aa3-d3d0ad1c468c",
  "status" : "finished",
  "waiting_time_in_queue" : 1,
  "processing_time" : 286,
  "solution" : {
    "costs" : 54828,
    "distance" : 1640434,
    "transport_time" : 54828,
    "no_unassigned" : 0,
    "routes" : [ {
      "vehicle_id" : "my_vehicle",
      "activities" : [ {
        "type" : "start",
        "location_id" : "berlin",
        "end_time" : 0,
        "distance" : 0
      }, {
        "type" : "service",
        "id" : "hamburg",
        "location_id" : "hamburg",
        "arr_time" : 9972,
        "end_time" : 9972,
        "distance" : 287064
      }, {
        "type" : "service",
        "id" : "munich",
        "location_id" : "munich",
        "arr_time" : 35300,
        "end_time" : 35300,
        "distance" : 1061169
      }, {
        "type" : "service",
        "id" : "frankfurt",
        "location_id" : "frankfurt",
        "arr_time" : 48551,
        "end_time" : 48551,
        "distance" : 1452248
      }, {
        "type" : "service",
        "id" : "cologne",
        "location_id" : "cologne",
        "arr_time" : 54828,
        "end_time" : 54828,
        "distance" : 1640434
      }, {
        "type" : "end",
        "location_id" : "cologne",
        "arr_time" : 54828,
        "distance" : 1640434
      } ]
    } ],
    "unassigned" : {
      "services" : [ ],
      "shipments" : [ ]
    }
  }
}
```

Now assume that you want to combine your round trip with an important date in Frankfurt where you need to be at latest 6 hours after you have started in Berlin,
then you need to assign an appropriate time window. This is as
easy as adding the ```time_windows``` attribute to your visit specification:

```json
{
    "id": "frankfurt",
    "name": "visit_frankfurt",
    "address": {
        "location_id": "frankfurt",
        "lon": 8.670,
        "lat": 50.109
    },
    "time_windows" : [ 
        {
            "earliest": 0,
            "latest": 21000
        }
    ]
}
```

This will force your vehicle to visit Frankfurt first and result in the following overall solution:

```json
{
  "job_id" : "274b1a80-396d-45f6-93c1-04a04c4a0a4f",
  "status" : "finished",
  "waiting_time_in_queue" : 1,
  "processing_time" : 256,
  "solution" : {
    "costs" : 73795,
    "distance" : 2218473,
    "transport_time" : 73795,
    "no_unassigned" : 0,
    "routes" : [ {
      "vehicle_id" : "my_vehicle",
      "activities" : [ {
        "type" : "start",
        "location_id" : "berlin",
        "end_time" : 0,
        "distance" : 0
      }, {
        "type" : "service",
        "id" : "frankfurt",
        "location_id" : "frankfurt",
        "arr_time" : 18009,
        "end_time" : 18009,
        "distance" : 545734
      }, {
        "type" : "service",
        "id" : "munich",
        "location_id" : "munich",
        "arr_time" : 31298,
        "end_time" : 31298,
        "distance" : 937378
      }, {
        "type" : "service",
        "id" : "cologne",
        "location_id" : "cologne",
        "arr_time" : 50201,
        "end_time" : 50201,
        "distance" : 1509761
      }, {
        "type" : "service",
        "id" : "hamburg",
        "location_id" : "hamburg",
        "arr_time" : 63764,
        "end_time" : 63764,
        "distance" : 1932043
      }, {
        "type" : "end",
        "location_id" : "berlin",
        "arr_time" : 73795,
        "distance" : 2218473
      } ]
    } ],
    "unassigned" : {
      "services" : [ ],
      "shipments" : [ ]
    }
  }
}
```

Note that when you work with time windows, infeasible problems result in unassigned services. For example, it is impossible to get from Berlin
to Frankfurt in 4 hours by car. Thus, if the latest arrival time in Frankfurt is

```json
"time_windows" : [ 
    {
        "earliest": 0,
        "latest": 14400
    }
]
```

Frankfurt then definitely ends up in the unassigned service list:

```json
 "unassigned" : {
      "services" : [ "frankfurt" ],
      "shipments" : [ ],
      "breaks" : [ ]
}
```

It is quite unrealistic that if you travel all the way from Berlin to Munich that your stay in Munich takes 0 seconds. Therefore, if your visit takes
for example 2 hours, just add a ```duration``` attribute to your Munich visit.

```json
{
     "id": "munich",
     "name": "visit_munich",
     "address": {
       "location_id": "munich",
       "lon": 11.570,
       "lat": 48.145
     },
     "duration": 7200
}
```

and you get

```json
{
  "job_id" : "f3abe8b8-8368-4951-b670-e315b48440d8",
  "status" : "finished",
  "waiting_time_in_queue" : 0,
  "processing_time" : 267,
  "solution" : {
    "costs" : 62180,
    "distance" : 1875953,
    "transport_time" : 62180,
    "no_unassigned" : 0,
    "routes" : [ {
      "vehicle_id" : "my_vehicle",
      "activities" : [ {
        "type" : "start",
        "location_id" : "berlin",
        "end_time" : 0,
        "distance" : 0
      }, {
        "type" : "service",
        "id" : "hamburg",
        "location_id" : "hamburg",
        "arr_time" : 9972,
        "end_time" : 9972,
        "distance" : 287064
      }, {
        "type" : "service",
        "id" : "cologne",
        "location_id" : "cologne",
        "arr_time" : 23512,
        "end_time" : 23512,
        "distance" : 709133
      }, {
        "type" : "service",
        "id" : "frankfurt",
        "location_id" : "frankfurt",
        "arr_time" : 29851,
        "end_time" : 29851,
        "distance" : 897614
      }, {
        "type" : "service",
        "id" : "munich",
        "location_id" : "munich",
        "arr_time" : 43140,
        "end_time" : 50340,
        "distance" : 1289258
      }, {
        "type" : "end",
        "location_id" : "berlin",
        "arr_time" : 69380,
        "distance" : 1875953
      } ]
    } ],
    "unassigned" : {
      "services" : [ ],
      "shipments" : [ ]
    }
  }
}
```

Naturally the arrival time `arr_time` in Munich does not equal the end time `end_time` anymore.

Let us now assume that you want to make this round trip a bit more exciting and challenging, 
thus you decide to switch from boring car to bike (you will definitely
be a hero if you manage the round trip by bike). Here, you
cannot use the default vehicle type anymore, but you need to define your bike yourself. This requires two changes, first define 
a vehicle type in `vehicle_types` and second make a reference to the specified type in your vehicle with `type_id`:

```json
"vehicles" : [
    {
        "vehicle_id": "my_vehicle",
        "start_address": {
            "location_id": "berlin",
            "lon": 13.406,
            "lat": 52.537
        },
        "type_id": "my_bike"
    }
],
"vehicle_types" : [
    {
        "type_id" : "my_bike",
        "profile" : "bike",
    }
]
```

The solution of your bike round trip indicates that it takes you more than 5 days, but only if you are strong enough to bike without rest.

```json
{
  "job_id" : "80deb088-096d-43da-8374-7aeb5216c1f5",
  "status" : "finished",
  "waiting_time_in_queue" : 0,
  "processing_time" : 232,
  "solution" : {
    "costs" : 449358,
    "distance" : 1975897,
    "transport_time" : 449358,
    "no_unassigned" : 0,
    "routes" : [ {
      "vehicle_id" : "my_vehicle",
      "activities" : [ {
        "type" : "start",
        "location_id" : "berlin",
        "end_time" : 0,
        "distance" : 0
      }, {
        "type" : "service",
        "id" : "hamburg",
        "location_id" : "hamburg",
        "arr_time" : 69222,
        "end_time" : 69222,
        "distance" : 317106
      }, {
        "type" : "service",
        "id" : "cologne",
        "location_id" : "cologne",
        "arr_time" : 168965,
        "end_time" : 168965,
        "distance" : 764859
      }, {
        "type" : "service",
        "id" : "frankfurt",
        "location_id" : "frankfurt",
        "arr_time" : 216860,
        "end_time" : 216860,
        "distance" : 955135
      }, {
        "type" : "service",
        "id" : "munich",
        "location_id" : "munich",
        "arr_time" : 303786,
        "end_time" : 303786,
        "distance" : 1338567
      }, {
        "type" : "end",
        "location_id" : "berlin",
        "arr_time" : 449358,
        "distance" : 1975897
      } ]
    } ],
    "unassigned" : {
      "services" : [ ],
      "shipments" : [ ]
    }
  }
}
```

You might have numerous other requirements to the sequence of your visits. For example, you might want to visit Cologne first and Munich right after 
you have visited Cologne. This might be enforced with time windows, however sometime you need additional business constraints. If so, contact us with your requirements.











