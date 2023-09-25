---
title: Optimization algorithm solving for unconventionally constrainted problems
date: 2023-06-24 12:00:00 +0300
categories: [top_category, sub_category]
tags: [tag1, tag3]     # TAG names should always be lowercase
author: Ahmed
toc: false
---
Everywhere there is a process, An optimization algorithm is needed to optimize such a process, the more complex a process is, the harder it is for a human to abide to such an algorithm and a machine is better used.
Some of such processes are problems such as 
1- Teacher scheduling.
2- Nurses rostering.
3- Job-shop scheduling.
4- Drivers scheduling.
5- Route optimization.

Route optimization for vehicles is an objective for many companies and individuals,  
It can be as simple as a vehicle visiting multiple locations and using Google directions to plot your route,  
It can be as complex as hundreds of trucks with specific capacities for weight, size and regulations visiting thousands of locations with time windows and specific requirements as well as penalties.  

I will list some of the problems I developed algorithms for to have a basic imagination of constraints that could be faced.  
  
A ride-share taxi company needed to optimize routes such that it picks up clients to drop them off at certain location at a certain timewindow,
The objective was to predict the pickup time, send the correct type of taxi depending on the client having a wheelchair or not, and having the ability to fill the place of such a wheelchair as well, while having a different "pickup time" for each client, Then optimizing a route for the taxi such that it picks up and delivers people along the route.  

A scooter rental company needed to monitor and replace scooter batteries live while also dispatching a separate team that maintains and fixes problems if problems are reported as well.

A freight trucking company needed trucks to visit locations to pickup and deliver based on timewindows, item size, capacity and many other constraints, the main problem were not actually the pickup and delivery,  
it was abiding to the breaks regulations of the [arbeitsagentur](https://www.arbeitsagentur.de/fuer-menschen-aus-dem-ausland/auslaendische-fachkraefte/saisonarbeit-in-deutschland/arbeitsrecht), there is a number of working hours limit per day, per week, extensions on such limit with a limit, multiple breaks per working hours batches.  

There are many languages as well as python libraries out there for modelling optimization problems, and there are solvers for such models.

A language such as [pymomo](https://pyomo.readthedocs.io/en/stable/) is a very powerful modelling tool where you can model your constraints but solving methods for such constraints can either be done manually or using a third party, some are [free](https://pyomo.readthedocs.io/en/stable/contributed_packages/index.html), others are commercial.

When solving for [vrp](https://en.wikipedia.org/wiki/Vehicle_routing_problem) problems, you are better off with ortools as it has well developped solvers especially suited for vrp, it is developped in C++ but has a python interface that makes life tremendously easier while retaining virtually the same speed and most of the C++ version functionalities.


Normally each location will have a timewindow that needs to be respected, how we represent time is the first thing to do, we can represent time in hour or minutes or seconds depending on the resolution that you would like but increasing the resolution means higher numbers which could potentially mean numerical overflow or speed penalties due to used precision.
A day is 24 hours, 1440 minutes, 86400 seconds.
Using seconds as a metric is very useless for most use cases as a few seconds difference is irrelevant, but a few minutes difference can be a problem.
A truck 5 minutes late is 300 seconds late as well, do you see how in terms of numerical manipulation, one is much further away than the other.
That means we best convert our time in the durations matrix to minutes, that will be faster than implementing such a logic within the duration inference callback 

First we start with the duration inference callback which depends on the duration between the start point and the end point along the arc.  

def time_callback(from_idx, to_idx):
        from_node, to_node = manager.IndexToNode(from_idx), manager.IndexToNode(to_idx)
        return data['durations_matrix'][from_node][to_node] + data['service_times'][from_node]

Checking ortools's [documentation](https://github.com/google/or-tools/blob/v9.4/ortools/constraint_solver/routing_index_manager.h#L92), IndexToNode is a function that returns (from_node) the node index corresponding to the location's index in our data mirroring (from_idx) its position in ortools's inner model of our problem  , That function referres to a C++ vector linking both values together.

Next is the distances between locations, which is another accumulated cost when a vehicle is traversing locations.

def distance_callback(from_idx, to_idx):
        from_node, to_node = manager.IndexToNode(from_idx), manager.IndexToNode(to_idx)
        return data['distances_matrix'][from_node][to_node]


We can also assume there is something that gets accumulated across locations, let that be an item getting picked up, a number of visited locations, 

def demand_callback(from_idx):
        from_node = manager.IndexToNode(from_idx)
        return data['demands'][from_node]