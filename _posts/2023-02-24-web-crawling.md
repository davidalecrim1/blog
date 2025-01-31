---
layout: post
title:  "How to Do Web Crawling and Web Scraping with Node.js and Puppeteer"
subtitle: ""
date: 2023-02-24 00:00:00 -0300
background: '/img/posts/nodejs-webcrawler.webp'
tags: [programming, nodejs]

---

At some point, everyone has needed to "Google something," search for information, and find a website to explore the content in more detail. However, one thing we rarely stop to think about is how the indexing of billions of websites is done and how search engines like Bing, Google, and Yandex present various results to us. But even more importantly, how does this system work?  

In this article, we will dive deeper into what’s "under the hood" of the bots that index content for search engines and understand how web crawling and web scraping are used for content indexing purposes.  

### Puppeteer  
Puppeteer is a browser automation tool available for Node.js and other languages. It provides an easy way to control a browser programmatically. With Puppeteer, you can create scripts that navigate websites, interact with page elements, collect information, and much more. Additionally, Puppeteer is compatible with most popular browsers, such as Chrome, Firefox, and Safari.  

With these capabilities, Puppeteer allows us to perform web crawling and web scraping on websites.  

### Web Crawling vs. Web Scraping  
Before we start, let’s clarify the difference between web crawling and web scraping.  

**Web crawling** is the process of systematically collecting data from various websites. It’s like scanning the internet for information. The goal of web crawling is to gather as much data as possible on a given topic, whether for research, market analysis, or other purposes.  

**Web scraping**, on the other hand, is the process of extracting specific information from a website. The goal of web scraping is to collect relevant data from a web page for a particular purpose, such as monitoring product prices, gathering contact information, extracting table data, or automating tasks that would otherwise be done manually by a user.  

Now that we understand the difference between web crawling and web scraping, let’s see how to use Puppeteer to perform these tasks.  

### Installing Puppeteer  
To get started, we need to install Puppeteer. You can install it using npm, the Node.js package manager.  

```bash
npm install puppeteer
```
Once installed, we can start using Puppeteer in our scripts.  

### Web Crawling Example  
Let’s begin with a web crawling example. In this example, we will use Puppeteer to collect all URLs on a webpage and then follow each URL to collect additional links from each visited page.  

To do this, create a file named `index.js` as follows:  

```javascript
const puppeteer = require('puppeteer');

(async () => {
  // Opens a browser session and a new tab
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  const urls = new Set();
  const queue = new Set();

  const crawlingDepthLimit = 5;

  queue.add('https://www.medium.com/');

  while (queue.size > 0) {
    const url = queue.values().next().value;

    // Removes the next URL from the queue and adds it to the results
    queue.delete(url);
    urls.add(url);

    // Stops adding new URLs if the limit is reached
    if (urls.size < crawlingDepthLimit) {
      // Navigates to the current URL
      await page.goto(url);

      // Extracts links and adds them to the queue
      const newUrls = await page.$$eval('a', links =>
        links.map(link => link.href)
      );

      // Avoids duplicate URLs before adding to the queue
      newUrls.forEach(newUrl => {
        if (!urls.has(newUrl) && !queue.has(newUrl)) {
          queue.add(newUrl);
        }
      });
    }
  }

  // Closes the browser
  await browser.close();

  console.log([...urls]);
})();
```

This script opens the [Medium](https://www.medium.com) website in a Puppeteer-controlled browser instance and collects all URLs from the page. It then adds each URL to a collected list and queues all newly discovered URLs for further processing.  

The script then follows each queued URL and repeats the process, collecting URLs from each linked page and adding them to the queue. As a best practice, a crawling limit is set to prevent excessive runtime.  

Running this script with Node.js using `node index.js` produces the following result:  

![](../../../img/posts/puppeteer-result-01.png)  

Now, let’s look at a web scraping example. The following script opens the homepage of [example.com](https://www.example.com/), extracts the page title (`<h1>`) and the first paragraph (`<p>`), and displays them in the terminal.  

```javascript
const puppeteer = require('puppeteer');

(async () => {
  // Opens a browser session and a new tab
  const browser = await puppeteer.launch();
  const page = await browser.newPage();

  // Navigates to the target page
  await page.goto('https://www.example.com/');

  // Extracts HTML elements from the page
  const title = await page.$eval('h1', el => el.innerText);
  const paragraph = await page.$eval('p', el => el.innerText);

  // Takes a screenshot of the website
  page.screenshot({path: `screenshot.png`});

  console.log(title);
  console.log(paragraph);

  // Closes the browser
  await browser.close();
})();
```

Running this script with Node.js using `node index.js` produces the following result:  

![](../../../img/posts/puppeteer-result-02.png)  

One of Puppeteer’s features is the ability to take screenshots of the browser session, as shown in the example below using [example.com](https://www.example.com):  

![](../../../img/posts/puppeteer-result-03.png)  

These are just simple examples, but the possibilities are limitless. You can extract a vast amount of useful information from websites, such as product prices, contact details, and table data. You can also automate repetitive tasks or conduct UI testing by simulating user interactions, such as clicking buttons and completing full user experiences.  

### Conclusion  
Puppeteer is a powerful tool for web crawling and web scraping. It allows us to automate browser actions, interact with webpage elements, and extract specific information. With Puppeteer, we can easily collect data from various sources, such as news sites, social media platforms, and e-commerce websites.  

However, it's crucial to perform web scraping ethically and legally. Some websites have terms of service that prohibit data scraping or consider it a violation of their copyright. Additionally, many websites use anti-bot mechanisms, such as Google’s reCAPTCHA, which detect and block automated scripts like the ones we created above.  

Understanding web crawling and web scraping concepts helps us see how search engines extract and index information from the web. When a specific term, phrase, or question is searched (e.g., "What is Node.js?"), the search engine retrieves results from its latest indexed version of public websites, which were indexed by bots similar to the ones we built.  

The image below illustrates how search engines aggregate information from multiple websites and use indexed data (i.e., storing and processing raw HTML, as we did above) to provide a customized experience for users.  

![](../../../img/posts/exemplo-google-01.png)  