# Coursera_Capstone

This repository is for the Applied Data Science Capstone for the IBM professional data science certificate issued by Coursera.

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

### Show the maps

Plot each venue on to the map by the location coordinates. Each venue is displayed as a blue point. The daker space means more points are closely gathering together on the map.

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/allVenuesMap.png" width="800px"</img> 
</div>

## Data Cleaning and Preparation

After the raw data cleaning, duplicates are removed. There are 1623 venues remained. The next stage is to do further transformation of the data.

So each data point is a Venue location with a category. To have a better understanding of our dataset and extent the information, we would want to transform the original dataset into a high-dimensional data matrix by one-hot encoding.

There are 261 unqiue categories in total and many of them are very redundant. Here a cleaning and combining process is needed to make the categories more clear and reduce the dimension. After this process, there will be only 197 unqiue categories.

After the one-hot encoding, the category column is extended to 197 columns. The data matrix is very spatial that each raw having a lot of 0s and a single 1. We are going to do futher aggregation based on this dataset after the spatial clustering is done.

The top 25 and 100 to 125 total number of unique categories are shown here.

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/originalVenuesnumbers.png" width="800px"</img> 
</div>

The categories are too detailed, which cause the problem that many categories only have one Venue (observation). This would cause a lot of meaningless dimension for the clustering and also cause the information loss. We need to check the categories again and combine them a bit more.

The categories combinations:

categories contain 'Pub', 'Bar', 'Club', 'Speakeasy' ---> 'Bar, Pub & Club'

categories contain 'Resturant' with number smaller than 10 ---> 'Other Restaurant'

categories contain 'Garden', 'Park', 'Landscaping' ---> 'Landscape'

categories contain 'School', 'College' ---> 'Education'

categories contain 'Hotel', 'Motel' ---> 'Hotel'

categories contain 'Gym', 'Dance', 'Recreation' ---> 'Gym and Physics Training'

After the cleaning, there are 194 unqiue combined categories left. The top 25 unique combined categories after the cleaning:

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/cleaneddataVenues.png" width="800px"</img> 
</div>

Now we can use the cleaned categories to do the **one-hot encoding**.

## Part 1: spatial clustering into groups

### Compute DBSCAN

The scikit-learn DBSCAN haversine distance metric requires data in the form of [latitude, longitude] and both inputs and outputs are in units of radians.

1. eps is the physical distance from each point that forms its neighborhood

2. min_samples is the min cluster size, otherwise it's noise - set to 1 so we get no noise.

Extract the lat, lon columns into a numpy matrix of coordinates, then convert to radians when you call fit, for use by scikit-learn's haversine metric.

### First Clustering for the most densed data poinst

According to the map we saw before, we could saw that there are areas venues gathering densely and loosely. We might need iteratively do the clustering and group the data points step by step. The data points labeled as noise in the first round will be reclustered in the second round.

Because we only use location coordinates for the DBSCAN, the data points will be clustered based on the geometric distance and density. The number of kilometers in one radian is 6371.0088 kms and we need to convert the epsilon used for DBSCAN from kilometers to randians to be used by haversine. I did two rounds of clustering. For the first round, I choose epsilon to be 0.2 kms divided by 6371.0088 kms with 10 minimal samples in each cluster. The second round has the epsilon equal to 0.5 kms divided by 6371.0088 kms with 5 minimal samples in each cluster.

1215 data points are clustered in the first round. 419 data points labeled as noise in the first round are used as inputs for the second round and 364 points are clustered. The rest 55 points are too loose in the map, which would be dropped out for the later K-means clustering.

### First Round

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/firstDCSCAN.png" width="800px"</img> 
</div>

-------------------------------------
The number of noises: 414
-------------------------------------
Total number of Venues clustered: 1220
-------------------------------------
The number of Venues in Cluster 0 is 13, 
The number of Venues in Cluster 1 is 99, 
The number of Venues in Cluster 2 is 36, 
The number of Venues in Cluster 3 is 101, 
The number of Venues in Cluster 4 is 61, 
The number of Venues in Cluster 5 is 14, 
The number of Venues in Cluster 6 is 20, 
The number of Venues in Cluster 7 is 70, 
The number of Venues in Cluster 8 is 46, 
The number of Venues in Cluster 9 is 23, 
The number of Venues in Cluster 10 is 20, 
The number of Venues in Cluster 11 is 53, 
The number of Venues in Cluster 12 is 17, 
The number of Venues in Cluster 13 is 49, 
The number of Venues in Cluster 14 is 13, 
The number of Venues in Cluster 15 is 21, 
The number of Venues in Cluster 16 is 62, 
The number of Venues in Cluster 17 is 65, 
The number of Venues in Cluster 18 is 32, 
The number of Venues in Cluster 19 is 55, 
The number of Venues in Cluster 20 is 18, 
The number of Venues in Cluster 21 is 39, 
The number of Venues in Cluster 22 is 58, 
The number of Venues in Cluster 23 is 50, 
The number of Venues in Cluster 24 is 29, 
The number of Venues in Cluster 25 is 16, 
The number of Venues in Cluster 26 is 59, 
The number of Venues in Cluster 27 is 19, 
The number of Venues in Cluster 28 is 31, 
The number of Venues in Cluster 29 is 10, 
The number of Venues in Cluster 30 is 11, 
The number of Venues in Cluster 31 is 10.

### Second Round

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/secondDCSCAN.png" width="800px"</img> 
</div>

-------------------------------------
The number of noises: 57
-------------------------------------
Total number of Venues clustered: 357
-------------------------------------
The number of Venues in Cluster 0 is 69, 
The number of Venues in Cluster 1 is 16, 
The number of Venues in Cluster 2 is 14, 
The number of Venues in Cluster 3 is 21, 
The number of Venues in Cluster 4 is 16, 
The number of Venues in Cluster 5 is 5, 
The number of Venues in Cluster 6 is 7, 
The number of Venues in Cluster 7 is 13, 
The number of Venues in Cluster 8 is 39, 
The number of Venues in Cluster 9 is 10, 
The number of Venues in Cluster 10 is 5, 
The number of Venues in Cluster 11 is 6, 
The number of Venues in Cluster 12 is 10, 
The number of Venues in Cluster 13 is 7, 
The number of Venues in Cluster 14 is 39, 
The number of Venues in Cluster 15 is 5, 
The number of Venues in Cluster 16 is 9, 
The number of Venues in Cluster 17 is 30, 
The number of Venues in Cluster 18 is 8, 
The number of Venues in Cluster 19 is 16, 
The number of Venues in Cluster 20 is 7, 
The number of Venues in Cluster 21 is 5.

### Evaluation of DBSCAN

The silhouette coefficient evaluates how close a point is to the other points in its cluster in comparison with how close it is to the points in the next nearest cluster. A high silhouette coefficient indicates the points are well-clustered and a low value indicates an outlier.

First Clustering Silhouette coefficient: 0.248

Second Clustering Silhouette coefficient: 0.285

We could see that both coefficients are not very high. One reason is that in each round, there are relatively large number of noises. However, it won't really affect the further analysis because we do the iterative process to group the data points.

Further Data Preparation
Now based on the clusters from the spatial clustering step, we could relabel each data point with an id of a group they belong to. We could visualize the number of venues clustered into each group. The -1 label means the group of noises. Ignoring the noise group, there are 54 groups intotal. We would do further aggregation of the data into each group and calculate the mean value of venues for each group based on the one-hot data matrix.

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/numberofVenuesDCSCAN.png" width="800px"</img> 
</div>

### Store top venues into the dataframe

Because now we have the mean value of venues in each group, we could sort the aggregated data and show the top 10 most common venues in each group.

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/top10MostCommon.png" width="800px"</img> 
</div>

## Part 2: K-Means Clustering evaluating the similarities among the groups

### PCA Dimension Reduction

We would like to do a PCA first to see how the data grouped naturally. After combining redundant categories, the dimension is still 197, which is much larger than the number of obseravations we have (54). To reduce the sparsity of the dataset and make the future model more stable, we need to do the dimension reduction first. PCA would be a good choice.

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/PCAvariance.png" width="400px"</img> 
</div>

Explained Variance Ratio in Top 50 Components: 0.9978814305591085.

The top 50 principle components explained more than 99% of the variance in the original dataset. We could simply used the transformed data for model fitting. The transformed dataset now has the shape 54x50.

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/PCA.png" width="800px"</img> 
</div>

However, according to the plots of the first three principle components, the dataset are not clearly seperated. We would use the tranformed data for T-SNE to see if we can have better seperations.

### Use T-SNE to visualize the data in 2-dimensional space

t-Distributed Stochastic Neighbor Embedding (t-SNE) is another technique for dimensionality reduction and is particularly well suited for the visualization of high-dimensional datasets. Contrary to PCA it is not a mathematical technique but a probablistic one.

Essentially what this means is that it looks at the original data that is entered into the algorithm and looks at how to best represent this data using less dimensions by matching both distributions. The way it does this is computationally quite heavy and therefore there are some (serious) limitations to the use of this technique. For example one of the recommendations is that, in case of very high dimensional data, you may need to apply another dimensionality reduction technique before using t-SNE, such as PCA.

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/Tsne.png" width="800px"</img> 
</div>

The T-SNE plots give us a chance to distinguish some outliers but they cannot help cluster the data. For better clustering purpose, we are going to apply the K-Means clustering on the transformed data.

### Fit the K-means model after PCA.

K-means clustering could perform well in clustering problems. However, we need to carefully select the value of k. One way to choose the value of k is to compare the within sum of squares for different k values. Considering the total number of observations is only 54, k equal to 4 is finally chosen for this project.

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/selectK.png" width="400px"</img> 
</div>

### Plot the map after K-Means clustering

**Red: Cluster 0**, **Purpule: Cluster 1**, **Blue: Cluster 2**, **Yellow: Cluster 3**

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/kmeansMap.png" width="800px"</img> 
</div>

Now the 54 grouped areas are labeled into 4 clusters. We could do some further analysis to help people determine which area they would want to stay. 

### Analysis

The top 10 Venues (with most numbers) of the 4 clusters are plotted below. According to the map, it is very clear that the cluster 1 having the most venues. The cluster 3 only has one data point, which would be least representative. 

In the cluster 3, this single data point has categories with very low appearance, such as 'Hobby Shop', 'Theater' and 'Field'. This data point can be considered as an outlier that not belong to areas with common venues.

The cluster 1 combine the areas with a lot of non-popular (other restaurants) and popular (Japanese, pizza, Cafe, Bar & Club) categorized restaurants. Places like century city westfield with many restaurants are clustered into the same group. People who love to try different kinds of food can pay attention to these areas.

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/Top10VenunesinClusters.png" width="800px"</img> 
</div>

Let's normalize the venue numbers to be ratios and compare the cluster 0, 1 and 2 with the total venues ratio. The cluster 0 grouped a lot of places for physical training (gyms and dance studios). Other functional places such as Landscapes (parks, gardens) and pharmacy are grouped. People who are more interested in going to gym or having some outdoor places for walking, running or jogging can pay attention to cluster 0 areas.

One big difference between cluster 1 and cluster 2 is that there are more bars and hotels in the cluster 2 area. This can be a good choice for visitors and tourists. As we expected, Beverly Hills and the areas between Beverly Hills and West Hollywood are grouped in cluster 2. Vistors could choose a plcae around here to stay and have fun. 

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/Top10VenueRatios.png" width="800px"</img> 
</div>

<div align="center">
        <img src="https://github.com/nji3/Coursera_Capstone/blob/master/readme_images/VenueRatiosComparison.png" width="800px"</img> 
</div>

## Future Study

Unsupervised learning or clustering is usually very unstable. It would be very sensitive to the data we have. This project can only show a way to do information aggregation and interpretation so that we could give some advice to people who want to relocate in te west Los Angeles area. To have a systametical recommendation method, we could qunatify functions of each area and the distance between venue-gathering areas and people's relocation choices. And we can calculate a rate or score to help people have a more straight-forward method to make a choice. A prototype would be discussed and designed in another jupyternote book. There could be different designs or solutions and I will just show an idea about it. I think there can be a good extention based on this idea. 

However, restricted to the data we have, it is still hard to transfer this recommendation problem from unsupervised learning to supervised learning. If we can have the real estate price in each area, we can give a better analysis for those people who want to buy a house in west Los Angeles.