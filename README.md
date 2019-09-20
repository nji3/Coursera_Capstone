# Coursera_Capstone

This repository is for the Applied Data Science Capstone for the IBM professional data science certificate issued by Coursera.

# Week 1

## Neighborhoods and living Community Recommendation Analysis in West Los Angeles Area

### Introduction: Description & Disscusion of the Background

West Los Angeles area is one of the most popular place in the US. This area contains well-known tuorist attractions, the most expensive real estates, the toppest education resources and friendly neighborhoods. It's a place that people love to live in and there are a lot of resturants and stores for people to hang out. To help locals and people who want to locate in this area make a good choice of living place and community, I would like to do a recommendation analysis based on the location data of different venues around this area from the Foursquare API.

The plan has several parts:

Part 1. Spatial clustering into groups

The capstone gathered the data by the neighbourhoods first. However, I did not use the same way. I simply collect the data by location coordinates first and use the spatial clustering method to group the location data points by the density. The method used here is DBSCAN (Density-based spatial clustering of applications with noise). It can sufficiently compress the information by the location density for further analysis on the venues gathering together.

Part 2. K-Means Clustering evaluating the similarities among the groups

Since the groups are built in the part 1, we could do a K-Means clustering on the aggreagted data by the data matrix after the one-hot encoding on the category. We need to select a meaningful value for K according to the within group standard error. Another thing I would like to do is to compare the result before and after PCA because the number of dimension is much larger than the number of observations. K-means does not perform very well when the dimension is too high when we use the euclidean distance. The final result could be used to give a general recommendation of locations that people could choose based on thier habits.

Part 3. working on

### Connect foursquare API to access the geospatial raw data

The raw data is collected from the Foursquare API like what we did before for this capstone. I choose the area range: (34.073421, -118.465343), (34.036368, -118.465343), (34.073421, -118.337496), (34.036368, -118.337496). I equally select 100 (10 by 10) location coordinates in this area and send them to the API to explore the nearby venues with a radius equal to 500. The name, address, location coordinates and most importantly, the categories are all collected.

Because each center point among those 100 points is used to explore in a 500 radius, it is higly possible that there are many duplicated venues. Before cleaning duplicates, there are 2473 venues. After cleaning duplicates, there are 1623 unqiue venues in total. And there are 263 unqiue categories.

