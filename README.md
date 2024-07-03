
# KhulnaSoft DevSecOps Insights Orb for CircleCi

## What is the Insights Orb?

KhulnaSoft DevSecOps's Insights enables you check your container images for vulnerabilities. If your image has any known high-severity issue, Insights can fail the image build, making it easy to include as a step in your CI/CD pipeline.

The Insights itself is a small, easy to implement container vulnerabilty scanning tool. It has been embedded in the Insights Orb to be called upon during more complex workflows. The Insights has two modes: community and enterprise. This Orb allows for both modes. When used with a communitry mode scanning token as described on the Insights GitHub site token holders may run 100 scans a month  When an Enterprise KhulnaSoft Console is specified, the Insights will utilize a more granular, configurable scan policy.

> Note: The freely-available Community Edition is aimed at individual developers and open source projects who may not have control over the full CI/CD pipeline. The <a href="https://www.khulnasoft.com/use-cases/continuous-image-assurance/">KhulnaSoft DevSecOps commercial solution</a> is designed to be hooked into your CI/CD pipeline after the image build is complete, and/or to scan images from a public or private container registry.

> Another note: this freely-available Community Edition of Insights scans for vulnerabilities in the image's installed packages. KhulnaSoft's commercial customers have access to [additional Enterprise Edition scanning features](#khulnasoft-security-edition-comparison), such as scanning with a customized vulnerability policy, looking at vulnerable files, PII and other sensitive data included in a container image as well as audit logging.

The Insights Orb is an easy way to get started creating free, automated vulnerability assessment reports. These reports are posted to CircleCi's build artifact area.


## How to use the KhulnaSoft Insights Orb in your config.yml

### 1. Register for a Insights token as described [here.](https://github.com/khulnasoft/insights)

### 2. Create a context in the CircleCi portal and assign the required variables
    Navigate to Org Settings > Context and create a context


### 3. Within your new context create an environment variable named "KHULNASOFT_TOKEN" and populate it with the token from step #1

<p align="left">
  <img alt="Add the KHULNASOFT_TOKEN environment variable" src="https://github.com/khulnasoft/circleci-orb-insights/blob/master/images/contextEnvVar.png">
</p>

### 4. Add the required configuration to your build configuration

The following `.circleci/config.yml` is an example of a docker build configuration based on https://circleci.com/docs/2.0/building-docker-images/

```bash
# CircleCI build config example for implementation of the KhulnaSoft DevSecOps Insights
# https://github.com/khulnasoft/insights

version: 2.1

orbs:
  insights: khulnasoft/insights@dev:0.0.1

jobs:
  docker-build:
    executor: insights/default
    steps:
      - checkout
      - run: docker build -t circleci/node:latest .

workflows:
  scan-image:
    jobs:
      - docker-build
      - insights/scan-image:
          requires:
            - docker-build
          context: insights
          image: circleci/node:latest

```

Add the first 2 lines of the following example to the beginning of 
your `.circleci/config.yml` and set the CircleCI `version` to the minimum version `2.1`.

```shell
orbs:
  insights: khulnasoft/insights@dev:0.0.1
 ...
```

Then add a CircleCi executor.

```shell
 ...
jobs:
  docker-build:
    executor: insights/default
 ...
```

The final step is to edit your workflow to trigger a vulnerability scan.

```shell
 ...
workflows:
  scan-image:
    jobs:
      - docker-build
      - insights/scan-image:
          requires:
            - docker-build
          context: insights
          image: circleci/node:latest
```

## Viewing Scan Results
By default the Insights will pass a `0` for a passing scan (that is, a scan that has no high ranking vulnerabilities) and a `4` for a failing scan. This `4` of course stops the CircleCi process.

A report is created upon a failed scan. This is linked to within the CircleCi as an artifact. Navigate to the artifact tab in the CircleCi dashboard for viewing this report.
