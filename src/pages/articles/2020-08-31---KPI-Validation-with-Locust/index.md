---
title: KPI validation with locust.io
date: "2020-08-31T02:46:37.121Z"
layout: post
draft: false
path: "/blog/kpi-validation-with-locust/"
category: "performance testing"
tags:
  - "locust.io"
  - "performance testing"
description: "Tutorial on how to create KPI validation for locust.io to automatically check whether application meets performance criteria."
---

In performance testing KPIs should be an inherent part of requirements, thus we don't blindly stress your application but rather we have clear idea what your stack should handle under defined circumstances. Real world example could be: “The 90 percentile should be below 50ms while handling 500 RPS (requests per second)”.

Once we fine tuned you application to meet defined performance criteria we can hook up this test into your CI/CD tool and run the same performance test as a part of your delivery pipeline to prevent performance regression. To fully automate whole process — we want want to look at the results only when the build has failed because of defined KPIs were not met — we need to add ability to decide whether to pass or fail to our Locust script. Locust doesn't have this feature baked in, but fortunately Locust is very hackable and provide nice API for customization.

For the purpose of automated KPI validation we are going to create custom plugin using locust's event hooks and its inner statistics. The overall plugin design is pretty simple:
1. Register quitting event
2. Get all statistics and serialize them
3. Calculate missing metrics (RPS, percentiles, …)
4. heck provided KPI definition
5. Validate provided KPI agains actual measurements

The whole KPI plugin code can be found [here](https://gist.githubusercontent.com/ludeknovy/af039ea46d568490cd8d09f5d0dad90d/raw/4c423298df3cd41ff42cc1ce89d78210f7ec9157/kpi_listener.py)

Now we have to register KpiPlugin class within your Locust script and define KPI(s) like this:

```python
events.init.add_listener
def on_locust_init(environment, **_kwargs):
    KPI_SETTINGS = [{'/store/inventory': [('percentile_90', 50), ('rps', 500), ('error_rate', 0)]}]
    KpiPlugin(env=environment, kpis=KPI_SETTINGS)
```

The above script will make your build fail in case /store/inventory endpoint won't meet one of the defined criteria — 90 percentile is worse than 50ms, RPS is under 500, error rate is higher than 0%.

Happy performance testing.