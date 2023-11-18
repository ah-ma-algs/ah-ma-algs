---
title: Route optimization using ortools
date: 2023-10-05 12:00:00 +0300
categories: [top_category, sub_category]
tags: [tag1, tag3]     # TAG names should always be lowercase
author: Ahmed
toc: false
---

Route optimization is used for all kinds of purposes, ranging from carpooling, Freight-hauling, taxi companies, nurses rostering, live maintenance services, etc.  
Such a problem is commonly referred to as the traveling salesman problem.  
It can be mathematically related to the [hamiltonian game](https://en.wikipedia.org/wiki/Icosian_game) which arguably is the first solution to address the traveling salesman problem, The game's object was to find a Hamiltonian cycle along the edges of a dodecahedron such that every vertex is visited a single time, and the ending point is the same as the starting point.  
Later in the 60s, a different approach was taken, which we use a variation of, instead of seeking the most optimal solution, we search for a solution that satisfies a bound, let that bound be such that the length of the solution be a multiple of the optimal length, Such an approach allows us to represent the solution/path as a minimum spanning tree graph.  
The most direct solution would be to try all the permutations (ordered combinations) and see which one is cheapest (using brute-force search) which lies within the optimal factor of the number of nodes to be visited, that being an impossible solving method to the problem, many heuristics and optimization algorithms are used instead to approximate a solution that will be near enough to the most optimal solution.  

The solution almost never is as simple as visiting nodes constructing the shortest graph possible, there always are some constraints which make it even harder, some constraints are harder than others, especially that we will use a tool to solve and each tool has its own limitations.  
Visiting thousands of locations using a single vehicle (Constructing a single graph) or multiple vehicles is almost always the first question even if the real world scenario won't contain more than a 100 locations.  
Some of the main Constraints are:  
* Having a time window per node such that a vehicle can only arrive within the boundaries of such a timewindow.  
* A vehicle can only be a allowed to visit certain nodes based on some condition.  
* A vehicle can have a capacity for pickup and dropoff problems with custom loading and unloading times.  
* A vehicle can have start and end points, instead of using a depot.  
* A vehicle can have a start and end time.  
* A vehicle can have a max distance travelled.  
* A vehicle can have a max number of nodes visited.

The unconventional constraints could be:
* Breaks (The worst and hardest constraint with ortools).
* Refueling.
* Multiple days routing.
* optimize using an equation that spans over multiple parameters.
*  

Graphical Interface: can be done using google maps or any free service:  

Check the routes, with different colors and markers on each stop displaying data when clicked and a seperate bouncy marker for starting location .


* Shifts scheduling for drivers:  
Normally you need to model a problem then solve such a model.  

[IBM's Deep blue](https://en.wikipedia.org/wiki/Deep_Blue_(chess_computer)) which won Gary kasparov, used an adaptation of it (an evaluation function) using many to-be-determined parameters through analyzing thousands of master games going through many positions and endgames ,the algorithm's capability was to search through 200 million chess positions per second with summary information.

There are many languages as well as python libraries out there for modelling optimization problems, and there are solvers for such models.  

A language such as pyomo is a very powerful modelling tool where you can model your constraints, but solving methods for such constraints can either be done manually or using a third party, some are free, others are commercial.  

When solving for vrp problems, you are better off with ortools as it has well developped solvers especially suited for vrp, it is developped in C++ but has a python interface that makes life tremendously easier while retaining virtually the same speed and most of the C++ version functionalities.  

I have seen a developer working in google saying that they actually use it for google products.  

It can be used for other purposes as discussed here (optimization algorithms) post.  

The output required by a user can be as simple as  

- A terminal output

![Desktop View](/assets/img/2023-10-05-Route-optimization-using-ortools/Terminal_output_example.png){: width="640" height="363" } 
_[Terminal output example]_


- A simple interactive map.

![Desktop View](/assets/img/2023-10-05-Route-optimization-using-ortools/Leaflet_example.jpg){: width="640" height="363" } 
_[Interactive map example]_

- A dark mode interactive map.

![Desktop View](/assets/img/2023-10-05-Route-optimization-using-ortools/dark.png){: width="640" height="363" } 
_[dark-mode map example](https://github.com/xtk93x/Leaflet.TileLayer.ColorFilter)_

- A fancy dark mode interactive map.

![Desktop View](/assets/img/2023-10-05-Route-optimization-using-ortools/dark-colorized.png){: width="640" height="363" } 
_[fancy dark-mode map example](https://github.com/xtk93x/Leaflet.TileLayer.ColorFilter)_

- Even fancier 3D interactive map.

![Desktop View](/assets/img/2023-10-05-Route-optimization-using-ortools/space-to-desk.png){: width="640" height="363" } 
_[3D map example](https://www.sitepoint.com/3d-maps-with-eegeo-and-leaflet/)_

- 3D interactive map close-up.

![Desktop View](/assets/img/2023-10-05-Route-optimization-using-ortools/leaflet-eegeo.png){: width="640" height="363" } 
_[3D map example](https://www.sitepoint.com/3d-maps-with-eegeo-and-leaflet/)_

I use ortools for many reasons detailed here(refer to other post).  

Data can be present at any shape or form, it can be an excel sheet, a google sheet or a database.  

Reading the data and formulating it into manageable output is out of this post's scope and should be simple enough.  

I will discuss the commonly requested features, as well as some uncommon ones and how I feed them to ortools as well as the theory behind that.   

Normally each location will have a timewindow that needs to be respected.  

How we represent time is the first thing to discuss, we can represent time in hours, minutes or seconds depending on the resolution that you would like but increasing the resolution means higher numerical values which could potentially mean numerical overflow or speed penalties due to used precision.  
A day is 24 hours, 1440 minutes, 86400 seconds.  
Using seconds as a metric is very useless for most use cases as a few seconds difference in arrival time has no prominent effect, but a few minutes difference can be a problem.   
A truck 5 minutes late is 300 seconds late, do you see how in terms of numerical manipulation, one is much further away than the other with relation to the start value (zero).  
That means we best convert our time in the durations matrix to minutes, that will be faster than implementing such a logic within the duration inference callback.  

Getting the durations matrix can be done through many solutions, some are expensive and slow, some are fast and free, some have certain needed features/constraints when computing the matrices.  

First we start with the duration inference callback which depends on the duration between the start point and the end point along the arc.  
```python
def time_callback(from_idx: int,
                  to_idx:   int) -> int:
        from_node, to_node = manager.IndexToNode(from_idx), manager.IndexToNode(to_idx)
        return data['durations_matrix'][from_node][to_node] + data['service_times'][from_node]
```
  
Checking ortools's [documentation](https://github.com/google/or-tools/blob/v9.4/ortools/constraint_solver/routing_index_manager.h#L92) IndexToNode is a function that returns **from_node** which is the node index corresponding to the location's index in our data mirroring **from_idx** which is its position in ortools's inner model of our problem ,That function referres to a C++ vector linking both values together.  

Next is the distances between locations, which is another accumulated cost when a vehicle is traversing locations.  
```python
def distance_callback(from_idx: int,
                      to_idx:   int) -> int:
        from_node, to_node = manager.IndexToNode(from_idx), manager.IndexToNode(to_idx)
        return data['distances_matrix'][from_node][to_node]
```
We can also assume that there is a value that gets accumulated across locations, let that be an item getting picked up, a number of visited locations, a sized item or even a multivariate.   

```python
def demand_callback(from_idx:   int) -> int:
        from_node = manager.IndexToNode(from_idx)
        return data['demands'][from_node]
```

Normally such values are dependent on the currently visited node/start point which is `from_idx`, but you can choose to use the `to_idx` as well.  

Next we need to register such functions to our ortools model.   
Ortools uses such functions to determine the arc costs between the from and to locations.  
we can use `RegisterUnaryCallback` which signals ortools to call such a function with only the from location as an argument, it expects an int back which returns the cost of the arc.  
we can also use `RegisterTransitCallback` which signals ortools to call such a function with the from and to locations as arguments, it expects an int back which returns the cost of the arc.  

Since we have decided that the demand of a location is dependent only on the start nodes in an arc, we use:  
`routing.RegisterUnaryCallback(demand_callback)`

The `distance_callback` and the `time_callback` on the other hand, need the start and end nodes to calculate an arc's cost, so we use: `routing.RegisterTransitCallback(demand_callback)`

Next we add such a [dimension](https://developers.google.com/optimization/routing/dimensions) to the solver such that it uses it to track quantities accumulated using many parameters as shown below:
```python
routing.AddDimension(routing.RegisterTransitCallback(time_callback),
                     MAX_SLACK,
                     MAX_CUMUL,
                     SET_INITIAL_CUMUL_TO_ZERO,
                     Dimension_name)
```
Assume an arc's start point is `i` and the end point is `j`.  
A graph is a number of connected arcs, in our case, they have a beginning and an end.  
`MAX_CUMUL` is the maximum value that can be accumulated before reaching the last point along a full graph.  
`MAX_SLACK` is the maximum slack that can be added to cumul at location `i` if needed such that we can reach location `j` at a certain value which can be pre-assigned.  
We could use an analogy of a vehicle waiting at node `i` for a certain time (slack) before moving to arrive (cumul) at a location `j` at its timewindow's start. **cumul<sub>j</sub> = cumul<sub>i</sub> + slack<sub>i</sub> + cost<sub>ij</sub>**  
`SET_INITIAL_CUMUL_TO_ZERO` simply sets the start point's cumul of each graph to zero, for example, multiple vehicles can either all start at the same time at the day's start (zero cumul) or they can start at different times, in that case, a vehicle that starts one hour late is assumed to have accumulated 1 hour at start.  
`Dimension_name` is what we use to call the dimension to maniuplate later.  

Next, we need to initialize the Routing manager, which is what handles the conversions from the ortools nodes representation to our own indices representation back and forth.  

We initialize it with the total number of nodes, number of nodes to visit and whether they all start and end at the same location or they get to have their distinct start and end locations (which signifies the number of separate graphs to have).  
If its the same location, then you can write which location it is in the data you provide , otherwise, you have to input two arrays containing the start and end indices in your data which preferably should be at the start of your data.  
`No_of_nodes` is the number of start points plus end points for the drivers plus the number of locations that need to be visited by the drivers.

```python
from ortools.constraint_solver import pywrapcp

# Have a single location for all vehicles as the start and end point of each route.
manager = pywrapcp.RoutingIndexManager(No_of_locations,
                                       No_of_drivers,
                                       0) # we prefer having 0 as the first index as the start of all graphs.


# Have distinct locations for each vehicles's start and end locations.
manager = pywrapcp.RoutingIndexManager(No_of_locations,
                                       No_of_drivers,
                                       [0,2,4,6], # Start point indices in our data.
                                       [1,3,5,7]) # End point indices in our data.
# I prefer start and end point indices to be in that order to make it easier for me when preparing the data, but there is no governing rule.
```


Since we are creating a routing model for this example, we feed the `RoutingModel` with the`RoutingIndexManager` to initialize the model with the manager's parameters.  

```python
routing = pywrapcp.RoutingModel(manager)
```
Next we can add the dimension as done previously and then query it to add constraints, Lets add constraints such as setting a range on the cumul of a point, which translates to time windows in the routing problem or start and end times in job scheduling problems or start and end times in employee scheduling problems.  


```python
dimension = routing.GetDimensionOrDie(dimension_name)
for idx in range(driver_no):
        index = routing.Start(idx)
        dimension.CumulVar(index).SetRange(window_start, window_end)
        index = routing.End(idx)
        dimension.CumulVar(index).SetRange(window_start, window_end)
#
start_locations = driver_no * 2 # each driver gets a start and an end location, so in ortools's model, 
for idx in range(start_locations, no_of_locations):
        index = manager.NodeToIndex(idx)
        dimension.CumulVar(index).SetRange(window_start, window_end)
```
But what if we want to allow multiple windows, multiple ranges of values that can be accumulated before arriving at such a point, Just like a vehicle can arrive at a location at **12:00 - 13:00**  or **14:00-15:00**, That means **12:00 >= accumulated time => 13:00  || 14:00 >= accumulated time => 15:00**  
  
Assuming the the total accumulatable interval is  **0 => 16**  

```python
dimension.CumulVar(manager.NodeToIndex(index)).RemoveInterval(0,12)
dimension.CumulVar(manager.NodeToIndex(index)).RemoveInterval(13,14)
dimension.CumulVar(manager.NodeToIndex(index)).RemoveInterval(15,16)
```

or you can add a conditional constraint    
That tells ortools, if a node is active then check if those conditions are obeyed, if true then output 1
```python
expression = routing.solver().ConditionalExpression(routing.ActiveVar(manager.NodeToIndex(index)), # if this node is chosen among the solution.
                                                    ((dimension.CumulVar(manager.NodeToIndex(index)) < 13) and (dimension.CumulVar(manager.NodeToIndex(index)) > 12)) or 
                                                    ((dimension.CumulVar(manager.NodeToIndex(index)) < 15) and (dimension.CumulVar(manager.NodeToIndex(index)) > 14)), # if this constraint is obeyed as well.
                                                    1) # output 1 if the above constraint is obeyed
routing.solver().AddConstraint(expression >= 1) # always ensure the above expression is obeyed
```

or you can add the constraints directly.
```python
routing.solver().Add(((dimension.CumulVar(manager.NodeToIndex(index)) < 13) and (dimension.CumulVar(manager.NodeToIndex(index)) > 12)) or \ 
                      ((dimension.CumulVar(manager.NodeToIndex(index)) < 15) and (dimension.CumulVar(manager.NodeToIndex(index)) > 14))  
                    ) # always ensure the above expression is obeyed
```

I have ordered them in the least buggy order that I have found through trials.  
you can also set a [soft bound](https://github.com/google/or-tools/blob/858fa626959f7e386153af82756384b79f983b5a/ortools/constraint_solver/routing.h#L2236-L2249) to the cumul variable, If the value of the cumul variable breaks the bound, a cost proportional to the difference between this value and the bound is added to the cost function of the model `cost = coefficient * (lower_bound - cumulVar)`
`dimension.SetCumulVarSoftLowerBound(manager.NodeToIndex(index), bound, coefficient)`
`dimension.SetCumulVarSoftUpperBound(manager.NodeToIndex(index), bound, coefficient)`
Mixing lower and upper bounds on an index are not recommended and bug inducing, also using a `SpanCostCoefficient` with them should introduce bugs.


Sometimes you need to add a capacity constraint, you can actually do it by assigining a limit on the cumul which is what was done earlier but lately they have provided a function specific to that.

```python
routing.AddDimensionWithVehicleCapacity(
    demand_callback,
    0,  # null capacity slack
    vehicle_capacities_array,  # vehicle maximum capacities
    True,  # start cumul to zero
    "Capacity",
)
```
Such a constraint you can use to easen up tracking a value added or subtracted across each node while obeying an upper limit, such as a vehicle loading and unloading at nodes while not exceeding the vehicle's capacity.  

One very essential part when using ortools for developing vrp, crvptw or any route optimization algorithm is getting the distances matrix and the durations matrix.

[OSRM](https://github.com/Project-OSRM/osrm-backend) has been the best for me when it comes to speed and ease of use.  
[Google maps](https://github.com/Project-OSRM/osrm-backend) has been the worst by a far margin.  
[Valhalla](https://github.com/Project-OSRM/osrm-backend) I only use for custom needed solutions for problems such as multiple types of vehicles or multiple road constraints such as speed limits.
