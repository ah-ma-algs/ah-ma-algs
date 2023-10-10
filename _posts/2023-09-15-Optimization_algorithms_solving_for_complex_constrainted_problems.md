---
title: Optimization algorithms solving for complex constrainted problems
date: 2023-09-15 12:00:00 +0300
categories: [top_category, sub_category]
tags: [tag1, tag3]     # TAG names should always be lowercase
author: Ahmed
toc: false
---
Everywhere, there is a process, An optimization algorithm is needed to optimize such a process, the more complex a process is, the more parameters it needs interlinked and optimized, the harder it is for a human to process to such an algorithm and a machine can better perform.  
   
Some of such processes are problems such as  
- Teacher scheduling.
- Nurses rostering.
- Job-shop scheduling.
- Drivers scheduling.
- Route optimization.
- Bin-packing.
- Knapsacking.  

Normally you need to model a problem then solve such a mode.  
There are many languages as well as python libraries out there for modelling optimization problems, and there are solvers for such models.  

A language such as [pymomo](https://pyomo.readthedocs.io/en/stable/) is a very powerful modelling tool where you can model your constraints, but solving methods for such constraints can either be done manually or using a third party, some are [free](https://pyomo.readthedocs.io/en/stable/contributed_packages/index.html), others are commercial.  

When solving for [vrp](https://en.wikipedia.org/wiki/Vehicle_routing_problem) problems, you are better off with ortools as it has well developped solvers especially suited for vrp, it is developped in C++ but has a python interface that makes life tremendously easier while retaining virtually the same speed and most of the C++ version functionalities.  

I have seen a developer working in google saying that they actually use it for google products.  

Working with ortools is harder and easier than it might appear, The main problem with ortools for me is that sometimes an error as simple as a float number returned by a function that feeds its output to an ortools dimension, can let the model output a very normal looking output, but when you examine the output, you will discover that all of the constraints were ignored.  
Upon further inspection, you will understand that somehow it failed internally but still ran and gave output.  

Sometimes it gives an error elsewhere because while running, a wrong value was returned and such a value caused the next run to fail, not the current run which is confusing and sometimes very hard to catch.  

Sometimes its just buggy because of the mixing of constraints, and breaking them up is the right thing to do, like breaking a problem into multiple problems and interchaining them.  

Route optimization for vehicles is an objective for many companies and individuals,  
It can be as simple as a vehicle visiting multiple locations and using Google directions to plot your route,  
It can be as complex as hundreds of trucks with specific capacities for weight, size and regulations visiting thousands of locations with time windows and specific requirements as well as penalties.  

I will list some of the problems that I developed algorithms for to have a basic imagination of constraints that could be faced.  
  
- A ride-share taxi company needed to optimize routes such that it picks up clients to drop them off at certain location at a certain timewindow.  
The objective was to predict the pickup time, send the correct type of taxi depending on the client having a wheelchair or not, and having the ability to fill the place of such a wheelchair as well, while having a different "pickup time" for each client, Then optimizing a route for the taxi such that it picks up and delivers people along the route.  

- A scooter rental company needed to monitor and replace scooter batteries live while also dispatching a separate team that maintains and fixes problems if problems are reported as well.

- A freight trucking company needs trucks to visit locations to pickup and deliver items based on timewindows, item size, capacity and many other constraints, the main problem was not actually the pickup and delivery,  
it was abiding to the breaks regulations of the [arbeitsagentur](https://www.arbeitsagentur.de/fuer-menschen-aus-dem-ausland/auslaendische-fachkraefte/saisonarbeit-in-deutschland/arbeitsrecht), there is a number of working hours limit per day, per week, extensions on such limit with a limit, multiple breaks per working hours batches, all of such were constraints that needed to be applied to the model evading ortools's bugs.    

- A nurses scheduling algorithm such that the nurses get to visit homes according to the needs of patients and their respective capabilities, respecting time windows, nursing time, nursing time limit and nursing visits limit.  

- A marble cutting saw factory that needed to optimize cutting 2D shapes(not necessarily rectangles) out of marble such that the saw's width is taken into account and some shapes need to be cut out of the same stone to ensure the same shade of color and pattern.  

- A 3D printer that needed to build multiple shapes simultaneously to maximize efficiency and minimize working time.  


