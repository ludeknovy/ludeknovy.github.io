---
title: Playwright session recording with Jest Playwright and Jest circus
date: "2021-05-27T02:46:37.121Z"
layout: post
draft: false
path: "/blog/playwright-session-recoring-with-jest/"
category: "test automation"
tags:
  - "e2e"
  - "playwright"
description: "Tutorial on how to record playwright session and rename videos to match test description names"
---

Of course, we all want to have our build with UI tests green all the time, but unfortunately, there are occasions when someone pushes changes and not fully realizing their consequences and the build goes red. Woohoo, debugging time. Back in the days, I was used to recording videos during E2E run time, so it was kind of natural to have them for Playwright UI tests as well. 

Luckily Playwright has an API for it and what more, it is easily configurable to record the browser sessions automatically. All you need to do is to add the `recordVideo` property with a path where the videos will be saved. You pass this setting into browser context settings. In case you use `jest-playwright` you pass it to the `contextOptions`, like this: 
```
// jest-playwright.config.js
module.exports = {
  contextOptions: {
      recordVideo: {
            dir: "videos/"
      }
  },
  ...
}
```
Run your tests and you will find the video recordings in the `videos` folder. Cool, right? But if you have a second look you will notice that the videos have awkward names. And if you have multiple tests it is impossible to link actual tests and videos without opening them individually until you find them. That can be quite annoying while dealing with failed tests. Fortunately, this can be fixed with a little bit of code. 
To make it work you need to swap jasmin with jest-circus. That's very easy to do, just install it via npm `npm install --save-dev jest-circus` and the following line into your jest config file:
```
// jest.config.js
    ...
    "testRunner": "jest-circus/runner"
    ...
 ```

Now jest uses jest-circus as a test runner and it allows us to use its `handleTestEvent` method. The next step is to create a [custom jest environment](https://jestjs.io/docs/configuration#testenvironment-string) where we will react on jest events using the `handleTestEvent` method. The goal is to rename the videos after all the tests end and get the test name from jest circus event and change the video name later. Our custom environment could look like this:

```
// customEnvironment.js
const PlaywrightEnvironment = require("jest-playwright-preset/lib/PlaywrightEnvironment")
    .default
const pwConfig = require("./jest-playwright.config")


class CustomEnvironment extends PlaywrightEnvironment {
    constructor(config, context) {
        super(config, context)
        this.videos = []
    }
    async setup() {
        await super.setup()
        // Your setup
    }

    async teardown() {
        await super.teardown()
        // Your teardown
        if (this.videos.length !== 0) {
            const { recordVideo } = pwConfig.contextOptions

            this.videos.forEach(video => {
                const exists = fs.existsSync(video.videoName)
                if (exists) {
                    try {
                        const newVideoName = `${recordVideo.dir}/${video.testName}.webm`
                        fs.renameSync(video.videoName, newVideoName)
                    } catch(error) {
                        // eslint-disable-next-line no-console
                        console.log(error)
                    }
                }

            })
        }
    }

    async handleTestEvent(event) {

        if (event.name === "test_done") {
            const success = event.test.errors.length === 0 ? true : false
            const testName = event.test.name.replace(/ /g, "_")
            const videoName = await this.global.page.video().path()
            this.videos.push({ testName, videoName, success })
        }
    }
}

module.exports = CustomEnvironment

```

I guess you get the idea now. You can easily change the code to keep only failed tests with updated names if wanted. But before we run the tests to check it really works, we need to do one more change in jest config - to plug the custom environment in: 
```
//jest.config.js
...
    testEnvironment: "./customEnvironment.js",
...
```

That's it. Run your tests and from now on Playwright session recordings will match the test description names.

***

Read more about Playwright:
* [Code coverage of E2E with Playwright](https://www.ludeknovy.tech/blog/code-coverage-of-e2e-tests-with-playwright/)