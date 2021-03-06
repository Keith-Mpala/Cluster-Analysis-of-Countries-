# Cluster-Analysis-of-Countries
---
title: "Cluster Analysis of Country Statistics UN Dataset."


author: "Xolani Keith Mpala"
date: "2/24/2022"
output: html_document
---
# Cluster Analysis

## Introduction.

The aim of this paper is to analyze the similarities and differences in the basic country statistics of the African continent with the goal of finding a useful grouping. When we cluster observations, we want observations in the same group to be similar and observations in different groups to be dissimilar. I will apply several clustering set of techniques for finding subgroups of observations within a country’s data set. These techniques include partitioning clustering (K-means, PAM) and hierarchical clustering.



## Review of the Data set.

Country Statistics UN Dataset used in this analysis was found on UN website. The dataset has 230 countries from all continents with about 50 columns which describe country economic statistics in the year 2017. Our data of interest is to examine the African continent, so we filter countries from the African Region (Southern, Northern, Eastern, and Western) to get 52 African countries. The columns of the Data set of interest are as following:

•	Population in thousands (2017)

•	GDP: Gross domestic product (million current US$)

•	Unemployment (% of labor force)

•	International trade: Exports (million US$)

•	International trade: Imports (million US$)

•	Individuals using the Internet (per 100 inhabitants)

•	Co2 emissions (million tons/tons per capita)

## Exploratory Data Analysis.

#### Loading the relevant data analysis libraries.

```{r}

load.libraries <- c( 'cluster', 'clustertend','corrplot', 'tidyverse','ggplot2','factoextra')
suppressPackageStartupMessages(library(dendextend))
suppressPackageStartupMessages(library(gridExtra))
suppressPackageStartupMessages(library(dplyr))
suppressPackageStartupMessages(library(knitr))
suppressPackageStartupMessages(library(data.table))
```

```{r}
#loading the data set
countries_data <- read.csv("country_profile_variables.csv")
```

``` {r}
#Select data of interest and rename some columns
data <- countries_data[,c(1,2,4,7,16,20,21,42,45)]
new_names <- c("Country","Region","Population","GDP_millions","Unemployment_rate",
               "Exports(millions)","Imports(millions)","Internet_use(/100)","cO2_emission")
names(data) <- new_names
```

We filter countries from the African Region (Southern, Northern, Eastern, and Western)
```{r}
library(data.table)
#Selecting African countries
data <- data[data$Region %like% "Africa", ]
```

The dataset now consists of 56 African countries however, there are columns with null variables such as ‘-~0.0’, ‘-99’, ‘…’, ‘~0’.  These variables are converted into NA values and columns with NA values removed. Observations with NA values were omitted because there isn’t any good justification to support the assumptions. After omitting observations with NA's, we are left with 52 African countries. Country Zimbabwe is from Southern African Region and hence it was changed to Southern Africa Region instead of Eastern Africa.
```{r}
data[data$Country=="Zimbabwe",]$Region <- "SouthernAfrica"
data$GDP_millions <- ifelse(data$GDP_millions == -99,NA,data$GDP_millions)
data$Unemployment_rate <- ifelse(data$Unemployment_rate== '...'|data$Unemployment_rate== '-99',
                                    NA,data$Unemployment_rate)
data$`Exports(millions)`<- ifelse(data$`Exports(millions)`== '~0'|data$`Exports(millions)`=='-99',
                                     NA,data$`Exports(millions)`)
data$`Imports(millions)`<- ifelse(data$`Imports(millions)`== '-99'|data$`Imports(millions)`=='~0'
                                     |data$`Imports(millions)`=='...',NA,data$`Imports(millions)`)
data$cO2_emission <- ifelse(data$cO2_emission== -99,0,data$cO2_emission)
data <- na.omit(data)
```

Table below shows number of countries in each region.
```{r}
#Shortening country name for clear visualization when clustering
data[data$Country=="Burkina Faso",]$Country <- "Burkina.F"
data[data$Country=="Sierra Leone",]$Country <- "Sierra.L"
data[data$Country=="Guinea-Bissau",]$Country <- "Guinea-B"
data[data$Country=="Sao Tome&Principe",]$Country <- "S.T&Principle"
data[data$Country=="United Republic of Tanzania",]$Country <- "Tanzania"
data[data$Country=="Democratic Republic of the Congo",]$Country <- "D.R Congo"
table(data$Region)
```

Some numeric variables are treated as characters . We therefore, need  to change such character variables to numeric using a for loop.
```{r}
#Changing char variables to numeric
countries <- data[-c(1,2)]
for(item in colnames(countries)) {
  countries[[item]] <- as.numeric(countries[[item]])
}
```

Since considering that variables in the dataset are measured in different scales, it is recommended to scale the data in order to make those variables comparable. This avoids a problem in which some features come to dominate solely because they tend to have larger values than others.
```{r}
countries_scaled <- as.data.frame(lapply(countries, scale))
View(countries_scaled)
row.names(countries_scaled) <- data[,1]
```

## Correlation Matrix.

Here we look at the relationship between the variables using a correlation matrix.
The correlation matrix share information about the distribution of each variable measured. The correlation plot shows us the correlation between all these different variables in the country dataset. There are fairly tightly linear relationships such as the correlation between GDP of the country and { Co2_emission, international trade Imports and Exports }. We can also observe the strongest positive linear correlation lies between between cO2 emission and International trade exports.

```{r}
library(corrplot)
corrplot(cor(countries_scaled, use="complete"), method="number", type="upper", diag=F,
         tl.col="black", tl.srt=30, tl.cex=0.9, number.cex=0.85, title="African Countries 2017", mar=c(0,0,1,0))
```


## Clustering tendency.

Before proceeding with any clustering method, it is worth to assess the general clustering tendency of the data. For this purpose, Hopkins’s statistic and visual assessment were used.
```{r}
suppressPackageStartupMessages(library(factoextra))
get_clust_tendency(countries_scaled, 2, graph=TRUE, gradient=list(low = 'blue', high = 'white'),seed=1234)
```

In the context of the conducted analysis, the results are rather satisfactory. Hopkins’s statistic is equal to 0.637119, which is above 0.5 and thus, according to R Documentation, we can conclude that the data set is significantly clusterable.

## The optimal number of clusters.

In the next step it is necessary to obtain the optimal number of clusters for each of partitional clustering method. Since the analyzed data set is rather small, therefore there is no need to consider CLARA which is intended for large data sets. However, for comparative purposes, both K-means and PAM will be implemented. The optimal number of clusters will be chosen primarily based on silhouette statistic.

## Silhouette statistic.

```{r}
f1 <- fviz_nbclust(countries_scaled, FUNcluster = kmeans, method = "silhouette") + 
  ggtitle("Optimal number of clusters \n K-means")
f2 <- fviz_nbclust(countries_scaled, FUNcluster = cluster::pam, method = "silhouette") + 
  ggtitle("Optimal number of clusters \n PAM")
grid.arrange(f1, f2, ncol=2)
```

## Analysis of Silhouette statistic.

The results indicate that the optimal number of clusters for the countries dataset was assessed as 2 for both K-means analysis and PAM analysis. The average silhouette width for K-means and PAM is therefore comparable in the case of 2 clusters. What is more, in both cases (K-means, PAM), the average silhouette width value for 3 clusters is only significantly lower than its value for the optimal number of clusters. 


## Total within-clusters sum of square.

To confirm the results, it is always good to look at an alternative method. Therefore, I check the stability of the above obtained results by using the WSS statistics.
```{r}
f1 <- fviz_nbclust(countries_scaled, FUNcluster = kmeans, method = "wss") +
  geom_vline(xintercept = 2, linetype = 2)+
  ggtitle("Optimal number of clusters \n K-means")
f2 <- fviz_nbclust(countries_scaled, FUNcluster = cluster::pam, method = "wss") + 
  geom_vline(xintercept = 4, linetype = 2)+
  ggtitle("Optimal number of clusters \n PAM")
grid.arrange(f1, f2, ncol=2)
```

### Analysis of Total within-clusters sum of square

Summing up, in both cases (K-means and PAM) the division into 2 clusters seems to be the most promising. However, due to the subject of interest of the analysis and the obtained results, the case for 3 and 4 clusters will also be considered and analyzed.

# K-means Clustering

### K-means with 2 clusters

Clustering will be produced based on the K-means algorithm for the case with two, three and four clusters. It was decided to use Euclidean distance to calculate dissimilarities between observations. 
```{r}
km2 <- eclust(countries_scaled, k=2 , FUNcluster="kmeans", hc_metric="euclidean", graph=F)
c2 <- fviz_cluster(km2, data=countries_scaled, elipse.type="convex", geom=c("point")) + ggtitle("K-means with 2 clusters")
s2 <- fviz_silhouette(km2)
grid.arrange(c2, s2, ncol=2)
```

 
### K-means with 3 clusters

```{r}
km3 <- eclust(countries_scaled, k=3 , FUNcluster="kmeans", hc_metric="euclidean", graph=F)
c3 <- fviz_cluster(km3, data=countries_scaled, elipse.type="convex", geom=c("point")) + ggtitle("K-means with 3 clusters")
s3 <- fviz_silhouette(km3)
grid.arrange(c3, s3, ncol=2)
```

### K-means with 4 clusters

```{r}
km4 <- eclust(countries_scaled, k=4 , FUNcluster="kmeans", hc_metric="euclidean", graph=F)
c4 <- fviz_cluster(km4, data=countries_scaled, elipse.type="convex", geom=c("point")) + ggtitle("K-means with 4 clusters")
s4 <- fviz_silhouette(km4)
grid.arrange(c4, s4, ncol=2)
```

There is a total of 5 Regions in African Continent. The table below shows the distribution of country Region among clusters when kmeans is 4.   
```{r}
table(data$Region, km4$cluster)
```

## Analysis

Silhouette statistic is higher in the case of 2 clusters however there is an imbalance in cluster membership as cluster 1 has 36 countries as compared to cluster 2 with just 4 countries. Both K-means with 3 clusters and 4 clusters have negative average silhouette value which means that there may be mis-allocation of some clusters. K-means with 4 clusters however, is slighter has a higher silhouette statistic than in the case of 3 clusters.  It can also be easily noticed that the case of 4 clusters was created by splitting one of the clusters from the case of K-means with 2 clusters.

### Visualization of country cluster with K-means = 4 cluster
 
```{r}
fviz_cluster(km4, data = countries_scaled)
```

The tables below present the basic statistics for the characteristics of each cluster. It is worth recalling that the data has been scaled, hence negative values appear.
```{r}
countries_cl <- as.data.frame(cbind(countries_scaled, km4$cluster))
colnames(countries_cl ) <- c(colnames(countries_scaled),"cluster")
```
```{r}
# Cluster 1 (red)
summary(countries_cl[countries_cl$cluster==1,1:7]) %>% kable() 
```
 
### Cluster 1 Analysis
Countries in cluster 1 generally have the highest unemployment rates as compared to other countries according to the statistics and relatively low population besides countries in cluster 3.
```{r}
# Cluster 2 (green)
summary(countries_cl[countries_cl$cluster==2,1:7]) %>% kable() 
```
 
### Cluster 2 Analysis
This cluster consists of the most developed countries in African continent with highest Gross domestic Product and high international trade Imports and Exports. These countries have developed economies and are the top 4 countries in Africa with the highest GDP in 2021 statistics. Here, we also observe high Co2 emissions as compared to countries in other clusters. 
```{r}
# Cluster 3 (blue)
summary(countries_cl[countries_cl$cluster==3,1:7]) %>% kable()
```

### Cluster 3 Analysis
Most of the countries fall in cluster 3 with similar characteristics of low- middle developed economies. Countries including Zimbabwe, Zambia, Burkina Faso, Mali, Chad are closer to the cluster centroid of cluster 3 and it can be observed that these are among the least developed economies in the country data set with characteristics of low GDP output, lower International trade Imports and Exports.
```{r}
# Cluster 4 (purple)
summary(countries_cl[countries_cl$cluster==4,1:7]) %>% kable() 
```
 
### Cluster 4 Analysis
Cluster 4 have highest internet use per 100 inhabitants and generally higher population compared to countries in cluster 1 and 3.

It is worth noting that countries like Morocco, Angola, Ethiopia, Kenya and D.R. Congo are on the boundary of cluster 3 closer to cluster 2 because they have higher GDP output and International trade import and Exports than the rest of the countries in that cluster.These countries are among the top 10 countries with highest GDP output and can be considered as mid-developed economies.

Below is the link of 52 African countries ranked according to their GDP output 2021 stats:

(https://www.statista.com/statistics/1120999/gdp-of-african-countries-by-country/).  

# PAM CLUSTERING

In this part the clusterisation is produced based on the PAM algorithm.

## PAM with 2 clusters

```{r}
#PAM with 2 clusters
pam2 <- eclust(countries_scaled, k=2 , FUNcluster="pam", hc_metric="euclidean", graph=F)
cp2 <- fviz_cluster(pam2, data=countries_scaled, elipse.type="convex", geom=c("point")) + ggtitle("PAM with 2 clusters")
sp2 <- fviz_silhouette(pam2)

grid.arrange(cp2, sp2, ncol=2)
```

### PAM with 3 clusters

```{r}
#PAM with 3 clusters
pam3 <- eclust(countries_scaled, k=3 , FUNcluster="pam", hc_metric="euclidean", graph=F)
cp3 <- fviz_cluster(pam3, data=countries_scaled, elipse.type="convex", geom=c("point")) + ggtitle("PAM with 3 clusters")
sp3 <- fviz_silhouette(pam3)
grid.arrange(cp3, sp3, ncol=2)
```

### PAM with 4 clusters

```{r}
#PAM with 4 clusters
pam4 <- eclust(countries_scaled, k=4 , FUNcluster="pam", hc_metric="euclidean", graph=F)
cp4 <- fviz_cluster(pam4, data=countries_scaled, elipse.type="convex", geom=c("point")) + ggtitle("PAM with 4 clusters")
sp4 <- fviz_silhouette(pam4)
grid.arrange(cp4, sp4, ncol=2)
```

There is a total of 5 Regions in African Continent. The table below shows the distribution of country Region among clusters when k is 4. 
```{r}
table(data$Region, pam4$cluster)
```

Summarizing PAM clustering, Silhouette statistic for PAM is almost the same as that for K-means and also negative average silhouette values are also observed in PAM clustering. 

### Visualization of country cluster with PAM = 4 cluster

```{r} mermaid
fviz_cluster(pam4, data = countries_scaled)
```

## Analysis of PAM Clustering 

Using PAM, we observe that there is much clearer cluster visualization of countries especially in the case of country Gabon which lied on the boundary of a neighboring cluster using K-means. Using PAM algorithm, we also observe that country Tunisia now falls under a different cluster {Cluster 3 - blue} than the one previously allocated using K-means and there is now a clearer separation between these 2 neighboring clusters.

As for the differences in countries based on country statistics, the results are similar to the K-means cases, with highly developed economies clustered together.


# Hierarchical clustering

As the last method for clusterization, hierarchical clustering will be used. This idea is based on setting the hierarchy of clusters depending on chosen way of calculating the similarity between clusters. In the below analysis I will use the agglomerative hierarchical clustering technique. In this technique all observations are initially in their own clusters and then iteratively similar clusters are merged with others until one cluster is formed.

In the hierarchical clustering method, it is necessary to compute the dissimilarity matrix and thus the linkage method needs to be specified first. There are multiple options for the linkage methods, but for this paper I will compare only two methods: Ward.D method and complete linkage. The first one is frequently claimed to be sensible by default, especially when we do not have any clear theoretical justifications for any other linkage criteria. The second method, however, does well in clustering when there is some kind of noise between clusters.

## Complete Linkage

```{r}
#Complete linkage
dist_countries <- dist(countries_scaled, method = "euclidean")
hc_countries <- hclust(dist_countries, method = "complete")
dend_countries <- as.dendrogram(hc_countries) 
dend_countries_colored <- color_branches(dend_countries, h = 2)
plot(dend_countries_colored,main = " H Clustering across 52 African Countries")
rect.hclust(hc_countries, k=2, border='blue')
```

## Ward's method

```{r}
#ward D
dist_countries <- dist(countries_scaled, method = "euclidean")
hc_countries <- hclust(dist_countries, method = "ward.D")
dend_countries <- as.dendrogram(hc_countries) 
dend_countries_colored <- color_branches(dend_countries, h = 4)
plot(dend_countries_colored, main = "Clustering across 52 African Countries")
rect.hclust(hc_countries, k=4, border='blue')
```

### Analysis of hierarchical clustering

The obtained results using Ward’s method and complete linkage are different. Using complete linkage method, countries are grouped better using 2 clusters, which was suggested as the optimal number of clusters in K-means, however using Ward’s linkage method, countries are grouped into four clusters and this fully confirm the previous results obtained with K-means and PAM. Moreover, the structure of the dendrograms perfectly show how countries connect based on the similarities between their country statistics (GDP, Population, Co2 emissions) and ultimately how the next clusters connect together to result in one large cluster.

## Conclusion
In this paper, I analyze the grouping of African countries depending on their similarities and differences in the basic country statistics of the year 2017. Based on fundamental clustering methods, it was shown that when clustering countries with 4 clusters using PAM clustering was most efficient as it provided 4 distinct clusters of countries based on their similarities. It was also observed that some countries were clustered based on the economic development of the country.

