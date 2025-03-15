
# Bulk redirects Worker

## Setting up a Cloudflare Worker for Bulk redirects

This guide will walk you through setting up a Cloudflare Worker that allows to redirect requests to certain URLs based on a mapped object to the request's URL. 


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

## 3. Write Code

Modify the src/index.js file in your project to contain the following code

```js
export default {
  async fetch(request) {
    const externalHostname = "examples.cloudflareworkers.com";

    const redirectMap = new Map([
      ["/bulk1", "https://" + externalHostname + "/redirect2"],
      ["/bulk2", "https://" + externalHostname + "/redirect3"],
      ["/bulk3", "https://" + externalHostname + "/redirect4"],
      ["/bulk4", "https://google.com"],
    ]);

    const requestURL = new URL(request.url);
    const path = requestURL.pathname;
    const location = redirectMap.get(path);

    if (location) {
      return Response.redirect(location, 301);
    }
    // If request not in map, return the original request
    return fetch(request);
  },
};

```

In this code snippet:

We define a `fetch` function that is triggered when a request is received.

- First, we create a `Map` called `redirectMap` which maps specific request paths (e.g., `/bulk1`, `/bulk2`) to corresponding external URLs (e.g., `https://examples.cloudflareworkers.com/redirect2`, `https://google.com`).
- We then extract the pathname from the request URL to identify the path being requested.
- Using the pathname, we look up the appropriate redirection URL in the `redirectMap`. If a match is found, we return a 301 permanent redirect to the location.
- If no matching redirection path is found in the map, the original request is fetched as usual.

The overall flow is to redirect specific paths to predefined external URLs, and if the request path is not in the map, it simply returns the original content.

## 4. Test Your Worker Locally

After starting wrangler dev, go to `http://localhost:8787` in your browser. The page should load, and if you visit any of the predefined paths (e.g., `/bulk1`, `/bulk2`, `/bulk3`, `/bulk4`), you will be redirected to the corresponding external URL defined in the redirectMap.

## 5. Deploy Your Project

```bash
npm run deploy
```

## Conclusion

Great job implementing URL redirection with your Cloudflare Worker! This setup allows you to easily redirect specific paths to external URLs using a simple mapping. If no match is found, the original request is served as usual, ensuring smooth handling for all other requests.