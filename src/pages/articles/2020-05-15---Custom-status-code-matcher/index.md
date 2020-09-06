---
title: Custom status code matcher for API testing in Python with PyHamcrest
date: "2020-05-15T02:46:37.121Z"
layout: post
draft: false
path: "/blog/custom-status-code-matcher/"
category: "api testing"
tags:
  - "api testing"
  - "python"
description: "Tutorial on how to create custom status code matcher with Python and PyHamcrest"
---

Incorporating custom response code matcher in your codebase is going to increase the readability of your tests and make it a little bit more maintainable — transition to another http library would be easier, as you would need to change only custom matcher logic instead of changing assertion of all your tests. But the biggest benefit lies in vastly improved debugging experience and thus reducing the time spent exploring failed tests and gathering informations. In this article I am going to show you how you can achieve this in python using PyHamcrest library.

If you ever had written any API automated test you certainly used an assertion for status code to ensure given endpoint behaves correctly. Using build-in python assert statement the very simple test could look like this:

```python
response = request.get('https://base_url/endpoint')
assert response.status_code == 200
```

And at some point the assertion failed and it would produce following assertion error message:

```python
assert 500 == 200
where 500 = <Response [500]>.status_code
```

Which is ok, but unfortunately it won't provide you any other information e.g about error returned in response body by the server. We could improve the assert by adding response body as fail message:

```python
response = request.get('https://base_url/endpoint')
assert response.status_code == 200, response.text
```

This would produce error message like this:

```python
AssertionError: 
<Any API response body>
.....
assert 500 == 200
where 500 = <Response [500]>.status_code
```

Alright, that would help a lot for debugging when any test fails, but it is not as clean solution as it could be.To make the code more cleaner we are going to implement custom response status code matcher using PyHamcrest library. This specific custom matcher expects that your test code uses `requests` library for making actual HTTP request, but you can easily adjust it for arbitrary http library.

```python
# response_matcher.py
from hamcrest.core.base_matcher import BaseMatcherfrom 
from requests import Response

class ReturnedStatusCode(BaseMatcher):
    def __init__(self, status_code: int):
        self._status_code = status_code
    def _matches(self, response: Response):
        if not response.status_code:
            return False
        return response.status_code == self._status_code
    def describe_to(self, description):
        description.append_text("status code ")
        description.append_description_of(self._status_code)
    def describe_mismatch(self, item, mismatch_description):
        mismatch_description.append_text("status code ")
        mismatch_description.append_description_of(item.status_code)
        mismatch_description.append_text(" received with body: \n")
        mismatch_description.append_text(item.text)

def has_returned_status_code(status_code: int):
    return ReturnedStatusCode(status_code)
```

Now we just need to import it and its ready for use:

```python
from hamcrest import assert_that
from response_matcher import has_returned_status_code

response = request.get('https://base_url/endpoint')
assert_that(response, has_returned_status_code(200))
```

And in case of test failure it would now produce following assertion error:

```python
AssertionError: 
Expected: status code <200>
    but: status code <500> received with body:
    <API response will be printed here>
```

In my eyes `assert_that(response, has_returned_status_code(200))` is more elegant solution than `assert response.status_code == 200` and providing very useful information for quick change assessment detected by API test.

Happy API testing.