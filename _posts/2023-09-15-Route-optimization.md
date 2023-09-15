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
  
A share taxi company needed to optimize routes such that it picks up clients to drop them off at certain location at a certain timewindow,
The objective was to predict the pickup time, send the correct type of taxi depending on the client having a wheelchair or not, and having the ability to fill the place of such a wheelchair as well, while having a different "pickup time" for each client, Then optimizing a route for the taxi such that it picks up and delivers people along the route.  

A scooter rental company needed to monitor and replace scooter batteries live while also dispatching a separate team that maintains and fixes problems if problems are reported as well.

A freight trucking company needed trucks to visit locations to pickup and deliver based on timewindows, item size, capacity and many other constraints, the main problem were not actually the pickup and delivery,  
it was abiding to the breaks regulations of the [arbeitsagentur](https://www.arbeitsagentur.de/fuer-menschen-aus-dem-ausland/auslaendische-fachkraefte/saisonarbeit-in-deutschland/arbeitsrecht), there is a number of working hours limit per day, per week, extensions on such limit with a limit, multiple breaks per working hours batches.  


