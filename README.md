
# K means clustering - Lab

## Introduction

In this lab, we'll learn how to use scikit-learn's implementation of the K-Means Clustering algorithm to analyze a dataset!

## Objectives

You will be able to:

* Demonstrate an understanding of how the K-Means Clustering algorithm works
* Perform K-Means Clustering with scikit-learn and interpret results
* Use metrics such as Calinski Harabaz Scores (Variance Ratios) to determine the optimal number of clusters


## Understanding the K-Means Algorithm 

The k means clustering algorithm is an iterative algorithm that reaches for a pre-determined number of clusters within an unlabeled dataset, and basically works as follows:

- select k initial seeds
- assign each observation to the cluster to which it is "closest" 
- recompute the cluster centroids
- reassign the observations to one of the clusters according to some rule
- stop if there is no reallocation

## Creating a Dataset

For this lab, we'll create a synthetic dataset to work with, so that there are clearly defined clusters we can work with to see how well the algorithm performs. 

In the cell below:

* Import `make_blobs` from `sklearn.datasets`
* Import pandas, numpy, and matplotlib.pyplot, and set the standard alias for each. 
* Set matplotlib visualizations to display inline
* Use numpy to set a random seed of `1`.
* Import `KMeans` from `sklearn.cluster`


```python
import warnings
import pandas as pd
import numpy as np

from sklearn.datasets import make_blobs
from sklearn.cluster import KMeans
from sklearn.metrics import calinski_harabaz_score
from collections import defaultdict

import matplotlib.pyplot as plt
import seaborn as sns

warnings.filterwarnings('ignore')
plt.rcParams['figure.figsize'] = [10, 10]
plt.xkcd()
```




    <contextlib._GeneratorContextManager at 0x1a179bbba8>



Now, we'll use `make_blobs` to create our dataset. 

In the cell below:

* Call `make_blobs`, and pass in the following parameters:
    * `n_samples=400`
    * `n_features=2`
    * `centers=6`
    * `cluster_std=0.8`


```python
X, y = make_blobs(n_samples=400, n_features=2, centers=6, cluster_std=0.8)
```

Now, let's visualize our clusters to see what we've created. Run the cell below to visualize our newly created "blob" dataset.


```python
plt.scatter(X[:, 0], X[:, 1], c=y, s=80, alpha=0.5)
plt.show()
```


![png](index_files/index_5_0.png)


The nice thing about creating a synthetic dataset with `make_blobs` is that it can assign ground-truth clusters, which is why each of the clusters in the visualization above are colored differently. Because of this, we have a way to check the performance of our clustering results against the ground truth of the synthetic dataset. Note that this isn't something that we can do with real-world problems (because if we had labels, we'd likely use supervised learning instead!). However, when learning how to work with clustering algorithms, this provides a solid way for us to learn a bit more about how the algorithm works. 

## Using K-Means

Let's go ahead and create a `KMeans` object and fit it to our data. Then, we can explore the results provided by the algorithm to see how well it performs. 

In the cell below:

* Create a `KMeans` object, and set the `n_clusters` parameter to `6`.
* `fit()` the KMeans object to the data stored in `X`.
* Use the KMeans object to predict which clusters each data point belongs to by using the `Predict` method on the data stored in `X`.

Now that we have the predicted clusters, let's visualize them both and compare the two. 

In the cell below: 

* Create a scatter plot as we did up above, but this time, set `c=predicted_clusters`. The first two arguments and `s=10` should stay the same. 
* Get the cluster centers from the object's `.cluster_centers_` attribute. 
* Create another scatter plot, but this time, for the first two arguments, pass in `centers[:, 0]` and `centers[:, 1]`. Also set `c='black'` and `s=70`.


```python
k_means = KMeans(n_clusters=6)
k_means.fit(X)
predicted_clusters = k_means.predict(X)
centers = k_means.cluster_centers_

plt.scatter(X[:, 0], X[:, 1], c=predicted_clusters, s=50, alpha=0.5)
plt.scatter(centers[:, 0], centers[:, 1], s=200, c='k', alpha=0.5)
plt.show()
```


![png](index_files/index_8_0.png)


**_Question:_**

In your opinion, do the centroids match up with the cluster centers?

Write your answer below this line:
_______________________________________________________________________________



## Tuning Parameters

As you can see, the k means algorithm is pretty good at identifying the clusters. Do keep in mind that for a real data set, you will not be able to evaluate the method as such, as we don't know a priori what the clusters should be. This is the nature of unsupervised learning. The Scikit learn documentation does suggest two methods to evaluate your clusters when the "ground truth" is not known: the Silhouette coefficient and the Calinski-Harabaz Index. We'll talk about them later, but first, let's look at the Scikit learn options when using the KMeans function.

The nice hing about the scikit learn k-means clustering algorithm is that certain parameters can be specified to tweak the algorithm. We'll discuss two important parameters which we haven't specified before: `init` and `algorithm`.

### 1. The `init` parameter

`init` specifies the method for initialization:

- `k-means++` is the default method, this method selects initial cluster centers in a smart way in order to pursue fast convergence.
- `random`: choose k random observations for the initial centroids.
- `ndarray`: you can pass this argument and provide initial centers.

### 2. The `algorithm` parameter

`algorithm` specifies the algorithm used:

- If `full` is specified, a full EM-style algorithm is performed. EM is short for "Expectation Maximization" and its name is derived from the nature of the algorithm, where in each iteration an E-step (in the context of K-means clustering, the points are assigned to the nearest center) and an M-step (the cluster mean is updated based on the elements of the cluster) is created. 
- The EM algorithm can be slow. The `elkan` variation is more efficient, but not available for sparse data.
- The default is `auto`, and automatically selects `full` for sparse data and `elkan` for dense data. 

### Dealing With an Unknown Number of Clusters

Now, let's create another dataset. This time, we'll randomly generate a number between 3 and 8 to determine the number of clusters, without us knowing what that value actually is. 

In the cell below:

* Create another dataset using `make_blobs`. Pass in the following parameters:
    * `n_samples=400`
    * `n_features=2`
    * `centers=np.random.randint(3, 8)`
    * `cluster_std = 0.8`


```python
a = 3
b = 8
X_2, y_2 = make_blobs(n_samples=400, n_features=2, centers=np.random.randint(a, b), cluster_std=0.8)
```

Now, we've created a dataset, but we don't know how many clusters actually exist in this dataset, so we don't know what value to set for K!

In order to figure out the best value for K, we'll create a different version of the clustering algorithm for each potential value of K, and find the best one using an **_Elbow Plot_**.   

First, we'll need to create a different 

In the cell below, create and fit each `KMeans` object. Each one should be initialized with a different value for `n_clusters` between 3 and 7, inclusive.

Then, store each of the objects in a list. 


```python
set(y_2)
```




    {0, 1, 2, 3, 4, 5, 6}




```python
k_means_3 = None
k_means_4 = None
k_means_5 = None
k_means_6 = None
k_means_7 = None
k_dict = dict()
for cluster_size in range(a, b):
    k_dict[cluster_size] = KMeans(n_clusters=cluster_size).fit(X_2)

k_list = None
```

Now, in the cell below, import `calinski_harabaz_score` from `sklearn.metrics`. 


```python
# imported above
```

This is a metric used to judge how good our overall fit is. This score works by computing a ratio of between-cluster distance to inter-cluster distance. Intuitively, we can assume that good clusters will have smaller distances between the points in each cluster, and larger distances to the points in other clusters.

Note that it's not a good idea to just exhaustively try every possible value for k. As K grows, the number of points inside each cluster shrinks, until K is equal to the total number of items in our dataset. At this point, each cluster would report a perfect variance ratio, since each point is at the center of their own individual cluster! 

Instead, our best method is to plot the variance ratios, and find the **_elbow_** in the plot. Here's an example of the type of plot we'll generate:

<img src='elbow-method.png'>

In this example, the elbow is at K=3. This provides the biggest change to the CH score, and every one after that provides only a minimal improvement. 

In the cell below:

* Create an empty list called `CH_score`
* Loop through the models we stored in `k_list`. 
    * For each model, get the labels from the `.labels_` attribute.
    * Calculate the `calinski_harabaz_score` and pass in the data, `X_2`, and the `labels`. Append this score to `CH_score`


```python
CH_score = []
for clus, model in k_dict.items():
    score = calinski_harabaz_score(X_2, model.labels_)
    CH_score.append(score)


```

Now, let's create a visualization of our CH scores. 

Run the cell below to visualize our elbow plot of CH scores. 


```python
plt.plot(list(range(a, b)), CH_score)
plt.xticks(list(range(a, b)))
plt.title("Calinski Harabaz Scores for Different Values of K")
plt.ylabel("Variance Ratio")
plt.xlabel("# of clusters (K=)")
plt.show()
```


![png](index_files/index_19_0.png)


**_Question:_**  Interpret the elbow plot we just created. Where is the "elbow" in this plot? According to this plot, how many clusters do you think actually exist in the dataset we created?

Write your answer below this line:
_______________________________________________________________________________


Let's end by visualizing our `X_2` dataset we created, to see what our data actually looks like.

In the cell below, create a scatterplot to visualize our dataset stored in `X_2`. Set `c=y_2`, so that the plot colors each point according to its ground-truth cluster, and set `s=10` so the points won't be too big. 


```python
plt.scatter(X_2[:, 0], X_2[:, 1], c=y_2, s=50,alpha=0.5)
k = k_dict[6]
clusters = k.cluster_centers_
plt.scatter(clusters[:, 0], clusters[:, 1], c='k', s=300, alpha=0.8)
plt.show()
```


![png](index_files/index_21_0.png)


We were right! The data does actually contain six clusters. Note that are other types of metrics that can also be used to evaluate the correct value for K, such as silhouette score. However, checking the variance ratio by calculating Calinski Harabaz Scores is one of the most tried-and-true methods, and should definitely be one of the first tools you reach for when trying to figure out the optimal value for K with K-Means Clustering. 

## A Note on Dimensionality

We should also note that for this example, we were able to visualize our data because it only contained two dimensions. In the real world, working with datasets with only two dimensions is quite rare. This means that you can't always visualize your plots to double check your work. For this reason, it's extra important to be considerate about the metrics you use to evaluate the performance of your clustering algorithm, since you won't be able to "eyeball" it to visually check how many clusters the data looks like it has when you're working with datasets that contain hundreds of dimensions!


## Summary

In this lesson, we learned how to use the K-Means Clustering algorithm in scikit-learn. We also learned a strategy for finding the optimal value for K by using elbow plots and variance ratios, for when we're working with data and we don't know how many clusters actually exist. 

### Can we write this ourselves?


```python
X, y = make_blobs(n_samples=500, n_features=2, centers=4, cluster_std=0.8)
```


```python
plt.scatter(X[:, 0], X[:, 1], c=y, s=50)
```




    <matplotlib.collections.PathCollection at 0x1a176efb38>




![png](index_files/index_25_1.png)



```python
# first step of Kmeans is to randomly sample 4 points in the data
```


```python
# yay, we can randomly sample clusters!!!!
centroid_indices = np.random.choice(list(range(X.shape[0])), size=4)
centroids = X[cluster_indices]
centroids
```




    array([[  7.93677671,  -1.30303027],
           [ -3.57405574,  -9.71330101],
           [ -6.06342008,  -0.19349175],
           [ -3.6496763 , -10.40400219]])




```python
def euclidean_distance(x1, x2):
    return np.sqrt(np.sum((x1 - x2)**2))
```


```python
def get_center_of_mass(lst):
    arr = np.array(lst)
    return arr.mean(axis=0)
```


```python
euclidean_distance(np.array([0, 0]), np.array([3, 4]))
```




    5.0




```python
p1 = X[0]
p1
```




    array([8.28394882, 0.7155532 ])




```python
# Yay! We can make a list of points for each centroid!!!!!!!!!!!!!!!!
for iteration in range(5):
    print("iteration {}".format(iteration))
    centroid_dict = defaultdict(list)
    for point in X:
        # step 1: calculate the distance from our point to each centroid

        distances = np.zeros(shape=(centroids.shape[0], 1))
        for index, centroid in enumerate(centroids):
            dist = euclidean_distance(centroid, point)
            distances[index][0] = dist

        # get index of smallest distance in distances array
        centroid_index = distances.argmin()

        # use index to find centroid from centroids array
        chosen_centroid = centroids[centroid_index]
        chosen_centroid

        # append our point to that centroid's list of points
        centroid_dict[tuple(chosen_centroid)].append(list(point))
    
    
    new_centroids = []
    for centroid, lst in centroid_dict.items():
        new_centroids.append(get_center_of_mass(lst))
    centroids = np.array(new_centroids)
```

    iteration 0
    iteration 1
    iteration 2
    iteration 3
    iteration 4



```python
centroids
```




    array([[  7.14019737,   2.58521557],
           [ -2.75488123, -10.16217521],
           [ -4.50408603,  -1.3486194 ],
           [ -2.71634255,  -8.80985101]])




```python
kmean = KMeans(n_clusters=4)
```


```python
kmean.fit(X)
```




    KMeans(algorithm='auto', copy_x=True, init='k-means++', max_iter=300,
        n_clusters=4, n_init=10, n_jobs=1, precompute_distances='auto',
        random_state=None, tol=0.0001, verbose=0)




```python
kmean.cluster_centers_
```




    array([[ 6.14490573,  5.51652346],
           [-2.73792421, -9.56715256],
           [-4.50408603, -1.3486194 ],
           [ 8.13548902, -0.34609233]])


