
# Cloudflare Object Worker

## Setting up a Cloudflare Worker for Accessing the Cloudflare Object

This guide will walk you through setting up a Cloudflare Worker that allows to access custom Cloudflare properties and control how Cloudflare features are applied to every request.


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
  async fetch(req) {
    const data =
      req.cf !== undefined
        ? req.cf
        : { error: "The `cf` object is not available inside the preview." };

    return new Response(JSON.stringify(data, null, 2), {
      headers: {
        "content-type": "application/json;charset=UTF-8",
      },
    });
  },
};

```

In this code snippet:

We define a fetch function that is triggered when a request is received.

- First, we check if the cf object is available in the request (req.cf). The cf object contains Cloudflare-specific data about the request, such as geographic information, security settings, and more.
- If the cf object is available, we store it in the data variable. If it's not available (e.g., in preview environments where Cloudflare data may not be accessible), we assign an error message to data.
- We then return a Response containing the data in JSON format, which is stringified and formatted for better readability (JSON.stringify(data, null, 2)).
- The response is set to have a content-type of application/json;charset=UTF-8, indicating that the response body is JSON data.

The overall flow checks if Cloudflare-specific data is available, then returns it as a formatted JSON response. If that data is unavailable, it returns an error message instead.

## 4. Test Your Worker Locally
After starting wrangler dev, go to http://localhost:8787 in your browser. The page should load, and the browser will display a JSON response containing Cloudflare-specific data, or an error message if the cf object is unavailable.

## 5. Deploy Your Project

```bash
npm run deploy
```

## Conclusion

Great job implementing Cloudflare-specific data handling in your Worker! This setup allows you to access useful request metadata via the req.cf object, enhancing content personalization or performance. 