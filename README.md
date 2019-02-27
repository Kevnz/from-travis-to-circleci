# Moving From TravisCI To CircleCI

With recent happenings with TravisCI

<blockquote class="twitter-tweet" data-lang="en"><p lang="en" dir="ltr">So apparently Travis CI is being strip-mined immediately after their acquisition by Idera. Sorry, I mean after &quot;joining the Idera family&quot; 🙄 <a href="https://t.co/CE5ERp1RsY">https://t.co/CE5ERp1RsY</a> A bunch of talented people are waking up to termination letters. Absolutely shameful. <a href="https://t.co/BbBRPdnswe">https://t.co/BbBRPdnswe</a></p>&mdash; Senior Oops Engineer (@ReinH) <a href="https://twitter.com/ReinH/status/1098663375985229825?ref_src=twsrc%5Etfw">February 21, 2019</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

## What they provided

With TravisCI I was able to setup a continuous build and test for my git repositories so every time I pushed I would get at least a test run against multiple versions of node.js just by including a `.travis.yml` file.

```yml
language: node_js
node_js:
  - "11"
  - "10"
  - "8"
  - "6"
```

But just because something is easy doesn't mean that we should accept the negative impact on actual people, so
I have decided that I no longer want to use their service.

## Enter the circle

I have used several continuous integration services as well as a number of "roll-your-own" options in the past. Of the services I have used I have found [CircleCI](https://circleci.com/) to be my current favourite service. So the decision was made and all that was left was to figure out how to get builds for all of those versions of node. Fortunately CircleCI lets you specify multiple jobs as part of a build and then you can put those jobs in any kind of execution order you want. For my NPM modules, I wanted to spin up multiple versions of node.js, checkout the code, install the dependencies for the module and then run my tests. Additionally I wanted the latest version of node.js (currently version 11) to take the test coverage results and submit them to [coveralls.io](https://coveralls.io).

### Step 1

Go to [Coveralls](https://coveralls.io) and setup an account if required and then add the repository you wish to gather coverage results for and get the token for that repository.


### Step 2

Go to [CircleCI](https://circleci.com) and setup an account if required and then add the repository you wish to start building.


### Step 3

Add the environment variable `COVERALLS_REPO_TOKEN` set to the token from [Coveralls](https://coveralls.io) (Though your test setup may be different, this covers using NYC or Tap)

### Step 4

Add `.circleci/config.yml` to the root of your project. That file should look like this:


```yml
version: 2
jobs:
  "node-11":
    docker:
      - image: circleci/node:11.9
    working_directory: ~/mydir
    steps:
      - checkout
      - run: npm install
      - run: npm test
      - run: npm run coverage
  "node-10":
    docker:
      - image: circleci/node:10
    working_directory: ~/mydir
    steps:
      - checkout
      - run: npm install
      - run: npm test

  "node-8":
    docker:
      - image: circleci/node:8
    working_directory: ~/mydir
    steps:
      - checkout
      - run: npm install
      - run: npm test

  "node-6":
    docker:
      - image: circleci/node:6
    working_directory: ~/mydir
    steps:
      - checkout
      - run: npm install
      - run: npm test

workflows:
  version: 2
  build:
    jobs:
      - "node-11"
      - "node-10"
      - "node-8"
      - "node-6"
```

This file defines four jobs, one for each version of node.js we want to test on (node-11, node-10, node-8, node-6). Then we define our workflows. In this case we have one workflow that executes all of the jobs in parallel.

When you push to github you can then check [CircleCI](https://circleci.com/) you should see your dashboard and your most recent jobs.

![Dashboard](/from-travis-to-circleci/CircleCI-App.png)

If you then click the build link shown in a job

![Build](/from-travis-to-circleci/CircleCI-Link.png)

It shows that workflow and you can see all the results of the workflow.

![workflow](/from-travis-to-circleci/CircleCI-Workflow.png)

Then you can drill down to each job and see the results from the steps in that job. This includes the setting up the environment, installing the dependencies, running the tests, and the coverage.

![Tests](/from-travis-to-circleci/CircleCI-Tests.png)

## Taking it further

With CircleCi workflows you can do so much more than just running tests. You can publish to GitHub pages, deploy the app, build docker images and publish them to docker hub, and so much more. Check out the [Docs](https://circleci.com/docs/2.0/workflows/) for more information.