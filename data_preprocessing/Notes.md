**Importing Libraries**
1. First anf foremost step is importing the third party libraries required to execute the code.

**Importing the dataset**
1. Dataset can be imported/loaded using the famous library i.e. pandas.
2. Pandas can read as well as write from different file format like csv, txt, excel, json etc.
3. There is one more method i.e. "pandas.read_fwf" which is used for reading fixed width file.
4. After importing the data, we have to create two major entities i.e. X,y. X is a group of independent features and y is a group of dependent features. 
5. Ideally in ML datasets dependent variables is always a last column.

*For below steps, if data is too huge data exploration is the key factor to understand data first then apply techniques used for below steps.
https://www.datacamp.com/community/tutorials/categorical-data*

**Taking care of missing data**
1. TODO: When to use which strategy for handling missing data

**Encoding categorical data**
TODO: how to apply one hot encoding  if there are large number of unique column values
1. One hot encoding is good for categorical encoding but it won't perform well if there are many categories. 
2. If there are lot of categories, we should use other encoding algorithms like binary encoding, base N encoding etc.
3. TODO: read about advanced encoding methods like Helmert contrast, polynomial contrast, backward difference
https://www.kdnuggets.com/2015/12/beyond-one-hot-exploration-categorical-variables.html

**Splitting the dataset into trainingset and test set**
When to apply feature scaling, i.e. before splitting data or after splitting data
We should always apply feature scaling after splitting data as if apply any scaling techniques before splitting it will be applied to considering whole data(train and test) which could create problem.


**Feature scaling**
1. It is applied to training dataset only.
2. It is the most crucial step in ML, Not all algorithms require feature scaling.
3. Algorithms which are based on some distance calculation i.e. KNN, K-means etc require feature scaling.
4. Neural Network , Deep Learning require feature scaling for fast convergence.
5. Algorithms like Random Forest doesn't have much impact of feature scaling. It is fine if we didn't apply feature scaling to those algorithms.
https://www.kdnuggets.com/2019/04/normalization-vs-standardization-quantitative-analysis.html