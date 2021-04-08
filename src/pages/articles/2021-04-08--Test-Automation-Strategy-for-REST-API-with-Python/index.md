---
title: Test Automation Strategy for REST APIs with Python — Tooling
date: "2021-04-08T02:46:37.121Z"
layout: post
draft: false
path: "/blog/test-automation-strategy-for-rest-api-with-python/"
category: "api testing"
tags:
  - "api testing"
  - "test strategy"
description: "Automated REST APIs testing with Python"
---

[In the previous post](https://www.ludeknovy.tech/blog/test-automation-strategy-for-rest-api/), I gave you a real-world example of a test automation strategy for REST APIs, and thus we know what we expect from our test harness. In today’s post, we are going to look at tooling which will help us to fulfill the strategy by using open source technologies only. As already hinted in the title I’d chosen Python as a programming language. The main reason behind this decision is very simple — its ecosystem of already existing tools allowed me to effectively fulfill the before-defined test strategy. Without further ado here is the list of tools used: locust.io, pytest, Schemathesis, PyHamcrest, ReportPortal. In the following paragraphs, I will try to explain how these tools support the test strategy.

### Locust and pytest
To achieve very high code reusability and fast authoring it is necessary to write performance tests in code. This way we can model the API calls to be used for functional and non-functional tests. For this reason, I’ve chosen locust.io as a performance testing tool and pytest as a functional testing framework. In the past, I used JMeter as well, but the tool itself does not allow storing the tests in code, but you have to store it in XML format, which is very difficult for tracking changes in the versioning system as it is not very readable compared to plain python code. And if you use JMeter or any other tool it requires you to define and maintain the API calls at two places — for the functional tests and the performance tests themselves. The Locusts standard request interface (Locust also provides FastHttp which has a slightly different request interface) shares the same interface with the requests python library which makes the reusability very easy to achieve.

### Schemathesis
Another very important tool increasing the effectiveness of authoring tests is schemathesis — especially if you have OpenAPI definition at your disposal. Schemathesis can be used not only to assert that API response conforms to API definition but can be used to do property-based testing for specified API input. This means two things — it allows you to get rid of lots of code by checking the response against OpenAPI definition instead of manually defining all the properties in code, and it supports clever property-based test case generation, thus reducing tons of time spent on negative test cases preparation.

### PyHamcrest
When a test fails it’s very important to provide enough information on what went wrong. PyHamcrest can be used to create a custom matcher and include domain-specific debug information as tracing id, service version, etc. I mainly use it for creating custom response matcher. Incorporating custom response code matcher in your codebase is going to increase the readability of your tests and make it a little bit more maintainable — transition to another HTTP library would be easier, as you would need to change only custom matcher logic instead of changing the assertion of all your tests. But the biggest benefit lies in vastly improved debugging experience and thus reducing the time spent exploring failed tests and gathering information.

### ReportPortal
The key part of a good testing infrastructure is test reporting. You know the situation when your tests fail and you have to open your CI and search in very long output to find the assertion error. And this gets even more frustrating when there is a couple of errors. With ReportPortal you will be able to navigate through your failed test in a more sane way. You can upload to ReportPortal other information collected during test runs such as communication logs. You will get the assertion error and test logs in one place, with that you will be able to find out what went wrong pretty quickly and easily. But the benefits of using ReportPortal do not stop here. It will track the history of your test results and allow you to spot repeating problems more easily. All that data gathered in ReportPortal gives you more detailed statistical insight into your tests and application under test. You will be able to visualize the pass/fail ratio, average run time, execution time per individual test, and more. Regular CI tool history limits your statistics data to pass/fail statistics over a couple of last builds. But that’s it. You do not get statistics on which particular tests have failed. With ReportPortal you will be able to get an overview of most often failing tests.