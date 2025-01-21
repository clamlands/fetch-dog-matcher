# Fetch Dog Matcher

##

This is the Fetch Frontend Take-Home Exercise. On this website app, users can search from a database of shelter dogs and filter and sort their results. The user can "favorite" dogs of their choice, and finally submit their favorites list to receive a dog match for them.

## Developing

After cloning to a local repository and installing dependencies with `npm install` (or `pnpm install` or `yarn`), you can start a development server:

```bash
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of the app:

```bash
npm run build
```

You can preview the production build with `npm run preview`.

## Known bugs

It appears that some browers, particularly Safari and even Chrome installed on Apple products, disable the use of the third-party cookies required for authentication on this app. You will be able to login, but the remainder of the app will be unfunctional. It is my understanding that since this issue appears to be contingent on the user's browser settings, there is nothing further that can be done on the client side to ensure crendentials are sent with fetch requests.
