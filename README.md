## Deploying a fullstack MERN app to Vercel

In this guide we discuss the steps on how you can deploy a fullstack app, with your React frontend and your Express backend all in ONE repository, to Vercel.

The docs out there how to do that are all not super clear. The info is very cluttered and not concise.

So let's go through the process step-by-step.

### The repo setup

For a fullstack deploy you need to serve your react app over express. Respectively serve the "build" folder of React.

We need to create a folder for the frontend, e.g. "client" in our express app and create / move the react app into that.

That's it.

Now your fullstack folder structure, e.g. for an express app generated by express generator, will look something like that:

```
/bin 
  www
/client => react lives here!
  /build
  /src
  package.json
/public
/routes
app.js
package.json
```

### Vercel deployment config

Now we configure the deployment of your express app starting with Vercel.

We need to tell vercel to run our Express app with NodeJS, once uploaded.

So store the following JSON in a file "vercel.json" in your main folder.

```
{
  "version": 2,
  "builds": [
      { "use": "@vercel/node", "src": "app.js" }
  ],
  "routes": [
      { 
        "src": "/(.*)", 
        "dest": "app.js",
        "methods": ["GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS"]
      }
    ]
}
```

With that config we will whitelist ALL our routes and do not need to configure them one by one. That simplifies the setup a lot.


### Serving React over Express

Now that we got our Express app deployment setup, we need to prepare express to serve React over a "route".

We now setup a special route in express to serve the whole build folder. We can share a whole folder by using the express.static middleware.

Put this into your app.js file:

```
// serve react app build statically
app.use(express.static(path.join(__dirname, 'client', 'build'))) // please adapt the path to react if it is different for you!
```

It is important that you do that BEFORE you setup any other routes! So kind of like "Serve React first" approach (to prevent name conflicts of some build folder files with express routes that could have the same name).

#### Enable routes of React router

Now we finally need to address the React routes.

E.g. we have a React frontend route /profiles. But remember: We serve now React over express! Express will look now for a route "profiles". We do not have an Express route with that name. So Express would just say "route does not exist" and throw an error.

So we now - AFTER all other routes were declared - need to setup a "catch all" route which will forward every route that does not match an express route - to React! Respectively to the index.html file. That is the final trick to make React routing work!

```
// for all routes not handled by a backend route: 
// forward the request to the index.html (so let React handle it!) 
app.get('*', function(req, res, next) {
  res.sendFile(path.join(__dirname, "client", "build", "index.html")) // adapt the path to the build folder!
}); 
```

This will, e.g., catch a request like yourverceldomain.com/profile (if we do not have a GET route /profile in Express!) and will forward it to React. 

So React router can handle the routes where Express has not matching route. If the route does not exist, React will also handle the error.

#### Prevent route naming conflicts 

In Fullstack deploys, where both frontend and backend run on the same domain, we need to take care of route name conflicts between frontend & backend routes.

E.g. let's say we have a route /users.

We have a frontend route yourdomain.com/users which should display all users in an UI. 

But we also have a backend route /users which will be used to fetch users from the database. 

And now we got a conflict. Express can only serve ONE of these requests on the same route.

To prevent naming conflicts between your frontend routes & backend routes we need to make sure, our API routes are all unique and not used by react router.

A very simple convention to do that:

Give all backend routes a prefix "api".

So e.g.:
- yourdomain.com/users  will be now the FRONTEND route to display users
- yourdomain.com/api/users will be the BACKEND route to fetch users from

No route name conflict anymore!

Unfortunately there is not way how to set a global prefix for ALL your backend routes in your app. You can just set prefixes when you setup a single router.

So you can easily prefix your routes in your API when you import your routers in the app.js.

So instead of:
`app.use('/users', usersRouter);`

you do:
`app.use('/api/users', usersRouter)`

And that's it! No conflict between frontend and backend routes anymore!


### Preparing the automatic build of React

Finally we need to tell vercel to build react on every upload. Vercel will only look for a "build" script in the package.json of your main folder (so of your express app!). It will not look for it in the client folder.

So we just create a build script in the top level package.json and  will here just call the build script of react in the client folder:
'build': 'npm run build --prefix client'


### Environment Variables

Sensitive information used in the BACKEND, like connection details for the database, should never be commited publicly to your code repository.

Therefore you can either use the dotenv package to store your sensitive information outside of your code in an .env file that you add to your .gitignore file (if it is not already listed there).

The other - more flexible - way is to create a env.json file so you can store even nested configuration.

Or, even more flexible, use an ".env.js" file, where you can store your configuration as json and export it at the end. 

I like this approach most, because you do not need to install a separate package (like .dotenv), it gives you the flexibiliy to use all of JavaScript in your config, you can store javascript comments for each section (JSON does not allow comments) and you do not have the strict JSON file rules (=> the quoting hell :)). 

All in all: Go for an .env.js, it will be worth it.

Example .env.js file:
```
const conf = {
  mongodb: {
    user: '<yourRealUserName>',
    pw: '<yourRealPassword>'
    serverUrl: '<mongoatlasServerUrl>',
  },
  auth: {
    jwt-secret: 'holySecret2020',
    cookie-name: 'auth-token',
    cookie-lifetime: 1000*60*15, // milliseconds * seconds * minutes 
  }
  ...
}

module.exports = conf
```

Important! Add the .env.js file to your .gitignore!

But how do others know, who will checkout your code, what they need to put in that file?

There you often see a sample config with dummy (non sensitive) values. 

Example:

```
const conf = {
  mongodb: {
    user: 'username',
    pw: 'pw123'
    serverUrl: 'mongodb://someatlasurl.com/your_db_name',
  },
  auth: {
    jwt-secret: 'holySecret2020',
    cookie-name: 'your-token',
    cookie-lifetime: 1000*60*15, // = 15 minutes (milliseconds * seconds * minutes)
  }
  ...
}

module.exports = conf
```

THAT one you can safely add and commit to your code repository. So other have a sample and create their own .env.js file and put in there THEIR OWN sensitive information.

That's it! This is how you can create a secure and easy to share configuration for your app easily. 

### Summary - the minimal steps 

Finally a quick walkthrough how to test a vercel fullstack deploy with the least effort right now. All necessary code snippets you find above.

- Create a project with express generator
- Create a folder "client" in there and run npx create-react-app in the client folder
- Add a build script for building react in the package.json of your main express folder (so vercel can find that)
- Place the two route handlers above for serving react from express, in your app.js file
- Add the vercel.json above in your express main folder
- Run vercel
- Open the generated URL

That's it. This should do it. 

Happy project deployment!
