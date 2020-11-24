---
title: Hands-on machine learning from linear regression
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

- Book: [Hands-on Machine Learning with Scikit-Learn, Keras, and TensorFlow, 2nd Edition](https://learning.oreilly.com/library/view/hands-on-machine-learning/9781492032632/) (try it free by your Microsoft account)
  - [Average rating 9.9 on Douban Book](https://book.douban.com/subject/30310982/) (till 11/24/2020)
  - [Errata](https://www.oreilly.com/catalog/errata.csp?isbn=0636920142874)
- Courses:
  - [Machine Learning 2020](http://speech.ee.ntu.edu.tw/~tlkagk/courses_ML20.html), Hung-yi Lee
  - [Machine Learning Crash Course](https://developers.google.com/machine-learning/crash-course/), Google
- Desserts: [Two Minute Papers](https://www.youtube.com/channel/UCbfYPyITQ-7l4upoX8nvctg)



## Recap of basic concepts

Do not worry about these theories if you can't catch up, just take it as an intro, this post will not introduce too many math knowledges also.

### Steps of machine learning

- Define the model, like linear model or neural network.
- Define the goodness/loss of model above, metrics can be error, cross entropy, etc.
- Calculate the best function by optimization algorithms.

### Linear model

Let's start with simplest linear model $y=ax+b$, and you can try more complex model if your train are underfitting.

Q: How to initialize parameters?

### Generalization

The model's ability to adapt properly to new, previously unseen data, drawn from the same distribution as the one used to create the model.

- Underfitting: model is too simple to learn the underlying structure of the data (large bias)
- Overfitting: model is too complex relative to the amount and noisiness of the training data (large variance)

![Goodness of fit](https://miro.medium.com/max/1400/1*iiPH0JyowvS3k12T0-W2HA.png)

Solutions: resources mentioned above, or [ref](https://towardsdatascience.com/underfitting-and-overfitting-in-machine-learning-and-how-to-deal-with-it-6fe4a8a49dbf).

### Loss function

We must have a dataset for training, it looks like: $(x_1, \hat{y_1})$, $(x_2, \hat{y_2})$, ..., $(x_n, \hat{y_n})$





## Further more

- Enjoy the reference

- Try assignments

  ![Learning map, http://speech.ee.ntu.edu.tw/~tlkagk/courses_ML20.html](http://speech.ee.ntu.edu.tw/~tlkagk/HW.png)

