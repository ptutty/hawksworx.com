---
title: A walkthrough guide to using await and async for working with API's 
description: In this post I'll show you how to make multiple concurrent API requests and then process the results when all promises have been resolved.
date: 2019-06-10
allowcomments: true
tags: ["APIs", "JavaScript", "Node.js"]
pageClass: "teal-theme"
---

In this post I'll show you how to make multiple concurrent API requests and then process the results into one JSON file. We'll use this file JSON file as the data used for a typeahead searchbox. As these are asynchronous operations we'll be working with promises, however with the introduction of async and await methods in ES6 we no longer need all that boilerplate promise code and can write clear synchronous like code. 


![promises, promises](/images/async.png "async")


<!--more-->
# What we'll build

The final project we'll build will have both client and server components, the cleint will allow a user to make keyword searches, and the server will crawler muliple APIs and create a JSON file for are frontend to use. 

#### Source code for this tutorial
If you wish to follow along [download the source code for this project](https://github.com/ptutty/student-opportunity-autocomplete).

## The client
The client will be a simple webpage containing a search field. This field will have typeahead functionality and find possible keywords as we type. We'll use the fuse.js library to give us this functionality. This library requires a structured json file to use as a data source. This file is a large array of key and value pairs (see below) generated by our Node.js server-side code.

```json
[{
    "link": "https://warwick.ac.uk/services/careers/help/",
    "title": "careers appointments",
    "keyword": "careers appointments",
    "id": 1
}, {
    "link": "https://warwick.ac.uk/services/careers/help",
    "title": "Employability help & advice",
    "keyword": "careers advice",
    "id": 2
}]

```
More later on using fuse.js. For now let look at the server code and dig into async and await...


## The server code

### Why did we implement this solution?

Student Opportunity consists of eight teams and five different websites under different domains. We wanted users to be able to search for opportunities in all these website from the Student Opportunity website. Searching within a defined list of URL's was not possible using the Warwick Search API.

### The solution
We didn't want to have to maintain manually maintain a list of keywords & urls which would quickly become out of date. So we wrote a program which used the Warwick search API and:

1. Poll's Warwick search API using a URL prefix and keyword query(so we only get results for that team website).

2. Our code requests all the endpoints, processes the results and writes a out a json 
   

First we need to generate an array of urls to query. This again is done programmically based on a config file (see below). This json file contains a list of keywords and url prefixes to search under.


```json
[{
        "team": "careers",
        "url": "https://warwick.ac.uk/services/careers",
        "keywords": ["careers appointments", "careers advice", "CVs", "careers", "covering letters", "careers fair", "work experience", "jobs", "careers guidance", "job vacancies", "applications", "job interviews", "advisor", "careers advisor", "internships", "Career planning", "further study", "WSI", "warwick summer internships", "myAdvantage", "assessment centres", "psychometric and aptitude tests", "graduate outcomes", "workshops", "myadvantage", "employability", "Applying for jobs", "job search", "job application", "strengthes", "personality types", "careers advisor"]
    },
    {
        "team": "immigration",
        "url": "https://warwick.ac.uk/study/international/immigration",
        "keywords": ["visas", "Leave to Remain", "tier 1 visa", "tier 2 visa", "tier 4 visa", "Entry to the UK", "CAS", "Biometric Residence Card", "Permission to Enrol", "Police Registration", "Visa Processing Delays", "Status letter request", "WORKING ON A TIER 4 VISA", "Lost passports", "Visa Rejections", "Doctorate Extension Scheme", "IELTS"]
    },

```

We pass the searchItems data to our function which creates each url string and added it to an array.

```js

const searchItems = require('../seed-data/searchSeeds'); // warwick domains and 

apiEndpoints: function(searchItems) {
  let a = [];
  searchItems.forEach(function (team) {
      var apiUrl = searchConfig.apiBase + "urlPrefix=" + team.url + "&resultsPerPage=" + searchConfig.resultsNum;
      // remove any possible duplicate keywords
      let keywords = _.uniq(team.keywords);
      for (i = 0; i < keywords.length; i++) {
          a.push(apiUrl + "&q=" + keywords[i]);
      }
  });
  return a;
}

```

We end with an array of API endpoints to start fetching.

```js
["http://warwick.ac.uk/ajax/lvsch/query.json?urlPrefix=https%3A%2F%2Fwarwick.ac.uk%2Fservices%2Fskills&resultsPerPage=1&q=RSSP",
"http://warwick.ac.uk/ajax/lvsch/query.json?urlPrefix=https%3A%2F%2Fwarwick.ac.uk%2Fservices%2Fskills&resultsPerPage=1&q=CV"]
```

This array is passed onto the function in our program which does the fetching of data. We must write this function to deal with the concurrent, asynchronous fetching of 150 odd API endpoints.

### Using async and await

This is the code block for getting all the results from the 150 odd url calls. Once all the call have been resolved we can pass all the results to another fucntion to write out the data into one json file. However, we have to make sure ALL the concurrent request resolve before we can do this.

```js
async function main() {
    const urls = apiEndpoints(); 
    const allResults = urls.map(async (url,index) => { // A
       return await fetch(url) // B
            .then(res => res.json())
            .then(json => {
                return helpers.resultTidy(json.results[0], url, index); // C
            });
    });
 
    return await Promise.all(allResults); // D

}
```

What we need to do this is to create an array of Promises and then use the Promise.all method to wait for all of these promises to be resolved. Once this is done we can return the results as one large array of results.

Lets break this code block down a little.

A. 'allresults' will be a new array filled with promises, we'll use the map function to create this array. The JS Map method lets us run a function on every item in an array. In this we going to create a new array as we want to return the fetch function which is a promise as an array item.

B. using the await methods will setup these promises, please not that asynch has to be called from within an asynch function.

C. From with this fetch function we ruturn the results of the API fetch and do porcessing on the result before we do so,

D. We pass the array of promises to the Promise.all function prepending it with an await so that the result do not get returned until all the promises have been resolved.


We then call the main function and use the .then method (available on async functions) for code we need to run only once all async code has been resolved, in this case write out the results.

```js
main()
    .then(results => {
            helpers.writeFile(results);
    })
```

In part two of this walk-through i'll talk about other feature in the code - including throttling API calls and dealing with error handling.




















