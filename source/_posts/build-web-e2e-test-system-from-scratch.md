---
title: Build web E2E test system from scratch
date: 2019-10-6 17:31:00
updated: 2019-10-19 17:50:00
categories:
- Web 前端
tags:
- e2e test
- javascript
- nodejs
- ui test
- jest
- testcafe
- puppeteer
- kusto
- geneva
---

## TL;DR

The topic of end-to-end (E2E) test is a platitude in web front-end (FE) domain, especially most of blogs are talking about testing frameworks out of the real web production. The real world needs solution, for E2E test which means a service can do E2E testing and alerting with stability and even good performance. This article will introduce how we design and build our E2E test service for the newer version of our [web application](https://docs.microsoft.com/en-us/azure/machine-learning/studio/what-is-ml-studio) based on popular web FE frameworks and integration with abundant Microsoft services, including several workarounds to tell the problems we met and how we overcame them.

In addition, it's for internal review at first, then it becomes my first blog article in English. Probably I will translate it into Chinese sooner or later.

<!-- more -->



## Introduction

> "End-to-end testing involves ensuring that the integrated components of an application function as expected. The entire application is tested in a real-world scenario such as communicating with the database, network, hardware and other applications." Defined by Techopeida. 

This topic is a platitude in web FE domain, especially most of online blogs are talking about testing frameworks out of the real web production. When we were facing the task to build E2E test for the newer version of our web application, these questions came across my mind:

- How did previous version of our production (V1) do E2E testing?

- Can we leverage V1's work?

- If the legacy solution doesn't meet our requirement and we need to build another one from scratch...

- - Which FE test framework is the best for us?
  - How can we run E2E test as service with stability?
  - How to associate our test data with internal alerting service?

As you can see, there were bunch of questions for us to figure out. It costed two teammates' and my 2 weeks full-time work and another 2 months part-time work (about 1 day per week) of mine to deliver the first **acceptable** version, though test service had been online after first two weeks work with some workarounds.

The following parts will cover the answers for questions above, several workarounds we made, and an use case to help our team achieve more 😊.



## Legacy solution

