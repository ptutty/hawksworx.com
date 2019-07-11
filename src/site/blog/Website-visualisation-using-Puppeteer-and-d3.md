---
title: "Website visualisation using Puppeteer and d3.js"
description: Crawls a website and creates a visual overview of the site structure.
date: "2019-07-11"
allowcomments: true
tags: ["Puppeteer", "d3.js", "Node.js"]
pageClass: "blue-theme"
---

I created this app to get a handle on the size and complexity of a website we are redeveloping, it also allowed us to generate something visual and interactive to act as a reference for stakeholders. It could be adapted to:

- Create overviews of complex websites
- Generate reports and run diagnostics
- General Automation, scraping and testing

### [Demo and source code](https://xenodochial-lamport-efa047.netlify.com/)


![website visualiser](/images/web-site-visualiser.png)

<!--more-->

### Puppeteer and Chrome

For this project we used [Puppeteer](https://developers.google.com/web/tools/puppeteer/). Puppeteer is a Node library which provides a high-level API to control headless Chrome or Chromium over the DevTools Protocol. It can also be configured to use full (non-headless) Chrome or Chromium - this is especially useful in testing. In our project we use will use Puppeteer as web crawler, extracting data from webpages and following links to other webpages.

Puppeteer can also be used for testing, accessing Chrome Dev Tools and much more..

Here is a simple example which grabs a screenshot of a webpage. I'll be need is Node.js installed and to install puppeteer from NPM which comes bundled with Chrome) 

```js

puppeteer.launch().then( async browser => {
    const page = await browser.newPage();
    await page.goto('https:/example.com');
    await page.screenshot({path: "example.png'});
    await browser.close()
})

```


# Overview of the visualiser app

1. THe program takes as an input a json configuration file with the domain to start to crawl

```js
{
    "host": "https://www.bbc.co.uk",
    "path": "/sport",
    "depth": 2,
    "headless": true,
    "filters": false
}
```

Our script used the Puppeteer API and starting on the homepage to scrap all child hrefs on the page as well as the pages title and URL. The script then continues to crawl these child pages recursively. As we do this we create a data structure which stores these parent/child relationships.

As this is a asynchronous operation we need to wait for all promises to be resolved before we can write out a JSON file containing our final data structure.

This file is used by the d3.js library in the frontend to visually generated the nodes and edges.


## Filters

Filter allow you to remove unwanted cruff from the visualisation, such as: page anchors links, links back to the homepage, links to documents, intranet links etc. See the array 'excludeAnchorsWhichContain' below

Sometime you may wish not to crawl the navigation again on each subpage, you can list URL fragments in the array 'excludeSubpageAnchorsEndingWith'

```js
{ 
    "excludeSubpageAnchorsEndingWith" : [
        "/live/",
        "/programmes/",
    ],
    "excludeAnchorsWhichContain" : [
        "#",
        ".pdf",
        "docx",
        "doc"
    ]
}

```

## Start a crawl and capture data

To start a crawl, run the command below in the console - make sure you are in the project directory.

```bash
  node app.js
```
You will see URL's being crawled in the console.
You can also run a crawl and capture optional screenshots

```bash
  node app.js --screenshots
```

## View the visualisation

Start a local server.

```bash
  node server.js
```
Then open the URL below in a browser:

http://localhost:8080/html/d3tree.html?url=../output/https___yourspa.com/crawl.json





