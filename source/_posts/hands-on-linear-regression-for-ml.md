---
title: Hands-on linear regression for machine learning
date: 2020-11-24 10:58:35
updated: 2020-11-24 10:58:35
categories:
- ML/DL
tags:
- machine learning
- linear regression
- regularization
- gradient descent
- feature engineering
mathjax: true
---

## Goal

Quick ramp to machine learning by linear regression case in about 1 hour, recap basic important concepts and practice with code.

<!-- more -->



## References

These are both references for this post as well as solid learning resources, if you could be or have been familiar with them, then you don't need this post 👻.

- Book: [Hands-on Machine Learning with Scikit-Learn, Keras, and TensorFlow, 2nd Edition](https://learning.oreilly.com/library/view/hands-on-machine-learning/9781492032632/) (try it free by your Microsoft account)
  - [Average rating 9.9 on Douban Book](https://book.douban.com/subject/30310982/) (till 11/24/2020)
  - [Errata](https://www.oreilly.com/catalog/errata.csp?isbn=0636920142874)
  - For Microsoft employees, acquire it [online](https://learning.oreilly.com/library/view/hands-on-machine-learning/9781492032632/) with your corporation account for free
- Courses:
  - [Machine Learning 2020](http://speech.ee.ntu.edu.tw/~tlkagk/courses_ML20.html), Hung-yi Lee
  - [Machine Learning Crash Course](https://developers.google.com/machine-learning/crash-course/), Google
- Desserts: [Two Minute Papers](https://www.youtube.com/channel/UCbfYPyITQ-7l4upoX8nvctg)



## Recap of basic concepts

Do not worry about these theories if you can't catch up, just take it as an intro, this post will not introduce too many math knowledges also.

### Steps of machine learning

0. Get familiar with dataset, do preprocessing works.
1. Define the model, like linear model or neural network.
2. Define the goodness/loss of model above, metrics can be error, cross entropy, etc.
3. Calculate the best function by optimization algorithms.

### Linear model

Let's start with simplest linear model $f(x)=wx+b$, you can also try more complex model if your train are underfitting.

*Question: How to initialize parameters?*

### Generalization

The model's ability to adapt properly to new, previously unseen data, drawn from the same distribution as the one used to create the model.

- Underfitting: model is too simple to learn the underlying structure of the data (large bias)
- Overfitting: model is too complex relative to the amount and noisiness of the training data (large variance)

![Goodness of fit](https://miro.medium.com/max/1400/1*iiPH0JyowvS3k12T0-W2HA.png)

*Solutions: resources mentioned above, or [ref](https://towardsdatascience.com/underfitting-and-overfitting-in-machine-learning-and-how-to-deal-with-it-6fe4a8a49dbf).*

### Loss/Cost function

There is a dataset for training, it looks like: $(x^1, \hat{y}^1)$, $(x^2, \hat{y}^2)$, ..., $(x^n, \hat{y}^n)$. The error of $x^i$ should be $\hat{y}^i-f(x)^i$, we can add all errors of dataset to define our loss function:

$$L(f)=\sum_{i=1}^{n}(\hat{y}^i-f(x^i))^2$$

$$L(w,b)=\sum_{i=1}^{n}(\hat{y}^i-(wx^i+b))^2$$

Obviously the smaller loss, the better model. So our target function should be:

$$f^\ast=\arg\min_{f} L(f)$$

Average value would be better than total sum, then we get the actual function that needs to be computed:

$$w^\ast,b^\ast=\arg\min_{w,b} \sum_{i=1}^{n}(\hat{y}^i-(wx^i+b))^2$$

Not big deal, just minize the mean square error of our trivial linear model.

### Vectorized form

You may have heard "feature" before, for each of data $x^i$, if the number of its features is $m$, then the actual model should be:

$$\hat{y}^i=w_0+w_1x_1^i+w_2x_2^i+\cdots+w_mx_m^i$$

Kind of verbose right? Let's use $\mathbf{w}$ to represent all feature weights $w_1$ to $w_m$ as well as the bias term $w_0$, which we call it $b$ before. Same way, use $\mathbf{x}$ to represent all the feature values $x_1$ to $x_m$ with $x_0$ is equal to 1. Then we can transform linear regression model to the vectorized form:

$$\hat{y}=h_w(x)=\mathbf{w}\cdot\mathbf{x}$$

Thus our loss function of vectorized form is:

$$\mathbf{w}^\ast=\frac{1}{n}\min_\mathbf{w} \sum_{i=1}^{n}(\hat{y}^i- \mathbf{w}^i\mathbf{x}^i)^2$$

Notice that $\mathbf{w}$ actually is $n*m$-dimensional matrix.

### Closed-form solution

As we already know the values of $\mathbf{x}$ and $\hat{y}$, it's easy to calculate the $\mathbf{w}$ by Normal Equation:

$$\hat{\mathbf{w}}=(\mathbf{x}^T\mathbf{x})^{-1} \mathbf{x}^T \mathbf{y}$$

Check out this [online course video](https://www.coursera.org/lecture/machine-learning/normal-equation-2DKxQ) (about 16min) from Andrew Ng to learn more.

Yes we're done. Our introduction is here.🤣🤣🤣

*Question: How to deal with complex models? How about computation burden?*

### Gradient Descent

> Gradient Descent is a generic optimization algorithm capable of finding optimal solution to a wide range of problems.
>
> Gradient descent is a first-order iterative optimization algorithm for finding a local minimum of a differentiable function.

Our loss function is differentiable indeed, so we can use it to find the local (also the global) minimum. Let's get it by one chart.

![Gradient Descent, credit to Hands-on ML](https://bn1301files.storage.live.com/y4mswG2flLOQG4BTxY2uv3jiGs1IbQ-VM8thBntnOmrtvJQGlthgCzW7nBj8fwkl6U7b1691jBXGBjxtsDb7dy7LF-FpJSTKouUhtfbrCRbZNUJNXt5M9WReSQKxdklCkhd7TNteQmNzZv89yr7hdMqCCgqKV9LZ1JK3nb1eWnNb2FohaTkcKiTTaVnezOQubsJ?width=2394&height=1186&cropmode=none)

So here is the last equation in this post (I promise, typing these LaTeX expression really wore me out), the gradient of our loss function:

$$\nabla{L} = \frac{\partial{L}}{\partial{\mathbf{w}}} = 2 \frac{1}{n} \sum_{i=1}^{n}(\hat{y}^i- \mathbf{w}^i\mathbf{x}^i) -\mathbf{x}^i$$

![Gradient Descent pitfalls, credit to Hands-on ML](https://bn1301files.storage.live.com/y4mXviZ8mp3JlnWxYxmhq62tUPn4MF0xLkK7Q-K63Mn1JsJwcX3tNZN2CpkbdT-3VuMKMcw_CAxtWu4XpjltS_VSYB35RZavo_zgNKd43bf3fATQhFMWbFU4v9GiYmbrTSYwbrExykkH7yZdYqKDRej4IN94CuZmKQ234pffSK3cXTPtQBQlDJgUGdySnzqIxsL?width=2554&height=1150&cropmode=none)

#### Variants optimizers

- SGD, Stochastic gradient descent
- Adam
- Mini-batch gradient descent
- Adagrad

### Training tips

I think maybe it's enough for us to dig into the code, so recap should be stopped here. Give this tips sections for some practical training methods.

- Hyperparameters tuning/optimization, like choose a good learning rate

- L2 (Ridge) regularization

- Early stopping

- Feature engineering

  - Feature selection by recursive feature elimination and cross-validation ([RFECV](https://scikit-learn.org/stable/modules/generated/sklearn.feature_selection.RFECV.html))

    ![Recursive feature elimination with cross-validation, credit to https://scikit-learn.org/](https://scikit-learn.org/stable/_images/sphx_glr_plot_rfe_with_cross_validation_001.png)

  - Feature scaling like normalization

  - Data correction for dirty part

  - Defining and removing outliers

  - Update model to make it fits dataset better like add high order term for most important feature, or even you can use a neural network if you want 😏

- Leveraging K-fold cross validation to split data and evaluate model performance



## Experiment environments

### Local

I highly recommend using Conda to run your Python code even on Unix-like OS, and [Miniconda](https://docs.conda.io/en/latest/miniconda.html) is good to get start.

> Conda as a package manager helps you find and install packages. If you need a package that requires a different version of Python, you do not need to switch to a different environment manager, because conda is also an environment manager. With just a few commands, you can set up a totally separate environment to run that different version of Python, while continuing to run your usual version of Python in your normal environment.

### Cloud

It's cloud computing era, we can write and save our code on the cloud and run it at anytime with any web client. Two cloud platforms will be introduced here, I suggest you try both of them and enjoy your experiment.

More specifically, these two products are all based on [Jupyter Notebook](https://jupyter.org/), which provides flexible Python run-time and Markdown document feature, it's easy to run code snippet just like on the local terminal.

#### Notebooks of Azure Machine Learning Studio

Here is a [brief introduction](https://docs.microsoft.com/en-us/azure/machine-learning/how-to-run-jupyter-notebooks) of Notebooks of AML Studio, and the advantages of this product are:

- IntelliSense and Monaco Editor adopted from Visual Studio Code are great.
- Rich sample notebooks are provided, and the tab view allows user to open several documents with several file types in one page.
- An one-stop platform for user to develop their machine learning project, you can take it as cloud IDE (Integrated Development Environment). For example, user can manager their huge datasets by Datasets product, and then consume them in Notebooks product.

![Notebooks of AML Studio](https://bn1301files.storage.live.com/y4m3DE0BuRvpw77hm-v1etWLkl-1ENO0FCEZ72iYUdpqa_kMkqSJx7VPzNlnLnhglTDuHjJggY5Bi7xBtgb8WX9BDLAZoHhyRnVy454EQDPLuE1IwySGJrXHvzSiL_PFqbClZn8x4URonUeSqBeprJXWa-IyVZv5AyByI-pEGKXgehthkyfFIlM8UJarHp62pH9?width=2776&height=1582&cropmode=none)

#### Google Colaboratory

You can open `ipynb` file on Google Drive by this product, there are also few advantages:

- Cleaner and larger workspace.
- "Code snippets" feature is interesting, but not smart enough (like intelligent recommendation), nor rich code exmaples.
- It will create compute target or VM (virtual machine) for the user automatically.
- You can download dataset from Google Drive easily.

![UI of Google Colab](https://bn1301files.storage.live.com/y4mNHPwPmALVZIF_KZgyK8_NBJfv4Wl3edLKx_iNaf8IICR9IXXjtoxFZ5mw6VGXOUB-RIT8IkeSia5amouV4zwfHJUD4fd6PJuMcstxvntd7TJZRAt5wPItx7L6aj_e61Y8fPGQa_NUtGbVYd2P7CzV1cweF8BnEv4XO_WOIoEHoGBKdOO1CEWby8h1FRtg002?width=2778&height=1588&cropmode=none)



## Code snippets

You can check the code example on Google Colab [here](https://colab.research.google.com/drive/131sSqmrmWXfjFZ3jWSELl8cm0Ox5ah3C), and code below will has slight differences.

### Target

To predict the PM2.5 value of first ten hour by other nine hours data.

### Data preprocessing

Original data structure looks like this:

|                     | 00:00 | 01:00 | ...  | 23:00 |
| ------------------- | ----- | ----- | ---- | ----- |
| Feature 1 of day 1  |       |       |      |       |
| Feature 2 of day 1  |       |       |      |       |
| ...                 |       |       |      |       |
| Feature 17 of day 1 |       |       |      |       |
| Feature 18 of day 1 |       |       |      |       |
| Feature 1 of day 2  |       |       |      |       |
| Feature 2 of day 2  |       |       |      |       |
| ...                 |       |       |      |       |

24 columns represent 24 hours, 18 features with every first 20 days of month in one year, we have 4320 (18\*20\*12) rows.

![Dataset preview in AML Studio](https://bn1301files.storage.live.com/y4mImqQmFAGS7aZxtYHvOYW7T9v-W25A2uAWpFRxJ2s5UXtn8MtiHALF_wPJsv9D0OXmYDlNLaDM1XupJqeMhu2itvtqoz3UqXxSODH13jY2UAuNZFzJFrYkjFAPYiYZYDFS3SU6pAYd_M8Di-68ZnplMS3zBoxPpqUSvAVGSTXHfV_NHQbBwKrCiV7armOD0wz?width=2782&height=1582&cropmode=none)

Our target data structure of $\mathbf{x}$ will be:

|                    | Feature 1 of 1st hour | Feature 1 of 2nd hour | ...  | Feature 1 of 9th hour | Feature 2 of 1st hour | ...  | Feature 18 of 9th hour |
| ------------------ | --------------------- | --------------------- | ---- | --------------------- | --------------------- | ---- | ---------------------- |
| 10th hour of day 1 |                       |                       |      |                       |                       |      |                        |
| 11st hour of day 1 |                       |                       |      |                       |                       |      |                        |
| ...                |                       |                       |      |                       |                       |      |                        |
| 24th hour of day 1 |                       |                       |      |                       |                       |      |                        |
| 1st hour of day 2  |                       |                       |      |                       |                       |      |                        |
| ...                |                       |                       |      |                       |                       |      |                        |

Number of columns should be $18\ast9=162$, and rows should be $(20\ast24-9)\ast12=5652$.

#### Preprocessing

```python
# Remove first useless columns: ID, Date, Feature name
data = data.iloc[:, 3:]
# Replace "NR" value by 0
data[data == 'NR'] = 0
raw_data = data.to_numpy()

def cook_raw(raw_data):
    month_data = {}
    for month in range(12):
        sample = np.empty([18, 480])
        for day in range(20):
            sample[:, day * 24 : (day + 1) * 24] = raw_data[18 * (20 * month + day) : 18 * (20 * month + day + 1), :]
        month_data[month] = sample

    x = np.empty([12 * 471, 18 * 9], dtype = float)
    y = np.empty([12 * 471, 1], dtype = float)
    for month in range(12):
        for day in range(20):
            for hour in range(24):
                if day == 19 and hour > 14:
                    continue
                x[month * 471 + day * 24 + hour, :] = month_data[month][:,day * 24 + hour : day * 24 + hour + 9].reshape(1, -1) #vector dim:18*9 (9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9 9)
                y[month * 471 + day * 24 + hour, 0] = month_data[month][9, day * 24 + hour + 9] #value
    x[x < 0] = 0

    return x, y

x, y = cook_raw(raw_data=raw_data)
```

#### Feature engineering by adding quadratice equation

```python
# Polynomial regression: quadratic equation
# 10th feature is PM2.5
x = np.concatenate((x, x[:, 9*9 : 10*9] ** 2), axis=1)
```

#### Normalization

```python
# Normalization
def _normalization(x):
  mean_x = np.mean(x, axis = 0) #18 * 9 
  std_x = np.std(x, axis = 0) #18 * 9 
  for i in range(len(x)): #12 * 471
      for j in range(len(x[0])): #18 * 9 
          if std_x[j] != 0:
              x[i][j] = (x[i][j] - mean_x[j]) / std_x[j]
  return x

x = _normalization(x)
```

#### Feature engineering by pruning unimportant features

```python
# Delete features to prevent overfitting
def prune(x):
  delete_cols = []
  # Remove trivial features: NOx(#7), RAINFALL(#11)
  remove_idx = [6, 10]
  for i in remove_idx:
    delete_cols.extend(range(i * 9 + 1, (i + 1) * 9 + 1))

  res = np.delete(x, delete_cols, 1)
  return res

# Initialize bias values with 1
x_pruned = prune(np.concatenate((np.ones([12 * 471, 1]), x), axis = 1).astype(float))
```

#### Split training data into training set and validation set

```python
x_train_set = x[: math.floor(len(x) * 0.8), :]
y_train_set = y[: math.floor(len(y) * 0.8), :]
x_validation = x[math.floor(len(x) * 0.8): , :]
y_validation = y[math.floor(len(y) * 0.8): , :]
```

### Training and prediction

#### Rough training

```python
def eval_loss(X, Y, w):
  return np.sqrt(np.sum(np.power(X @ w - Y, 2))/X.shape[0])

# Batch gradient descent
def train(X, Y, w = 0, reg = 1, iter = 8000):
  dim = X.shape[1]
  if type(w) == int:
    w = np.zeros([dim, 1])

  learning_rate = 1.6
  adagrad = np.zeros([dim, 1])
  eps = 0.0000000001
  for t in range(iter):
    loss = eval_loss(X, Y, w)
    if(t%500==0):
        print('#' + str(t) + ":" + str(loss))
    # Ridge regularization
    gradient = 2 * (X.T @ (X @ w - Y)) + 2 * reg * w
    # Learning schedule by Adagrad
    adagrad += gradient ** 2
    w = w - learning_rate * gradient / np.sqrt(adagrad + eps)
  return w

w = train(x_train_set, y_train_set)
```

#### Validate training

```python
eval_loss(x_validation, y_validation, w)
```

#### Training again and remove outliers

```python
w = train(X = x_pruned, Y = y, w = w)

outliers = []
for i in range(x_pruned.shape[0]):
  if np.absolute(x_pruned[i] @ w - y[i]) > 10:
    outliers.append(i)

# Try to eliminate irreducible error
x_pruned = np.delete(x_pruned, outliers, 0)
y = np.delete(y, outliers, 0)

w = train(X = x_pruned, Y = y, w = w)
print('\nFinal loss on full training dataset: {}'.format(eval_loss(x_pruned, y, w)))
```



## Further more

- Enjoy the references

- Try assignments

  ![Learning map, http://speech.ee.ntu.edu.tw/~tlkagk/courses_ML20.html](http://speech.ee.ntu.edu.tw/~tlkagk/HW.png)

