---
layout: post
title:  "Faster deployments of Azure Premium Functions with GitHub Actions"
author: Nik Osvalds
date:   2021-09-10
categories: blog
---

Increase deployment speed of Azure Premium SKU Functions from GitHub Actions through use of Docker containers and an Azure maintained action.

## Background

The IATI Technical team has been working hard over the last year on delivering a new Unified Platform for IATI services based on a microservice architecture in Azure.

We use Azure (Serverless) Functions written in Node.js for our microservices. The Function code is managed in GitHub and deployed using GitHub Actions workflows. 

## Problem

One of our Consumption tier Functions [IATI/validator-services](https://github.com/IATI/validator-services) was hitting a file size limit as we work with up to 60 MB XML files for the IATI Standard. So we needed to upgrade from a Consumption tier SKU to a Premium tier SKU for this function.

After doing this upgrade (which did involve removing and redeploying the Function since we're using Linux based Functions), we noticed that our deployment times increased significantly. From about 1min 15sec on average to over 15min. 

We were using the Azure maintained [Azure/functions-action](https://github.com/Azure/functions-action) for deployment from GitHub Actions. Watching the deployment logs (Azure Portal > Your Function> Deployment Center > Logs) while the deployment was happening, I noticed that the longest time was spent on KuduSync for copying files:

```
2021-09-08T17:36:43.1221316Z,Omitting next output lines...
2021-09-08T17:37:03.7262014Z,Processed 1006 files...
2021-09-08T17:37:23.7327714Z,Processed 2482 files...
2021-09-08T17:37:43.7330765Z,Processed 3940 files...
2021-09-08T17:38:03.7386083Z,Processed 5485 files...
2021-09-08T17:38:23.8800551Z,Processed 7085 files...
2021-09-08T17:38:43.8839515Z,Processed 8467 files...
2021-09-08T17:39:03.8845949Z,Processed 9828 files...
2021-09-08T17:39:23.9059631Z,Processed 11371 files...
2021-09-08T17:39:43.9081679Z,Processed 12600 files...
2021-09-08T17:40:03.9098378Z,Processed 13959 files...
2021-09-08T17:40:23.9108179Z,Processed 15139 files...
2021-09-08T17:40:43.9161550Z,Processed 16375 files...
2021-09-08T17:41:03.9283262Z,Processed 17970 files...
2021-09-08T17:41:23.9332671Z,Processed 19246 files...
2021-09-08T17:41:43.9448335Z,Processed 20742 files...
```

Using my searching skills in the GitHub repo for [projectkudu/kudu](https://github.com/projectkudu/kudu) I found an [issue](https://github.com/projectkudu/kudu/issues/2602) that described a similar problem with Node.js deployments using Kudu. According to this issue:

`The root of the problem is simply the fact that a XX,XXX-count file operation onto the site volume is going to be slow.`

Great, I knew what the issue was now, but this also described no solution to the problem and with the way the issue was closed (and the age) didn't give me any hope for future improvement.

Fortunately I had been deploying a Premium SKU Function [IATI/js-validator-api](https://github.com/IATI/js-validator-api) due to a requirement of a customised Functions image using [Azure/functions-container-action](Azure/functions-container-action). I noticed that deployment times for this were much quicker (~3min) than with [Azure/functions-action](https://github.com/Azure/functions-action).

## The Solution

Through the troubleshooting and research steps described above, I realised I could set up my Function to use [Azure/functions-container-action](Azure/functions-container-action) and take advantage of the speedier deployment times offered there.

### Steps

- Deleted my existing Premium SKU Function that was using Kudu deployment.

  - Note: It may be possible just to switch this Function to using Docker deployment, however I've had issues with this in the past where the new code would not deploy from the Docker image.

- Using the relevant parts of this guide: [Create a function on Linux using a custom container](https://docs.microsoft.com/en-us/azure/azure-functions/functions-create-function-linux-custom-image?tabs=in-process%2Cbash%2Cazure-cli&pivots=programming-language-javascript)

  - Created a [Dockerfile](https://github.com/IATI/validator-services/blob/develop/Dockerfile) in my Function repo. Note that I make no customisations to the image as I don't need them for this Function.
  - Create a [.dockerignore](https://github.com/IATI/validator-services/blob/develop/.dockerignore) file to ensure only the necessary files are included in the image
- Created a new Function in the Azure Portal

  - Make sure to choose the `Publish: Docker Container` option on the Basics tab

- Modified the GitHub Actions deployment [workflow](https://github.com/IATI/validator-services/blob/develop/.github/workflows/develop-func-deploy.yml) following the instructions for [Azure/functions-container-action](Azure/functions-container-action)  

  - Note that I've found you need to set `DOCKER_REGISTRY_SERVER_URL`, `DOCKER_REGISTRY_SERVER_PASSWORD`, `DOCKER_REGISTRY_SERVER_USERNAME` on your Function App and enable your Container registry Admin user if you're using an Azure Container Registry. [stackoverflow](https://stackoverflow.com/questions/60163440/docker-fails-to-pull-the-image-from-within-azure-app-service)

- Push some code to trigger the GitHub Action workflow

## Result

Our deployments for [IATI/validator-services](https://github.com/IATI/validator-services) went from over 20min to ~3-4min! A great improvement. Still not as fast as the original Consumption SKU Function, but good enough for our purposes.

![Deployment Time Comparison Screenshot](/assets/2021-09-10-faster-prem-functions-deploy/GitHubActions_Runtimes_screenshot.png)

## Side Note

As part of writing this article I came across another [GitHub Issue](https://github.com/projectkudu/kudu/issues/2087) that mentions using an "in-place" deployment could speed things up for the Kudu (non-containerised) deployment. I have not tried this option, but thought it worth mentioning here as it's likely easier to try first (setting `SCM_REPOSITORY_PATH` to `wwwroot` in your Function Configuration) before doing what I've done.

