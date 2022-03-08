---
layout: post
title:  "Implementing a Scalable Download Feature"
author: nik_osvalds
date:   2022-03-08
categories: blog
---

Today we released [Datastore Search](https://datastore.iatistandard.org). One of the main features of the product is to allow the user to download the search results in JSON, CSV, or original IATI XML format. This enables data users to export and use the IATI data in their own systems or analysis. Previous iterations of the IATI Datastore either required pagination over results to get a full view of the data or struggled to consistently handle requests for large amounts of data. This post covers how we tackled this issue on the IATI Unified Platform.

![IATI Datastore Search Downloader](/assets/download_modal.png)

## Overview 

This backend of this functionality was implemented in [Node.js](https://nodejs.org/en/) using cloud resources from Microsoft Azure. The same pattern could likely be reused for any data source HTTP API that can efficiently stream large amounts of data (in our case Apache Solr).

## Process Flow

1. A User performs a search on the [Datastore Search](https://datastore.iatistandard.org) site. The results of the search query are served from our [Apache Solr](https://solr.apache.org) instance and paginated to 10 results per page. This only contains a small subset of the data contained in each Activity.
1. The User clicks one of the download buttons to download their search results, picks the desired format, and pressed download.
![download buttons](/assets/download_buttons.png)
1. A POST request is sent to an [Azure Durable (Serverless) Function](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp), containing the format (XML, CSV, JSON) and Solr query string of the search.
1. This Durable Function is used to implement the [Async HTTP API](https://docs.microsoft.com/en-us/azure/azure-functions/durable/durable-functions-overview?tabs=csharp#async-http) pattern, where it passes the payload on to a Function to do the work, while returning a status endpoint to the front end page.
![Async HTTP API Pattern](/assets/async-http-api.png)
1. The Azure Function doing the work can then run for an extended period of time doing the download work, while the front end polls the status endpoint for the result and displays a nice spinny loader to the user.
![download spinny loader](/assets/downloading_in_action.png)
1. The backend [Download Function](https://github.com/IATI/datastore-services/blob/main/Download/index.js) takes the format and Solr query string and gets to work. 
1. The Download Function sends the query to our Solr instance, which returns as stream of the data with no pagination. That stream is then piped directly to an Azure Blob storage container using the [@azure/storage-blob](https://www.npmjs.com/package/@azure/storage-blob) npm package, and specifically the [BlockBlobClient.uploadStream()](https://docs.microsoft.com/en-gb/javascript/api/@azure/storage-blob/blockblobclient?view=azure-node-latest#@azure-storage-blob-blockblobclient-uploadstream) function.
1. The benefits of streaming the data are that we don't have to hold the large responses from Solr in memory. This allows us to use the [Consumption SKU](https://docs.microsoft.com/en-us/azure/azure-functions/consumption-plan) Functions which are very scalable, but only have 1.5 GB of memory available. 
1. Once the Download Function finishes downloading the data from Solr to the Azure Blob, it returns the publicly accessible URL of the Blob to the Durable Function Orchestrator, when then passes that along the the Status Endpoint that the Front End is polling.
1. The VueJS front end uses the blob URL to create an `<a>` element and programically click it [code](https://github.com/IATI/datastore-search/blob/8cd2782ac17ed05e72495d9e9295ad9c5d01c0b6/src/global.js#L512-L518), which triggers the browser's download functionality. 
![download prompt](/assets/download_prompt.png)

## Lessons Learned

- Azure Blob API version https://github.com/IATI/datastore-services/blob/2f7e7ad65989edab8c1f0a52b523969ff6123cf0/Download/index.js#L59-L60
- Required Headers to trigger browser download https://github.com/IATI/datastore-services/blob/c9c72fc168cbd7fd05e77d17a57d682c1773276b/Download/index.js#L74-L79

## Resources

- [IATI/datastore-search](https://github.com/IATI/datastore-search) - Front End VueJS Application
- [IATI/datastore-services](https://github.com/IATI/datastore-services) - Back End Node.js Azure Durable Function
