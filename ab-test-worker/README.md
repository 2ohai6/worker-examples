
# A/B test Worker

## Setting up a Cloudflare Worker for A/B testing with same-URL direct access

This guide will walk you through setting up a Cloudflare Worker that allows to set up an A/B test by controlling what response is served based on cookies. 


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
const NAME = "myExampleWorkersABTest";

export default {
  async fetch(req) {
    const url = new URL(req.url);

    // Enable Passthrough to allow direct access to control and test routes.
    if (url.pathname.startsWith("/control") || url.pathname.startsWith("/test"))
      return fetch(req);

    // Determine which group this requester is in.
    const cookie = req.headers.get("cookie");

    if (cookie && cookie.includes(`${NAME}=control`)) {
      url.pathname = "/control" + url.pathname;
    } else if (cookie && cookie.includes(`${NAME}=test`)) {
      url.pathname = "/test" + url.pathname;
    } else {
      // If there is no cookie, this is a new client. Choose a group and set the cookie.
      const group = Math.random() < 0.5 ? "test" : "control"; // 50/50 split
      if (group === "control") {
        url.pathname = "/control" + url.pathname;
      } else {
        url.pathname = "/test" + url.pathname;
      }
      // Reconstruct response to avoid immutability
      let res = await fetch(url);
      res = new Response(res.body, res);
      // Set cookie to enable persistent A/B sessions.
      res.headers.append("Set-Cookie", `${NAME}=${group}; path=/`);
      return res;
    }
    return fetch(url);
  },
};

```

In this code snippet:

We define a `fetch` function that is triggered when a request is received.

- First, we check if the request path starts with `/control` or `/test`. If it does, the request is passed through without modification, allowing direct access to the control and test routes for testing.
- Next, we check if the request contains a cookie indicating which A/B test group (control or test) the user belongs to by looking for the cookie named `myExampleWorkersABTest`.
- If the cookie is found, the path is modified to either `/control` or `/test` based on the value of the cookie, ensuring the user is served the correct version of the content.
- If no cookie is found (indicating a new user), we randomly assign the user to either the control or test group with a 50/50 chance, set the appropriate path (`/control` or `/test`), and create a response with a `Set-Cookie` header to store the user's group in future visits.
- Finally, the content is fetched from the updated URL and returned to the user, along with the set cookie for persistent A/B testing sessions.

The overall flow determines the A/B test group for the user (either `control` or `test`), modifies the request path accordingly, and ensures the user's group assignment is persistent using cookies.

## 4. Test Your Worker Locally
After starting `wrangler dev`, go to `http://localhost:8787` in your browser. The page should load, and the browser will display content from either the `/control` or `/test` path, depending on which group the user is assigned to in the A/B test.

- If it's a new user, the worker will randomly assign them to either the `control` or `test` group and return the corresponding content.
- If the user has already been assigned a group (i.e., a cookie exists), they will be served the content from the correct group.

- Open the browser's Developer Tools (F12 or right-click > Inspect) and go to the **Application** tab to check the cookies. You should see the `myExampleWorkersABTest` cookie with a value of either `control` or `test` indicating the group the user belongs to.

- You can also inspect the **Network** tab to see the request for the content and verify that the appropriate A/B test path (`/control` or `/test`) is being served.

## 5. Deploy Your Project

```bash
npm run deploy
```







## Conclusion

Great job implementing A/B testing with your Cloudflare Worker! This setup allows you to randomly assign users to different test groups and serve personalized content accordingly. By using cookies, you ensure that users remain in the same group across sessions, providing a seamless testing experience.