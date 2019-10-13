# Mixed Naive Bayes

Naive Bayes classifiers are a set of supervised learning algorithms based on applying Bayes' theorem, but with strong independence assumptions between the features given the value of the class variable (hence naive).

This module implements **Categorical** (Multinoulli) and **Gaussian** naive Bayes algorithms (hence *mixed naive Bayes*). This means that we are not confined to the assumption that features (given their respective *y*'s) follow the Gaussian distribution, but also the categorical distribution. Hence it is natural that the continuous data be attributed to the Gaussian and the categorical data (nominal or ordinal) be attributed the the categorical distribution.

The motivation for writing this library is that [scikit-learn](https://scikit-learn.org/) does not have an implementation for mixed type of naive bayes. They have one for `CategoricalNB` [here](https://github.com/scikit-learn/scikit-learn/blob/86aea9915/sklearn/naive_bayes.py#L1021) but it's still in its infancy.

I like `scikit-learn`'s APIs  😍 so if you use it a lot, you'll find that it's easy to get started started with this library (there's `.fit()`, `.predict()`, `.predict_proba()` and `.score()`).

I've also written a tutorial [here](https://remykarem.github.io/blog/naive-bayes) for naive bayes if you need to understand a bit more on the math.

## Contents

- [Installation](#installation)
- [Quick start](#quick-start)
- [Requirements](#requirements)
- [Performance (Accuracy)](#performance-accuracy)
- [Performance (Speed)](#performance-speed)
- [Tests](#tests)
- [API Documentation](#api-documentation)
- [To-Dos](#to-dos)
- [References](#references)
- [Related work](#related-work)
- [Contributing ️❤️](#contributing)

## Installation

### via pip

```bash
pip install mixed-naive-bayes
```

or

```bash
pip install git+https://github.com/remykarem/mixed-naive-bayes#egg=mixed-naive-bayes
```

## Quick starts

### Example 1: Discrete and continuous data

Below is an example of a dataset with discrete (first 2 columns) and continuous data (last 2). Specify the indices of the features which are to follow the categorical distribution (columns `0` and `1`). Then fit and predict as per usual.

```python
from mixed_naive_bayes import MixedNB
X = [[0, 0, 180.9, 75.0],
     [1, 1, 165.2, 61.5],
     [2, 1, 166.3, 60.3],
     [1, 1, 173.0, 68.2],
     [0, 2, 178.4, 71.0]]
y = [0, 0, 1, 1, 0]
clf = MixedNB(categorical_features=[0,1])
clf.fit(X,y)
clf.predict(X)
```

**NOTE: The module expects that you treat the categorical data be label encoded accordingly. See the following example to see how.**

### Example 2: Discrete and continuous data

Below is a similar same dataset. However, for this dataset we assume a categorical distribution on the first 3 features, and a Gaussian distribution on the last feature. Feature 3 however has not been label-encoded. We can use sklearn's [`LabelEncoder()`](https://scikit-learn.org/stable/modules/generated/sklearn.preprocessing.LabelEncoder.html) preprocessing module to fix this.

```python
from sklearn.preprocessing import LabelEncoder
X = [[0, 0, 180, 75.0],
     [1, 1, 165, 61.5],
     [2, 1, 166, 60.3],
     [1, 1, 173, 68.2],
     [0, 2, 178, 71.0]]
y = [0, 0, 1, 1, 0]
X = np.array(X)
y = np.array(y)
label_encoder = LabelEncoder()
X[:,2] = label_encoder.fit_transform(X[:,2])
print(X)
# array([[ 0,  0,  4, 75],
#        [ 1,  1,  0, 61],
#        [ 2,  1,  1, 60],
#        [ 1,  1,  2, 68],
#        [ 0,  2,  3, 71]])
```

Then fit and predict as usual, specifying `[0,1,2]` as the indices that we assume categorical distribution.

```python
from mixed_naive_bayes import MixedNB
clf = MixedNB(categorical_features=[0,1,2])
clf.fit(X,y)
clf.predict(X)
```

### Example 3: Discrete data only

If all columns are to be treated as discrete, specify `categorical_features='all'`.

```python
from mixed_naive_bayes import MixedNB
X = [[0, 0],
     [1, 1],
     [1, 0],
     [0, 1],
     [1, 1]]
y = [0, 0, 1, 0, 1]
clf = MixedNB(categorical_features='all')
clf.fit(X,y)
clf.predict(X)
```

**NOTE: The module expects that you treat the categorical data be label encoded accordingly. See the previous example to see how.**

### Example 4: Continuous data only

If all features are assumed to follow Gaussian distribution, then leave the constructor blank.

```python
from mixed_naive_bayes import MixedNB
X = [[0, 0],
     [1, 1],
     [1, 0],
     [0, 1],
     [1, 1]]
y = [0, 0, 1, 0, 1]
clf = MixedNB()
clf.fit(X,y)
clf.predict(X)
```

### More examples

See the `examples/` folder for more example notebooks or jump in to a notebook hosted at MyBinder [here](https://mybinder.org/v2/gh/remykarem/mixed-naive-bayes/master?filepath=%2Fexamples%2Fdataset_digits.ipynb).

## Requirements

- `Python>=3.6`
- `numpy>=1.16.1`

The `scikit-learn` library is used to import data as seen in the examples. Otherwise, the module itself does not require it.

The `pytest` library is not needed unless you want to perform testing.

## Performance (Accuracy)

Measures the accuracy of (1) using categorical data and (2) my Gaussian implementation.

Dataset | GaussianNB | MixedNB (G) | MixedNB (C) | MixedNB (G+C) |
------- | ---------- | ----------- | ----------- | ------------- |
Iris    | 0.960      | 0.960       | -           | - |
Digits  | 0.858      | 0.858       | **0.961**   | - |
Wine    | 0.989      | 0.989       | -           | - |
Cancer  | 0.942      | 0.942       | -           | - |
covtype | 0.616      | 0.616       |             | **0.657** |

G - Gaussian only
C - categorical only
G+C - Gaussian and categorical

## Performance (Speed)

The library is written in [NumPy](https://numpy.org/), so many operations are vectorised and faster than their for-loop counterparts. Fun fact: my first prototype (with many for-loops) took me 8 times slower than sklearn's 😱.

(Still measuring)

## Tests

I'm still writing more test cases, but in the meantime, you can run the following:

```bash
pytest tests.py
```

## API Documentation

For more information on usage of the API, visit [here](https://remykarem.github.io/docs/mixed_naive_bayes.html). This was generated using pdoc3.

## To-Dos

- [ ] Implement `predict_log_proba()`
- [ ] Write more test cases
- [ ] Performance (Speed)
- [X] Support refitting
- [X] Regulariser for categorical distribution
- [X] Variance smoothing for Gaussian distribution
- [X] Vectorised main operations using NumPy

Possible features:

- [ ] Masking in NumPy
- [ ] Support label encoding
- [ ] Support missing data

## References

- [scikit-learn's naive bayes](https://scikit-learn.org/stable/modules/classes.html#module-sklearn.naive_bayes)

## Related Work

- [Categorical naive Bayes by scikit-learn](https://scikit-learn.org/dev/modules/generated/sklearn.naive_bayes.CategoricalNB.html)
- [Naive Bayes classifier for categorical and numerical data](https://github.com/wookieJ/naive-bayes)
- [Generalised naive Bayes classifier](https://github.com/ashkonf/HybridNaiveBayes)

## Contributing ️❤️

Please submit your pull requests, will appreciate it a lot ❤
