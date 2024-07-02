---
title: "How to Build a Bot Detection Script From Scratch: A Step-by-Step Guide"
description: "Bot Detection"
image: "/images/posts/09.png"
date: 2024-06-12
draft: false
authors: ["Mark Dinn"]
tags: ["Bot", "Web"]
categories: ["Javascript"]
---

Did you know an estimated half of all web traffic comes from bots? And over half of that automated traffic is from "bad bots." Websites constantly face threats from automated bots designed to carry out malicious activities like credential stuffing attacks, web scraping, spam distribution, and fraud.

Accurately distinguishing this automated bot traffic from genuine human traffic is critical to protecting websites, user data, and online assets. As these bots continue to evolve, businesses must add robust measures to detect and mitigate bot attacks effectively and keep up with the latest bot technology.

Many methods exist to identify potential bot activity, whether you are analyzing network and traffic patterns, user browsers and device information, or behavior patterns. These methods allow you to take immediate action and prevent data breaches, protect user privacy, and ensure the integrity of your website.

This step-by-step guide for client-side bot detection explores common techniques for identifying bot activity. By the end of this tutorial, you'll have gained hands-on experience in building a basic bot detection script and learn practical methods to distinguish human visitors from malicious bots and protect your site.

Understanding bots
Bots are automated programs that can execute tasks and interactions over the internet by mimicking human behavior. These programs visit websites and apps and perform predefined actions, usually in a “headless” mode without a graphical interface.

On the one hand, bots facilitate many legitimate functions, such as web crawling for search engines, data collection for research, website performance monitoring, and automating repetitive online tasks.

However, bad actors can also use bots for malicious purposes, such as trying countless username/password combinations, illicitly scraping proprietary data or copyrighted content, launching distributed denial of service (DDoS) attacks, and spreading spam or malware.

Fundamentals of bot detection
Bot detection strategies can be broadly categorized into server-side and client-side techniques, each harnessing different methods to identify and manage bot traffic.

Server-side detection uses your backend to analyze data like IP addresses, HTTP headers, and session durations. Common methods include:

IP Blocklists: Checking user IP addresses against lists of known IPs or bot networks associated with malicious activity.
Rate Limiting: Monitoring the frequency of requests to identify and mitigate unusually high traffic from single sources, usually indicative of bots.
Behavioral Analysis: Assessing access and interactions over time to detect anomalies in behavior that deviate from human patterns.
Client-side detection operates within the user's browser and analyzes user behaviors and environmental data in real-time. Some client-side methods include:

CAPTCHAs: Challenges that differentiate bots from humans by asking visitors to do tasks that are trivial for humans but difficult for bots.
Browser Analysis: Gathering and analyzing browser configurations and capabilities to identify patterns or inconsistencies typical of automated scripts.
Behavioral Analysis: Monitoring actions like mouse movements, keystrokes, and interaction timings to detect non-human activity.
This guide will focus on client-side techniques, mainly how scripts can analyze the inadvertent data bots emit as they interact with a webpage. These leaks may manifest as anomalies in interaction patterns, browser settings, or how a page's layout is processed. By examining these factors, we can develop scripts that identify potential bots. This approach allows for rapid, on-the-spot bot detection, enhancing user experience by minimizing intrusive checks and maintaining performance efficiency.

Building a basic bot detection script
Let's start building a basic bot detection script for a sample application. This tutorial will use vanilla JavaScript to keep it accessible to a broad audience and various web environments. The objective of our application is simple: to analyze specific data from the visitor's browser to determine whether they are likely a bot. We'll do this by collecting data, analyzing them against common bot patterns, and sending a message in the application indicating if the script detected a bot. This example will showcase how client-side scripts can assess environmental signals to make this distinction and will not include server-side processing.

Prerequisites

To follow along with the tutorial locally, you must have Node.js installed on your machine and a code editor like Visual Studio Code. You can also use a cloud development environment like Stackblitz or CodeSandbox.

Set up the sample web application
To begin creating our bot detection script, we first need a simple web application where we will integrate it.

1. Create the project structure

Start by creating a new directory for your project. Inside this directory, create the following files:

index.html: This will be the main HTML document.
script.js: This JavaScript file will hold our bot detection logic.
Your project directory should look like this:

```javascript
bot-detection-app/
|-- index.html
|-- script.js
```

1. Set up the HTML file

Open the index.html file and set up the basic HTML structure, including linking to the script.js file. We’ll be using tailwind for some simple styling. Here’s a simple template to start with:


``` html
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <link
      rel="icon"
      href="data:image/svg+xml,<svg xmlns=%22http://www.w3.org/2000/svg%22 viewBox=%220 0 100 100%22><text y=%221em%22 font-size=%2280%22>🤖</text></svg>"
    />
    <title>Simple Bot Detection App</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="script.js" defer></script>
  </head>
  <body class="bg-gray-100 text-gray-900 min-h-screen pt-20 text-center">
    <h1 class="text-4xl font-bold mb-4">🤖</h1>
    <h1 class="text-2xl font-bold mb-4">Bot Detection Sample Application</h1>
    <div id="result" class="text-xl font-medium"></div>
  </body>
</html>
```

1. Serve and preview the application

You can use a simple HTTP server to run the sample web application locally and preview it in your browser. A simple way to do this is with http-server, an easy-to-use zero-configuration command-line HTTP server great for quickly serving static files. First, install the package:

npm install -g http-server
Then navigate to your project directory in the terminal and run:

http-server
Note: You can also install this as a local project dependency with npm install http-server and run it with npx http-server instead.

The HTTP server will serve your project, making it available at http://localhost:8080. Open this URL in your browser to view the application as you work.

With these steps completed, you have set up a simple web application that is ready to implement bot detection functionalities. This setup will allow us to focus now on collecting data and determining if a visitor might be a bot in the upcoming sections of the tutorial.

Collect and analyze visitor data
In this section, we'll focus on collecting data that we will use to determine if a visitor might be a bot. We will gather browser characteristics often exploited or modified by bots that serve as good indicators. Create a new detectBot function with a detectors object to store each detected signal.

```javascript
function detectBot() {
  const detectors = {};
}
```

1. Detect WebDriver automation

One of the simplest places to find signals for bot detection is to look at the Navigator object. This object is a part of the Window interface and represents the state and identity of the user's browser. It has information about the browser itself, including its version, the operating system it's running on, and various capabilities of the browser environment.

Within the navigator object, the webdriver property is especially useful as it indicates whether the browser is being controlled by automation tools such as Selenium, Puppeteer, or other automated testing frameworks. Unlike many other indicators that require interpretation or analysis, these tools typically set the navigator.webdriver property to true to indicate automation control. Add this property to the detectors object.

```javascript
function detectBot() {
  const detectors = {
    webDriver: navigator.webdriver, // Checks if the browser is controlled by automation
  };
}
```

As we progress, we will add more properties to the detectors object to collect additional data from different aspects of the visitor's environment that may indicate bot activity. Starting with navigator.webdriver establishes a solid foundation for recognizing the most obvious automated interactions.

1. Look for Headless Chrome

The navigator.userAgent property is another valuable piece of data for bot detection. This user agent string identifies the browser, its version, the underlying operating system, and sometimes other details about the device. Looking at the user agent can help determine whether the traffic originates from common headless browsers that might be used for scraping, automation, or other scripted activities. Add a check for the user agent that looks for “Headless”, a common value found in the most common tools used for automated tasks on the web. You can also check for other tools with string searches for values like “PhantomJS” or “Electron”.

```javascript
function detectBot() {
  const detectors = {
    webDriver: navigator.webdriver, // Checks if the browser is controlled by automation
    headlessBrowser: navigator.userAgent.includes("Headless"), // Detects headless browsers
  };
}
```

The webdriver and userAgent properties can cover quite a large amount of bot activity already, but the more indicators you analyze, the more accurate your bot detection can become. Especially since many automation tools add features and settings that try to evade common bot detection techniques. Let’s continue to add a few more helpful detector examples.

1. Check for missing preferred language

One potential indicator of a bot is the lack of set languages. The navigator.languages property returns a list of the user's preferred languages ordered by preference, with the most preferred language (first in the list) also set as the navigator.language property. However, the navigator.languages property sometimes returns an empty string in headless browsers. We’ll add the length of this property as a new signal for our bot detection.

```javascript
function detectBot() {
  const detectors = {
    webDriver: navigator.webdriver, // Checks if the browser is controlled by automation
    headlessBrowser: navigator.userAgent.includes("Headless"), // Detects headless browsers
    noLanguages: (navigator.languages?.length || 0) === 0, // Checks if no languages are set, uncommon for regular users
  };
}
```

1. Evaluate expected browser features

You can also check the properties of JavaScript functions that are sometimes altered by automation scripts or in a headless environment. Bots can easily change user agent strings, so this check looks to see if there is a mismatch between the browser from the user agent and the features that should be available for that browser. For example, you can check the length of the eval function and compare it to what is expected for that browser.

This check requires more work, first detecting the browser and then comparing the appropriate length values. Let’s add a new function to do this check and use the returned result for our inconsistentEval detector.

```javascript
function detectBot() {
  const detectors = {
    webDriver: navigator.webdriver, // Checks if the browser is controlled by automation
    headlessBrowser: navigator.userAgent.includes("Headless"), // Detects headless browsers
    noLanguages: (navigator.languages?.length || 0) === 0, // Checks if no languages are set, uncommon for regular users
    inconsistentEval: detectInconsistentEval(), // Check for inconsistent eval lengths
  };
}

function detectInconsistentEval() {
  let length = eval.toString().length;
  let userAgent = navigator.userAgent.toLowerCase();
  let browser;

  if (userAgent.indexOf("edg/") !== -1) {
    browser = "edge";
  } else if (
    userAgent.indexOf("trident") !== -1 ||
    userAgent.indexOf("msie") !== -1
  ) {
    browser = "internet_explorer";
  } else if (userAgent.indexOf("firefox") !== -1) {
    browser = "firefox";
  } else if (
    userAgent.indexOf("opera") !== -1 ||
    userAgent.indexOf("opr") !== -1
  ) {
    browser = "opera";
  } else if (userAgent.indexOf("chrome") !== -1) {
    browser = "chrome";
  } else if (userAgent.indexOf("safari") !== -1) {
    browser = "safari";
  } else {
    browser = "unknown";
  }

  if (browser === "unknown") return false;

  return (
    (length === 33 && !["chrome", "opera", "edge"].includes(browser)) ||
    (length === 37 && !["firefox", "safari"].includes(browser)) ||
    (length === 39 && !["internet_explorer"].includes(browser))
  );
}
```

1. Look for automation-related attributes

Some automation tools add specific attributes to the DOM. Looking for these attributes can be useful for detecting specific automation tools, like Selenium, which may leave traces in the form of custom attributes. Let’s add another data point to our bot detection that gets the attributes of the document's root element and looks for properties commonly associated with automation tools.

```javascript
function detectBot() {
  const detectors = {
    webDriver: navigator.webdriver, // Checks if the browser is controlled by automation
    headlessBrowser: navigator.userAgent.includes("Headless"), // Detects headless browsers
    noLanguages: (navigator.languages?.length || 0) === 0, // Checks if no languages are set, uncommon for regular users
    inconsistentEval: detectInconsistentEval(), // Check for inconsistent eval lengths
    domManipulation: document.documentElement
      .getAttributeNames()
      .some((attr) => ["selenium", "webdriver", "driver"].includes(attr)), // Looks for attributes commonly added by automation tools
  };
}
```

These are just a handful of example signals that you can use to identify bots, and the more sophisticated the bot, the more signals you will need to check to detect it. Using this data, let's look at how you can now detect if a visitor is a bot.

Detect the presence of a bot
After gathering the necessary data points about the visitor's environment, the next step is to analyze this information to determine whether the visitor is likely a bot. We can loop through each detector, and if a signal for a bot is found, we’ll record which detector was flagged and set our bot verdict to true.

```javascript
function detectBot() {
  const detectors = {
    …
  };

  // 1. Stores the detection results and the final verdict
  const detections = {};
  let verdict = { bot: false };

  // 2. Iterates over the detectors and sets the verdict to true if any of them detects bot-like activity
  for (const detectorName in detectors) {
    const detectorResult = detectors[detectorName];
    detections[detectorName] = { bot: detectorResult };
    if (detectorResult) {
      verdict = { bot: true }; // Sets the verdict to true at the first detection of bot-like activity
    }
  }

  // 3. Returns the detection results and the final verdict
  return { detections, verdict };
}
```

The new additions to the detectBot function:

Creates variables to store the bot detection results and a verdict.
Loops through each detector, and if a bot signal is found, it is added to the detections list and sets the verdict to true.
Returns the list of detections and the final bot verdict.
Use the bot detection result
At this point, you can decide how to handle the visitor based on your bot detection result. This tutorial will display the result on the page and log the detections and verdict in the console. Add the following after your function declarations.

```javascript
function detectBot() {
	...
}

function detectInconsistentEval() {
	...
}

const { detections, verdict } = detectBot();
document.getElementById('result').innerText = verdict.bot ? 'Bot detected' : 'No bot detected'; // Displays the detection result on the web page
console.log(JSON.stringify(verdict, null, 2)); // Logs the final verdict
console.log(JSON.stringify(detections, null, 2)); // Logs detailed detections
```

Test the bot detection
Now, we can visit our page and see the result of our bot detection script. If you followed along locally, ensure the http-server is running and visit http://localhost:8080 in your browser. If you are using a cloud IDE, visit the preview page provided for your project. You should see the result “No bot detected” displayed on the screen. You can also see the verdict and details in the developer console.

Screenshot of the final bot detection sample app

Now, let’s test the script using a bot. Within your root folder in a new terminal, we’ll add Puppeteer to test our basic bot detection.

npm install puppeteer
Next, create a new file called bot_test.js with the following code to run Puppeteer and test out your application.

```javascript
const puppeteer = require("puppeteer");

(async function testBot() {
  const url = "http://localhost:8080";
  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  // Log console messages from the page
  page.on("console", (message) =>
    console.log(`${message.type().toUpperCase()} ${message.text()}`)
  );
  await page.goto(url);
  await page.close();
  await browser.close();
})();
```

In this file, we use the Puppeteer library to launch a headless browser and navigate to your local application. You’ll notice there is a line to display any console messages from the page in your terminal to see the output of the verdict and detections. If you are following along in a cloud IDE, make sure to use the appropriate URL.

Now that the script is ready, in your new terminal at the root, run the bot_test.js script.

node bot_test.js
Optionally, you can use our instance of Browserless to visit your public-facing project link as a bot.

You will see the verdict and detections shown from the console log and should see an output similar to the one below that shows a bot was detected.

```javascript
LOG {
	"bot": true
}

LOG {
  "webDriver": {
    "bot": true
  },

  "headlessBrowser": {
    "bot": true
  },

  "noLanguages": {
    "bot": false
  },

  "inconsistentEval": {
    "bot": false
  },

  "domManipulation": {
    "bot": false
  }
}
```

This sample tutorial simply displays the result of the bot detection, but you can easily incorporate the verdict into your application to decide how to handle the visitor.

Improving your bot detection
While the basic bot detection script provided earlier serves as an introduction to identifying automated traffic, it naturally has limitations that could affect its use in more demanding or varied environments:

Limited Scope: The script checks only a handful of potential indicators, such as navigator.webdriver, user agent content for "Headless", and a few others. This narrow focus can miss more sophisticated bots that do not trigger these specific detectors.

Tool Specificity: Some of the checks, like looking for "Headless", are tailored to detect specific types of automation tools. While effective against those, they won't catch bots using different tools or customized solutions that don't modify the user agent string or use different mechanisms.

Browser Dependency: The detection techniques used can be highly dependent on the behaviors of specific browsers. Some indicators can have legitimate differences and variances with their implementations. Keeping up with these variations across updates and different browsers adds complexity and maintenance overhead.

Dynamic Web Ecosystem: Browsers and bot technologies evolve quickly. A method that works today might become obsolete tomorrow as both browsers update their features for privacy and security, and bot operators update their tools to evade detection.

Suggestions for script improving accuracy
To boost the accuracy and reliability of your bot detection mechanisms, consider extending your script beyond basic checks. By combining different types of data points and detection methods, you can create a more robust defense against various bot activities. For example, integrating behavioral analysis—such as monitoring mouse movements, keystrokes, and even scrolling behaviors—can help identify bots that might otherwise pass more static checks. These behavioral signals are often more difficult for bots to convincingly mimic and can reveal the non-human characteristics of their interactions.

Additionally, keeping your detection methods up-to-date is essential. As browsers evolve and bot operators refine their techniques, your detection strategies must also adapt. Regular updates to your scripts to align with the latest browser versions and known bot signatures ensure that your detection mechanisms remain effective against current and emerging threats.

Using the open-source BotD library
For those looking to improve their bot detection capabilities without the burden of developing and maintaining a complex in-house system, the open-source BotD bot detection library presents a robust and user-friendly alternative.

BotD provides more comprehensive coverage, incorporating more detection techniques that address multiple bot behaviors. Being an open-source library, it can easily be added to websites to increase defenses against bot traffic, minimizing the technical challenges typically associated with detecting bots.
