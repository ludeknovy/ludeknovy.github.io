---
title: TimescaleDB setup for Locust real-time monitoring
date: "2021-01-15T02:46:37.121Z"
layout: post
draft: false
path: "/blog/locust-timescaledb-setup/"
category: "performance testing"
tags:
  - "performance testing"
  - "locust"
description: "Setting up TimescaleDB for monitoring your Locust performance tests"
---

The Locust plugin repository contains TimescaleDB listener to log performance stats. But unfortunately for some reason, it is very difficult to do the initial TimescaleDB setup. The provided schema did not work for me. There is an [issue](https://github.com/SvenskaSpel/locust-plugins/issues/11) for it with possible mitigation and my solution for it. I took the original schema and modified it, so it works out of the box when you include it into docker.
You can find the adjusted schema in my [repository]([locust-timescaledb/schema.sql at main · ludeknovy/locust-timescaledb · GitHub](https://github.com/ludeknovy/locust-timescaledb/blob/main/schema.sql)).

Then build it: `docker build -t timescale .`
And run the docker image: `docker run -d --name timescaledb -p 5432:5432 -e POSTGRES_PASSWORD=password timescale`

Now you can use the prepared TimescaleListener from locust-plugins repository. To include it in your test all you have to do is:

```@events.init.add_listener
def on_locust_init(environment, **_kwargs):
    TimescaleListener(env=environment, testplan=„timescale_listener_ex“, target_env=„myTestEnv“)
```

Now you might need to provide a user and password in `create_dbconn` in `TimescaleListener` and set up `PGHOST` env var.

Now you are all set up and you can start your test and new records should start popping up in TimescaleDB.

To visualize the results you have to connect your TimescaleDB to 
Grafana. The team behind locust-plugins has prepared a dashboard definition you can easily import and have a very robust monitoring solution for your performance tests.