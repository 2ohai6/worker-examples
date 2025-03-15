
# 103 Early Hints Worker

## Setting up a Cloudflare Worker for 103 Early Hints
This guide will walk you through setting up a Cloudflare Worker that allows a client to request static assets while waiting for the HTML response.


## What is Early hints?

103 Early Hints is an HTTP status code designed to speed up content delivery. When enabled, Cloudflare can cache the Link headers marked with preload and/or preconnect from HTML pages and serve them in a 103 Early Hints response before reaching the origin server. Browsers can use these hints to fetch linked assets while waiting for the origin’s final response, dramatically improving page load speeds 

## Get Started in the Dashboard

To ensure Early Hints are enabled on your zone:

- Log in to the Cloudflare Dashboard ↗ and select your account and website.
- Go to Speed > Optimization > Content Optimization.
- Enable the Early Hints toggle to on.


## Prerequisites

Ensure you have following:

- A Cloudflare account.
- Node.js and npm installed.
## 1. Create a New Worker Project

If you are setting up a new project, use the following command:

```bash
npm create cloudflare@latest
```

## 2. Develop with Wrangler Command Line Interface (CLI)

Navigate to your project directory and start developing with the following command:

```bash
npm run dev
```
This command will set up a local server. The worker will be available at http://127.0.0.1:8787.

## 3. Write Early Hints Code

Modify the src/index.js file in your project to contain the following code

```js
const CSS = "body { color: red; }";
const HTML = `
<!doctype html>
<html lang="en">
<head>
    <meta charset="utf-8">
    <title>Early Hints test</title>
    <link rel="stylesheet" href="/test.css">
</head>
<body>
    <h1>Early Hints test page</h1>
</body>
</html>
`;

export default {
  async fetch(req) {
    // If request is for test.css, serve the raw CSS
    if (/test\.css$/.test(req.url)) {
      return new Response(CSS, {
        headers: {
          "content-type": "text/css",
        },
      });
    } else {
      // Serve raw HTML using Early Hints for the CSS file
      return new Response(HTML, {
        headers: {
          "content-type": "text/html",
          link: "</test.css>; rel=preload; as=style",
        },
      });
    }
  },
};

```

In this code snippet:

We define a fetch function that is triggered when a request is received.

- We first check if the request URL is for the test.css file by using a regular expression (/test\.css$/). If the request is for test.css, we return the raw CSS code with a text/css content-type header.
- If the request is for any other resource (such as HTML), we return the raw HTML content and use an "Early Hints" header (link: </test.css>; rel=preload; as=style). This header tells the browser to preload the test.css file before rendering the page, potentially improving the page load time.

The overall flow serves HTML with an early hint to preload the CSS and serves the actual CSS file when requested.

## 4. Test Your Worker Locally
After starting wrangler dev, go to http://localhost:8787 in your browser. The page should load, and the browser should make an early request for the /test.css file based on the Link header.

- Open the browser's Developer Tools (F12 or right-click > Inspect) and go to the Network tab.
- You should see the preloading of the /test.css file (look for a preload request before the main HTML request).

## 5. Deploy Your Project

```bash
npm run deploy
```







## Conclusion

Great job on implementing Early Hints with your Cloudflare Worker! With this setup, you can now preload important resources like CSS files, improving your page load speed by ensuring the browser fetches necessary assets ahead of time. This results in a faster and more efficient user experience.