
# Building an SVM using scikit-learn - Lab

## Introduction

In the previous lab, you learned how to build an SVM from scratch. Next, you'll learn how to use scikit-learn to create SVMs!

## Objectives

You will be able to:
- Use scikit-learn to build a linear SVM when there are 2 groups
- Use scikit-learn to build a linear SVM when there are more than 2 groups

## Generate four data sets in scikit-learn

Let's use the scikit-learn dataset generator again. 
- The first data set contains the same blobs as for the first SVM in the last lab
- The second data set contains the same blobs as for the second SVM (Soft Margin Classifier) in the last lab
- The third data set contains four separate blobs
- The fourth data set contains slightly different data with two classes, yet this time the classes are not blobs but in the shape of half moons (generated using `make_moons`).


```python
from sklearn.datasets import make_blobs
from sklearn.datasets import make_moons
import matplotlib.pyplot as plt
%matplotlib inline  
import numpy as np

plt.figure(figsize=(10, 10))

plt.subplot(221)
plt.title("Two blobs")
X_1, y_1 = make_blobs(n_features = 2, centers = 2, cluster_std=1.25, random_state = 123)
plt.scatter(X_1[:, 0], X_1[:, 1], c = y_1, s=25)

plt.subplot(222)
plt.title("Two blobs with more noise")
X_2, y_2 = make_blobs(n_samples=100, n_features=2, centers=2, cluster_std=3,  random_state = 123)
plt.scatter(X_2[:, 0], X_2[:, 1], c = y_2, s=25)

plt.subplot(223)
plt.title("Four blobs")
X_3, y_3 = make_blobs(n_samples=100, n_features=2, centers=4, cluster_std=1.6,  random_state = 123)
plt.scatter(X_3[:, 0], X_3[:, 1], c = y_3, s=25)

plt.subplot(224)
plt.title("Two interleaving half circles")
X_4, y_4 = make_moons(n_samples=100, shuffle = False , noise = 0.3, random_state=123)
plt.scatter(X_4[:, 0], X_4[:, 1], c = y_4, s=25)

plt.show()
```


![png](index_files/index_6_0.png)


## A model for a perfectly linearly separable data set

Let's have a look at our first plot again. 


```python
X_1, y_1 = make_blobs(n_features = 2, centers = 2, cluster_std=1.25, random_state = 123)
plt.scatter(X_1[:, 0], X_1[:, 1], c = y_1, s=25);
```


![png](index_files/index_9_0.png)


We'll start with this data set and fit a simple linear support vector machine on these data. You can use the scikit-learn library `svm` to do so.
- import svm from scikit-learn
- save the SVC-method (which stands for Support Vector Classification) with `kernel='linear'` as the only argument.
- call the `.fit()` method with the data as the first argument and the labels as the second. 



```python
from sklearn import svm

clf = svm.SVC(kernel='linear')
clf.fit(X_1, y_1)
```




    SVC(C=1.0, cache_size=200, class_weight=None, coef0=0.0,
      decision_function_shape='ovr', degree=3, gamma='auto', kernel='linear',
      max_iter=-1, probability=False, random_state=None, shrinking=True,
      tol=0.001, verbose=False)



Let's save the first feature (on the horizontal axis) as X_11 and the second feature (on the vertical axis) as X_12.


```python
X_11= X_1[:,0]
X_12= X_1[:,1]
```


```python
clf.coef_
```




    array([[-0.4171222 ,  0.14320341]])



Next, let's store the min and maximum values X_11 and X_12 operate in. We'll use these minimum and maximum values to create our plots later. Add some slack (equal to 1) to the minimum and maximum boundaries.


```python
# plot the decision function
X11_min, X11_max = X_11.min() - 1, X_11.max() + 1
X12_min, X12_max = X_12.min() - 1, X_12.max() + 1
```

Next, we'll create a grid. You can do this by using the numpy function `linspace`, which creates a numpy array with evenly spaced numbers over a specified interval. The default of numbers is 50 and we don't need that many, so let's specify `num = 10` for now. You'll see that you need to take a higher number once we get to classification of more than 2 groups.


```python
x11_coord = np.linspace(X11_min, X11_max, 10)
x12_coord = np.linspace(X12_min, X12_max, 10)
```

To create our decision boundary, you'll need to create a mesh of points to plot in. You can do this by using `np.meshgrid` with the two arguments equal to the `np.linspace` objects created for X11 and X12.


```python
X12_C, X11_C = np.meshgrid(x12_coord, x11_coord)
```

Now we want to create a numpy array of the shape (100, 2) that concatenates the coordinates for X11 and X12 together in one numpy object. Use `np.c_` and make sure to use `.ravel()` first. Use `np.shape()` on your resulting object first to verify the resulting shape.


```python
x11x12 = np.c_[X11_C.ravel(), X12_C.ravel()]

np.shape(x11x12)
```




    (100, 2)



Bow we want to get a decision boundary for this particular data set. Using your (100,2) numpy array and calling `clf.decision_function()` on it, the decision function returns the distance to the samples that you generated using meshgrid. Make sure you change your shape in a way that you get a (10,10) numpy array.


```python
df1 = clf.decision_function(x11x12)
df1 = df1.reshape(X12_C.shape)
```

Now, let's plot our data again with the result of svm in it. 
- The first line is simply creating the scatter plot like before
- Next, you need to specify that what you will do next uses the same axes as the scatter plot. You can do this using `plot.gca()`. Store it in an object and for the remainder you'll use this object to create the lines in your plot
- Use `.countour()`. The first two argument are the coordinates created usiung the meshgrid, the third argument the result of your decision function call. 
- You'll want three lines: one decision boundary, and the 2 lines going through the support vectors. Incluse `levels = [-1,0,1]` to get all three.


```python
plt.scatter(X_11, X_12, c = y_1)
axes = plt.gca()
axes.contour(X11_C, X12_C, df1, colors=["blue","black","blue"], levels= [-1, 0, 1], linestyles=[':', '-', ':'])
plt.show()
```


![png](index_files/index_26_0.png)


The coordinates of the support vectors can be found in the `support_vectors_`-attribute. Have a look:


```python
clf.support_vectors_
```




    array([[ 1.27550827, -2.97755444],
           [-3.01370675, -1.50501182]])



Now create your plot again but highlight your support vectors. 


```python
plt.scatter(X_11, X_12, c = y_1)
axes = plt.gca()
axes.contour(X11_C, X12_C, df1, colors= "black", levels= [-1, 0, 1], linestyles=[':', '-', ':'])
axes.scatter(clf.support_vectors_[:, 0], clf.support_vectors_[:, 1], facecolors='blue') 
plt.show()
```


![png](index_files/index_30_0.png)


## When the data is not linearly separable

The previous example was pretty easy. The 2 "clusters" were easily separable by one straight line classifying every single instance correctly. But what if this isn't the case? Let's have a look at the second dataset we had generated.


```python
X_2, y_2 = make_blobs(n_samples=100, n_features=2, centers=2, cluster_std=3,  random_state = 123)
plt.scatter(X_2[:, 0], X_2[:, 1], c=y_2, s=25);
```


![png](index_files/index_33_0.png)


Unlike what we've seen in the previous lab, we can just simply use the same SVC function to this problem, as this algorithm automatically allows for slack variables. Repeat the code from above here, and plot the result.


```python
plt.scatter(X_2[:, 0], X_2[:, 1], c=y_2, s=25)

from sklearn import svm

clf = svm.SVC(kernel='linear')  #, C=1000000,
clf.fit(X_2, y_2)

X_21= X_2[:,0]
X_22= X_2[:,1]
X21_min, X21_max = X_21.min() - 1, X_21.max() + 1
X22_min, X22_max = X_22.min() - 1, X_22.max() + 1

x21_coord = np.linspace(X21_min, X21_max, 10)
x22_coord = np.linspace(X22_min, X22_max, 10)

X22_C, X21_C = np.meshgrid(x22_coord, x21_coord)

x21x22 = np.c_[X21_C.ravel(), X22_C.ravel()]

df2 = clf.decision_function(x21x22)
df2= df2.reshape(X21_C.shape)

plt.scatter(X_21, X_22, c = y_2)
axes = plt.gca()
axes.contour(X21_C, X22_C, df2, colors=["blue","black","blue"], levels= [-1, 0, 1], linestyles=[':', '-', ':'])
plt.show()
```


![png](index_files/index_35_0.png)


As you can see, 3 instances are misclassified (1 yellow, 2 purple). We probably can't do better in this situation, but it's worth to look at changing your hyperparameter C, which can be done in the .SCV command, adding a high value for the argument `C`. Set C = 5,000,000. 


```python
plt.scatter(X_2[:, 0], X_2[:, 1], c=y_2, s=25)

from sklearn import svm

clf = svm.SVC(kernel='linear', C = 5000000) 
clf.fit(X_2, y_2)

X_21= X_2[:,0]
X_22= X_2[:,1]
X21_min, X21_max = X_21.min() - 1, X_21.max() + 1
X22_min, X22_max = X_22.min() - 1, X_22.max() + 1

x21_coord = np.linspace(X21_min, X21_max, 10)
x22_coord = np.linspace(X22_min, X22_max, 10)

X22_C, X21_C = np.meshgrid(x22_coord, x21_coord)

x21x22 = np.c_[X21_C.ravel(), X22_C.ravel()]

df2 = clf.decision_function(x21x22)
df2= df2.reshape(X21_C.shape)

plt.scatter(X_21, X_22, c = y_2)
axes = plt.gca()
axes.contour(X21_C, X22_C, df2, colors=["blue","black","blue"], levels= [-1, 0, 1], linestyles=[':', '-', ':'])
plt.show()
```


![png](index_files/index_37_0.png)


## Other options in Scikit Learn



When you dig deeper in Scikit Learn, you'll notice that there are several ways to get to linear SVM's for classification:

- `svm.SVC(kernel = "linear")` , which we've used so far. Documentation can be found [here](https://scikit-learn.org/stable/modules/generated/sklearn.svm.SVC.html#sklearn.svm.SVC). 
- `svm.LinearSVC()`, which is very similar to the simple SVC method, but:
    - which does not allow for the keyword "kernel", as it is assumed to be linear (more on non-linear kernels later)
    - In the objective function, `LinearSVC` minimizes the squared hinge loss while `SVC` minimizes the regular hinge loss.
    - `LinearSVC` uses the One-vs-All (also known as One-vs-Rest) multiclass reduction while `SVC` uses the One-vs-One multiclass reduction (this is important only when having >2 classes!)
- `svm.NuSVC()`, which is again very similar,
    - Does have a "kernel" argument
    - `SVC` and `NuSVC` are essentially the same thing, except that for `nuSVC`, C is reparametrized into nu. The advantage of this is that where C has no bounds and can be any positive number, nu always lies between 0 and 1. 
    - One-vs-one multiclass approach.
    
    
So what does One-vs-one mean? what does One-vs-all mean?
- One-vs-one means that with $n$ classes, $\dfrac{(n)*(n-1)}{2}$ boundaries are constructed! 
- One-vs-all means that when there are $n$ classes, $n$ boundaries are created.

The difference between these three types of classifiers is mostly small, but generally visible for data sets with 3+ classes. Let's have a look at our third example and see how the results differ!

## Classifying four classes


```python
plt.figure(figsize=(7, 6))

plt.title("Four blobs")
X, y = make_blobs(n_samples=100, n_features=2, centers=4, cluster_std=1.6,  random_state = 123)
plt.scatter(X[:, 0], X[:, 1], c = y, s=25);
```


![png](index_files/index_41_0.png)


Try four different models and plot the results using subplots where:
    - The first one is a regular SVC (C=1)
    - The second one is a regular SVC with C=0.1
    - The third one is a NuSVC with nu= 0.7
    - The fourth one is a LinearSVC (no arguments)
    
Make sure all these plots have highlighted support vectors, except for LinearCSV (this algorithm doesn't have the attribute `.support_vectors_`. Here, instead of contour() use contourf() to get filled contour plots.


```python
X1= X[:,0]
X2= X[:,1]
X1_min, X1_max = X1.min() - 1, X1.max() + 1
X2_min, X2_max = X2.min() - 1, X2.max() + 1

x1_coord = np.linspace(X1_min, X1_max, 200)
x2_coord = np.linspace(X2_min, X2_max, 200)

X2_C, X1_C = np.meshgrid(x2_coord, x1_coord)

x1x2 = np.c_[X1_C.ravel(), X2_C.ravel()]

clf1 = svm.SVC(kernel = "linear",C=1) 
clf1.fit(X, y)
Z1 = clf1.predict(x1x2).reshape(X1_C.shape)

clf2 = svm.SVC(kernel = "linear",C=0.1) 
clf2.fit(X, y)
Z2 = clf2.predict(x1x2).reshape(X1_C.shape)

clf3 = svm.NuSVC(kernel = "linear",nu=0.7) 
clf3.fit(X, y)
Z3 = clf3.predict(x1x2).reshape(X1_C.shape)

clf4 = svm.LinearSVC() 
clf4.fit(X, y)
Z4 = clf4.predict(x1x2).reshape(X1_C.shape)

### 
plt.figure(figsize=(12, 12))

plt.subplot(221)
plt.title("SVC, C=1")
axes = plt.gca()
axes.contourf(X1_C, X2_C, Z1, alpha = 1)
plt.scatter(X1, X2, c = y, edgecolors = 'k')
axes.scatter(clf1.support_vectors_[:, 0], clf1.support_vectors_[:, 1], facecolors='blue', edgecolors= 'k') 

plt.subplot(222)
plt.title("SVC, C=0.1")
axes = plt.gca()
axes.contourf(X1_C, X2_C, Z2, alpha = 1)
plt.scatter(X1, X2, c = y, edgecolors = 'k')
axes.scatter(clf2.support_vectors_[:, 0], clf2.support_vectors_[:, 1], facecolors='blue', edgecolors= 'k') 

plt.subplot(223)
plt.title("NuSVC, nu=0.5")
axes = plt.gca()
axes.contourf(X1_C, X2_C, Z3, alpha = 1)
plt.scatter(X1, X2, c = y, edgecolors = 'k')
axes.scatter(clf3.support_vectors_[:, 0], clf3.support_vectors_[:, 1], facecolors='blue', edgecolors= 'k') 

plt.subplot(224)
plt.title("LinearSVC")
axes = plt.gca()
axes.contourf(X1_C, X2_C, Z4, alpha = 1)
plt.scatter(X1, X2, c = y, edgecolors = 'k')
plt.show()
```


![png](index_files/index_43_0.png)


Now, let's have a look at the coefficients of the decision boundaries. Remember that a simple `SVC` uses a one-vs-one method. this means that for 4 classes, $\dfrac{(4 * 3)}{2}= 6$ decision boundaries are created. The coefficients can be accessed in the attribute `.coef_`. Compare with the coefficients for the LinearSVC. What do you notice?


```python
print(clf2.coef_)

print(clf4.coef_)
```

    [[ 0.30750887 -0.22003386]
     [-0.00165148 -0.54016115]
     [-0.37433577 -0.27813528]
     [-0.35649837  0.13697254]
     [-0.19009595 -0.03838317]
     [-0.55475847 -0.24295554]]
    [[ 0.02240566 -0.33707085]
     [-0.34583713  0.19128388]
     [ 0.02200227 -0.05809006]
     [ 0.26165316  0.27936273]]


## To non-linear boundaries


```python
plt.title("Two interleaving half circles")
X_4, y_4 = make_moons(n_samples=100, shuffle = False , noise = 0.3, random_state=123)
plt.scatter(X_4[:, 0], X_4[:, 1], c = y_4, s=25)

plt.show()
```


![png](index_files/index_47_0.png)


Let's look at our fourth plot. We can try and draw a line here,  but it's pretty apparent that a linear boundary is not appropriate here. In the next section you'll learn about SVMs with non-linear boundaries!

## Additional reading

It is highly recommended to read up on SVMs in the scikit-learn documentation!
- https://scikit-learn.org/stable/modules/svm.html
- https://scikit-learn.org/stable/modules/generated/sklearn.svm.SVC.html
- https://scikit-learn.org/stable/modules/generated/sklearn.svm.LinearSVC.html
- https://scikit-learn.org/stable/modules/generated/sklearn.svm.NuSVC.html

## Summary

You now know how to use scikit-learn to build linear support vector machines. In the next lesson, you'll learn how SVMs can be extended to have non-linear boundaries.