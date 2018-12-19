# HEROKU TIME

![internet](https://media.giphy.com/media/SZUnyVdIDAEQU/giphy.gif)

* Heroku is a cloud application platform – a new way of building and deploying web apps.
* Heroku allows app developers spend their time on their application code, not managing servers, deployment, ongoing operations, or scaling. Heroku makes it easy!
* We can deploy full stack apps to Heroku for FREE!
* Heroku is also PSQL & Node friendly! Awesome!

### Before we begin, a word of warning:
Warning: Never git add, commit, or push sensitive information to a remote repository. Sensitive information can include, but is not limited to:

* Passwords
* SSH keys
* AWS access keys
* API keys
* Credit card numbers
* PIN numbers

***


## Part 1: Heroku Set Up

### Let's Get Started

##### These steps only need to be completed once.

Go to [heroku.com](http://www.heroku.com) and sign up for a free account. Remember your name and password - You will need it in a minute!!

###### Mac users 
1. Open a terminal window and `brew install heroku/brew/heroku`
2. `heroku autocomplete`
3. Once installed, you'll have access to the heroku app from the terminal by running `heroku login`. Enter the email address and password you used when creating your Heroku account.

###### windows users
got to [heroku-cli](https://devcenter.heroku.com/articles/heroku-cli) 

##### At this point you should have a personal Heroku account and the Heroku CLI toolbelt should be installed

---

## Part 2: Getting Production Ready

When we build on our computers, we've been working locally in a development environment and all our variables and configurations reflect this. We need to prepare our files so our app can switch to production mode and declare our dependencies before we launch.

1. Create the Procfile: Make sure you are inside the root of your App's directory.
    * From the command line run ```touch Procfile``` exactly like that, no extension, and a capital 'P'.
    * From your text editor go into your Procfile and type: ```web: node myentrypoint.js```.
      > HEY! YOU! The file name (here `myentrypoint.js`) should be the entry point for YOUR APP! (index.js, for example)

    * If no Procfile is present in the root directory of your app during the build process, your web process will be started by running the npm start script from your package.json.
      > Heroku says it is best practice to include a Procfile, so just do it!

2. We need to update our package.json:
    * add a start script where myentrypoint.js is the entry file for YOUR APP:
```javascript
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node myentrypoint.js" // this should be whatever YOUR entrypoint file is named
  },
```
* add your engines. First, from terminal, type ```node -v``` and copy just the numbers. Go back to your package.json and include engines:

```javascript
  "engines": {
    "node": "7.0.0" // put YOUR version of node here, in this format
  },
```
3. Make sure all the NPM packages you are using are in your `package.json`.

  * Example:
```javascript
  {
  "name": "heroku_node",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "engines": {
    "node": "6.2.1"
  },
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "node index.js"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "axios": "^0.16.2",
    "body-parser": "^1.17.2",
    "color": "^1.0.3",
    "express": "^4.15.3",
    "morgan": "^1.8.2",
    "mustache-express": "^1.2.4",
    "pg-promise": "^6.1.0"
  }
}
```
4. Okay, now we need to make sure both our database and our port are accessible both in Dev and Production mode. Go to your `myentrypoint.js` file (the entry file for YOUR APP)(again, probably `app.js` or `index.js`) and add these lines:
    * Replace your port variable with: ```const PORT = process.env.PORT || 3000; ```
        * This line tells our Node app to first check environment variables for a port number (for Heroku), otherwise default to port = 3000. Make sure your app is listening on this new PORT variable.
    * Replace your database variable with:
    ```javascript
    const db = pgp(process.env.DATABASE_URL || 'postgres://myname@localhost:5432/mydatabasename');
    ```
    or, if you're using a configuration object instead, such as
    ```javascript
    const cn = {
        host: 'localhost',
        port: 5432,
        database: 'mydatabasename',
        user: 'myname'
    };
    ```
    you can just say
    ```javascript
    const db = pgp(process.env.DATABASE_URL || cn); // cn is the configuration object
    ```
    *
        > HEY! YOU! make sure the postgres address and database name are YOUR APP's information

4. Finally, make sure any sensitive information, like API keys, are saved as variables and exported from your bash profile or a .env file (while cover dotenv soon, like in the next couple of minutes).

---


## Part 3: Create A Heroku App

1. In the terminal, type `heroku create`. The heroku create command creates a new application on Heroku – along with a git remote that must be used to receive your application source. Heroku will provide you a random name for your app. If you want to provide a custom name, type `heroku create mycustomname`. Make sure to use all lowercases and no spaces (but dashes are okay). It is totally possible to change the name of your app at a later date, so don't get too wrapped up in this step.
2. Now let's make sure our remotes are set, run `git remote -v`, you should see your heroku remote here.
3. Sweet, we have our app. Love it. Now we need to communicate with Heroku. Run a `git add`, `git commit`, then `git push heroku master`. Heroku will now rebuild your app on it's servers. If all goes well, when Heroku is done you'll see ```remote: Verifying deploy... done.```. If deployment fails, read the errors in your terminal.
4. Heroku gives us 2 easy command to check out our logs, let's run it now to see what happened:
    * See the last 100 lines of your log: ```heroku logs```
    * Real time continous logging: ```heroku logs --tail``` (This one is probably best utilized in it's own terminal window)
1. Run `heroku open`, this will open a browser page with your app. Don't be surprised to see *Application Error* - we still have stuff to do. Let's head to step 4!

---

## Part 4: Set Up Your PSQL Database#

Your database has been living a tidy but sheltered life on your local machine. It's time to give it wings so the world can interact with it by hosting it on Heroku.

Run ```heroku addons:create heroku-postgresql:hobby-dev```. This tells heroku to add a postgres database under the free hobby-dev plan.

---

## Part 5: Seed and Schema

Regardless if you used Heroku or terminal to set up you DB, we need to run our files from the command line:  

1. Run ```cat db/myseedfile.sql | heroku pg:psql```
    >HEY! YOU! make sure that you are running this command from the root of your directory, that your seed file is located in a db directory, and you insert the name of YOUR APP's seed/schema file!

2. You enter Heroku's database shell by typing the command ```heroku pg:psql```.
    * You can test a query here if you need to just like you do locally inside the psql shell.
3. You can use the handy ```heroku pg:info``` command to access information about your db on heroku.

## HEROKU DOCS

* [Heroku Postgres Docs](https://devcenter.heroku.com/categories/heroku-postgres)
* [Heroku CLI](https://devcenter.heroku.com/articles/creating-apps)
* [Heroku Node](https://devcenter.heroku.com/articles/getting-started-with-nodejs#introduction)
