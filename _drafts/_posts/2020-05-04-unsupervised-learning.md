---
layout: post
title: "Unsupervised Learning"
description: "In unsupervised learning, there is no response variable. Instead, we're looking to find subgroups among variables or observations, discover interesting things about the measurements, or visualize the data informatively. Two common methods are principal components analysis (for data visualization/pre-processing) and clustering (for discovering unknown subgroups)."
thumb_image: 
tags: [academic]
---

<div class="toc" markdown="1">
### Contents 

- [I. Principle Components Analysis](#i-principle-components-analysis)
- [II. K-Means Clustering](#ii-k-means-clustering)
- [III. Hierarchical Clustering](#iii-hierarchical-clustering)
- [IV. Practical Issues with Clustering](#practical-issues-with-clustering)
- [V. Applications in R](#v-applications-in-r)

</div>


### I. Principle Components Analysis

Principle Components Analysis (PCA) is not only useful as a [dimension reduction technique](linear-model-selection-regularization#iv-dimension-reduction-techniques) before applying other regression or classification methods for prediction, but it can also be used to explore data, but it also serves as a tool for data visualization and unsupervised learning.

In this section, we'll discuss PCA in greater detail, focusing on its use with unsupervised data.

The first principle component of a set of features $$X_1, ..., X_p$$ is the normalized $$\bigl( \sum_{j = 1}^p \phi_{j1}^2 = 1 \bigr)$$ linear combination of $$Z_1 = \phi_{11}X_1 + \phi_{21}X_2 + ... + \phi_{p1}X_p$$ with the largest variance. 

We then look for the linear combination of the sample feature values (observed data), $$z_{i1} = \phi_{11} x_{i1}  + \phi_{21} x_{i2} + ... + \phi_{p1} x_{ip}$$, that has the largest sample variance. 

In other words, the first principle component is calculating by maximizing:

$$ \max_{\phi_{11}, ..., \phi_{p1}} \Biggl\{ \frac{1}{n} \sum_{i = 1}^n \Bigl( \sum_{j = 1}^p \phi_{j1} x_{ij} \Bigr) ^ 2 \Biggr\} \text{ subject to } \sum_{j = 1}^p \phi_{j1}^2 = 1$$

- Note that the objective in the equation above can be written as $$\frac{1}{n} \sum_{i = 1}^n z_{i1}^2$$
- $$z_{11}, ..., z_{n1}$$ are called the scores of the 1st principle component and average to zero
- $$\phi_1, ..., \phi_p$$ are called loadings, and make up the loading vector $$\phi_1 = (\phi_{11}, \phi_{12}, \phi_{p1})^T$$


The second principle component $$Z_2$$ is calculated in a similar way, with the constraint that it must be uncorrelated with $$Z_1$$. It follows that the direction $$\phi_2$$ is orthogonal to $$\phi_1$$. The second principle component scores $$z_{12}, z_{22}, ..., z_{n2}$$ take the form $$z_{i2} = \phi_{12} x_{i1} + \phi_{22} x_{i2} + ... + \phi_{p2} x_{ip}$$.


**Interpretations:**
1. Principle component loading vectors are the directions within the feature space which the data vary the most.
2. Principle components provide low-dimension linear surfaces that are closest to the observations.
    - The first $$M$$ principle component loading vectors and and score vectors provide the best $$M$$-dimensional approximation to the $$i$$th observation $$x_{ij}$$, where $$x_{ij} \approx \sum_{m = 1}^M z_{im} \phi_{jm}$$
    - Note that when $$M = \min (n - 1, p)$$, then the the equation above is exact

**Scaling Variables**
- Variables are typically scaled to mean 0 and standard deviation 1, since both these measures affect the final model.
- Make exceptions as necessary. For example, if all variables have the same units, you may not want to scale standard deviation.

**Uniqueness**
- Each principle components vector is unique up to a sign flip. The signs may differ if running PCA in different software packages because each principal component loading vector specifies a direction in $$p$$-dimensional space, and flipping the sign has no effect on the direction.

**Proportion of Variance Explained (PVE)**
- Total variance is given by $$\sum_{j = 1}^p Var(x_j) = \sum_{j = 1}^p \frac{1}{n} \sum_{i = 1}^n x_{ij} ^ 2$$
- The variance explained by the $$m$$th components is given by $$\frac{1}{n} \sum_{i = 1}^n \Bigl( \sum_{j = 1}^p \phi_{jm} x_{ij} \Bigr) ^ 2 $$
- The PVE of the $$m$$th principle component is the variance of explained by the $$m$$th component divided by the total variance. There are a total of $$\min (n - 1, p)$$ principle components, and their PVEs sum to 1.

$$ \text{PVE} = \frac{ \sum_{i = 1}^n \Bigl( \sum_{j = 1}^p \phi_{jm} x_{ij} \Bigr) ^ 2} {\sum_{j = 1}^p \sum_{i = 1}^n x_{ij} ^ 2}$$

**Deciding how many principle components to use**
- Generally, look for an "elbow" in the "scree plot" of principle components and the proportion of variance that they each explain. An example image is shown below.[^1]
- For unsupervised learning, this is subjective; for supervised learning, use cross validation. 

<center>{% include image.html path="documentation/05-04-scree-plot.png"
                      path-detail="documentation/05-04-scree-plot.png"
                      alt="Scree Plot Example" %}</center>

[^1]: Source: James et. al, *Intro to Statistical Learning 7th edition*, pg. 383


---
---
---
### II. K-Means Clustering

Clustering looks to find homogenous subgroups among observations, while PCA looks to find a low-dimensional representation of observations that explain its variance. For example, you may want to perform clustering if you're examining gene expression measurements to find heterogeneity to identify breast cancer subtypes, or if you're performing market segmentation by identifying subgroups of a population.

Note that we can cluster observations based on features, or features based on observations by transposing the data matrix. For simplicity, this post will focus on clustering observations based on features.

K-Means Clustering partitions a data set into $$K$$ distinct, non-overlapping clusters. Let $$C_1, ..., C_k$$ denote sets containing the indices of the observations in each cluster.

1. Each observation belongs to one of the $$K$$ clusters: $$C_1 \cup C_2 \cup ... \cup C_k = \{1, ..., n\}$$.
2. No observation belongs to more than one cluster: $$ C_k \cap C_{k'} = 0 \text{ for all } k \neq k' $$.
3. We want to minimize the within-cluster variation, $$W(C_k)$$ so observations within each cluster are similar.

$$ \min_{C_1 ... C_k} \Biggl\{ \sum_{k = 1}^K W(C_K) \Biggr\} $$

The most common way to define within-cluster variation is with **Euclidian distance**:

$$ W(C_K) = \frac{1}{\vert C_K \vert} \sum_{i, i' \in C_K} \sum_{j = 1}^p (x_{ij} - x_{i'j}) ^ 2 $$

- $$\vert C_K \vert$$ represents the number of observations in the $$k$$th cluster.

<span class="boxheader">Algorithm for a local optimum:</span>
1. Randomly assign a number from 1 to $$K$$ to each observation as the initial cluster assignments.
2. Iterate until the cluster assignments stop changing:
    - For each cluster, compute the centroid [a vector of $$p$$ feature means].
    - Assign each observation to the cluster whose centroid is closest, by Euclidian distance.

The results of this algorithm depend on the inintial cluster assignment. We should run the algorithm multiple times with different initial assignments, then select the one for which $$W(C_K)$$ is smallest.

The algorithm is guaranteed to decrease the within-cluster variation. This is because within-cluster variation can be rewritten as $$ 2 \sum_{i \in C_k} \sum_{j = 1}^p (x_ij - \bar x_{kj}) ^ 2$$, where $$ \bar x_{kj} = \frac{1}{C_k} \sum_{i \in C_k} x_{ij} $$ is the mean for feature $$j$$ in cluster $$C_k$$. Reallocating the observations from their previous cluster assignment can only improve this equation. 

---
---
---
### III. Hierarchical Clustering

Hierarchical clustering creates a tree-based representation of observations called a **dendrogram**, and it does not require a choice of $$K$$ at the beginnning. The most common type of hierarchical clustering is bottom-up, or agglomerative clustering that is built from the bottom (leaves) up to the top (trunk).

**Interpreting Dendrograms**

A dendrogram is created as the result of hierarchical clustering. The earlier (lower on tree) fusions occur, the more similar the groups of observations are, and the height of the fusion indicates how different the two observations are. An example is shown below.[^2]

<center>{% include image.html path="documentation/05-04-dendrogram.png"
                      path-detail="documentation/05-04-dendrogram.png"
                      alt="Dendrogram Example" %}</center>

[^2]: Source: James et. al, *Intro to Statistical Learning 7th edition*, pg. 392

Notes:
- There are $$2^{n - 1}$$ possible re-orderings of the dendrogram
- Only the vertical axis matters. We cannot draw conclusions about two observations based on horizontal proximity.
- Clusters are created by making a horizontal cut across the dendrogram
- In practice, cuts are often created subjectively just by looking at the dendrogram 
- Drawback: if the data is not hierarchical (i.e. the true clusters are not nested), then the model will not be good

<span class="boxheader">Algorithm:</span>
1. Begin with $$n$$ observations and a measure (i.e. Euclidian distance) of all $$n \choose 2 = \frac{n(n-1)}{2}$$ pairwise dissimilarities. Treat each observation as its own cluster.
2. For $$i = n, n - 1, ..., 2$$:
    - Examine all pairwise inter-cluster dissimilarities among the other $$i$$ clusters
    - Identify the pair of clusters that are least dissimilar and fuse them together
    - The dissimilarity between the two clusters indicates the height of the fusion on the dendrogram

We measure dissimilarity between groups with different **types of linkage**:
- Complete
    - Maximal intercluster dissimilarity
    - Used often, Preferred for creating a balanced dendrogram
    - Compute all pairwise dissimilarities between observations in clusters A and B, and record the largest of those dissimilarities
- Single 
    - Minimal intercluster dissimilarity
    - Used sparingly, since it can cause unbalanced dendrograms with trailing clusters where single observations are fused one at a time
    - Compute all pairwise dissimilarities in A and B, and record the smallest dissimilarity
- Average 
    - Mean intercluster dissimilarity
    - Used often, preferred for creating a balanced dendrogram
    - Compute all pairwise dissimilarities in A and B, and record the average of the dissimilarities
- Centroid
    - Least used because it can result in undesirable inversions, where two clusters are fused at a height below eather of the individual clusters in the dendrogram, leading to difficulties in visualizatino and interpretation.
    - Calculated as the dissimilarity between the centroid for cluster A (a mean vector of length $$p$$) and the centroid for cluster B.

**Choice of dissimilarity measure**

- Euclidian distance is most common
- Correlation-based distance groups objects as close if their features are highly correlated. It focuses on the shape of observation profiles rather than the magnitude.
- Also consider, based on the application, whether variables be scaled to have a standard deviation of 1. This affects dissimilarity.

---
---
---
### IV. Practical Issues with Clustering

Decisions to make:
- Should observations and features be standardied?
- For k-means clustering, how many clusters should we look for?
- For hierarchical clustering, which dissimilarity measure and type of linkage should be used, and where should we cut the dendrogram to obtain clusters?

Considerations:
- Validating clusters is not very easy. It is possible to assign p-values to clusters, but there is no general consensus (see Hastie et al, 2009).
- Mixture models can be used if a small subset of observations are different from each other and all other subgroups. Those observations may not belong to any cluster (see Hastie et al, 2009).
- Clustering is not very robust to perturbations in the data (removing subsets of observations).

Recommendations:
- Perform clustering with different chocies of parameters and tracking patterms
- Cluster subsets of the data to judge robustness of clusters
- Use clustering as a starting point for developing a hypothesis, and further study its results on an independent data set.

---
---
---
### V. Applications in R

{% highlight R %}
library("ISLR")
library("tidyverse")
{% endhighlight R %}

{% highlight R %}
#### PCA

states <- row.names(USArrests)
names(USArrests)

map_dbl(USArrests, mean) # Murder 7.788 Assault 170.76 UrbanPop 65.54  Rape 21.232 
map_dbl(USArrests, var) #  Murder 18.97 Assault 6945.2 UrbanPop 209.52 Rape 87.7

pca <- prcomp(USArrests, scale = TRUE)
names(pca)

# basic PCA info

pca$center # means
pca$scale # SDs
pca$rotation # principal component loadings
dim(pca$x) # principal component score vectors

biplot(pca, scale = 0) # graph of PC1 vs PC2

# proportion of variance explained

pca$sdev # SDs of each principle comp
pve <- (pca$sdev)^2 / sum((pca$sdev)^2) # calculate proportion of variance explained by each comp

plot(pve, xlab="Principal Component", ylab="Proportion of Variance Explained ", ylim=c(0,1),type='b')
plot(cumsum(pve), xlab="Principal Component ", ylab=" Cumulative Proportion of Variance Explained ", ylim=c(0,1), type='b')


#### Clustering

### K-Means Clustering
# kmeans(data, means, nstart = 50)

### Hierarchical lustering
# hclust(dist(data), method = "complete")

### Unsupervised Learning Application

nci_labs <- NCI60$labs
nci_data <- NCI60$data

dim(nci_data) # 64 x 6830
table(nci_labs)

## PCA
pca_out <- prcomp(nci_data, scale = TRUE)

Colors=function(vec){
  cols=rainbow(length(unique(vec)))
  return(cols[as.numeric(as.factor(vec))])
}

par(mfrow = c(1, 2))
plot(pca_out$x[, 1:2], col = Colors(nci_labs), pch = 19, xlab = "Z1", ylab = "Z2")
plot(pca_out$x[, c(1, 3)], col = Colors(nci_labs), pch = 19, xlab = "Z1", ylab = "Z3")
# -> cell lines from the same cancer type tend to have similar gene expression levels

summary(pca_out) # gives each comp, sd, and pve

pve <- 100 * pca_out$sdev^2 / sum(pca_out$sdev^2)
par(mfrow = c(1, 2))
plot(pve, type = "o", ylab = "PVE", slab = "Principle Component", col = "blue")
plot(cumsum(pve), type = "o", ylab = "Cumulative PVE", xlab = "Principal Component", col = "brown3")
# -> looks like we should examine 7 principal components


## Clustering

sd_data <- scale(nci_data) # scale data
data_dist <- dist(sd_data) # euclidian distance

# hierarchical
par(mfrow = c(1, 1))
plot(hclust(data_dist), labels = nci_labs, main = "Complete Linkage", xlab = "", sub = "", ylab = "")
plot(hclust(data_dist, method = "average"), labels = nci_labs, main = "Average Linkage", xlab = "", sub = "", ylab = "")
plot(hclust(data_dist, method = "single"), labels = nci_labs, main = "Single Linkage", xlab = "", sub = "", ylab = "")

hc_out <- hclust(dist(sd_data)) # use complete linkage hierarchical clustering
hc_out
hc_clusters <- cutree(hc_out, 4)
table(hc_clusters, nci_labs)

par(mfrow = c(1, 1))
plot(hc_out, labels = nci_labs)
abline(h = 139, col = "red")

# k-means
set.seed(2)
km_out <- kmeans(sd_data, 4, nstart = 20)
km_clusters <- km_out$cluster

table(km_clusters, hc_clusters)

# hierarchical clustering on the first few PC score vectors
hc_out_2 <- hclust(dist(pca_out$x[,1:5]))
plot(hc_out, labels = nci_labs, main = "Hier. Clust on First Five Score Vectors")
table(cutree(hc_out, 4), nci_labs)
{% endhighlight R %}
