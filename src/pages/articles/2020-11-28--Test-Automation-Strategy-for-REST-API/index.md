---
title: Test Automation Strategy for REST APIs
date: "2020-11-28T02:46:37.121Z"
layout: post
draft: false
path: "/blog/test-automation-strategy-for-rest-api/"
category: "api testing"
tags:
  - "api testing"
  - "test strategy"
description: "Test strategy example for building test harness focused on testing REST APIs"
---

Writing down even a short test (automation) strategy should come before any actual implementation. This document can help you to show to your boss and other team members that you have a plan and the action you make are on purpose. Last but not least it can help to get others involved in the whole process and make it rather a team effort instead of a one-man/woman show. The following text is a model test strategy I used for building a test automation framework for testing microservices.

## Mission
* Provide quick and reliable feedback on newly pushed changes into VSC with the aim of preventing regression function and non-functional bugs.
* Effective authoring of tests

## Strategy
To fulfill its mission the testing framework and authored tests must fulfill the following criteria:

1. Automated tests should cover the most business-critical parts of the tested system as these tests provide the biggest value by risk mitigation
2. Automated tests must be reliable and their outcome deterministic. If any of the tests does not adhere to this, it must be located and either fixed or disabled quickly. The testing framework should support this by providing statistics on test metadata and provide an overview of most often failing tests.
3. In case one of the tests fails, the testing framework must provide as much information as possible, that allows easy investigation and fast issue location. In practice, this means using the „fail messages" in assertions, central logging, and other practices.
4. It is necessary to treat test automation code as any other production code and use the same development practices to ensure the test automation efforts will keep maintainable over time.
5. Functional automated tests should run fast and the testing framework should support parallel test execution. Nonetheless, the test execution speed should not come at the cost of test execution reliability. It is better to have a slower test, that is reliable than a fast test providing a flaky outcome.
6. The outcome of automation efforts is a docker image published in the docker registry for easy and quick integration into the CI/CD tool. The testing framework must supports parametrization via environment variables. The list of variables must be documented in the readme.md file in the project repository.
7. Test framework must implement the request signature to allow easy request tracing.
8. Test stack must include reporting tool to allow further results aggregation and analysis and sharing additional test information to allow straightforward exploration of a failed test. Here is a couple of metrics the logged data should answer:
    * Execution time per individual test
    * Pass/Fail ration
    * Flaky tests overview
    * What projects/tests suites are executed most frequently

In the upcoming post, we will have a look at what tools can help us achieve the above-defined strategy.