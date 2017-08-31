---
title: How to set up a NodeJS website with Authentication and Back office (and Blog)
author: "Pedro Costa"
tags: ["NodeJS", "KeystoneJS", "Blog", "Authentication"]
date: 2017-09-01
---

In this tutorial we're going to build a NodeJS website with the following features:
- User authentication
- Back office
- Blog (optional)

<!--more-->

The first to parts of the tutorial are meant to quickly set you up and running. The third part is a more in depth explanation of some points I find important, so feel free to read them later as you need.


We'll be using [KeystoneJS](http://keystonejs.com) , which is a promising framework, based on [ExpressJS](http://expressjs.com/), that allows for a quick setup of the previous features.


**Prerequisites:**
- [Node.js 0.10+](https://nodejs.org/en/)


## Initial set-up


**Install the generator**
You'll be using the KeystoneJS generator made with Yeoman. In your terminal run:

```
$ npm install -g generator-keystone
```


**Create a folder for your project**

```
$ cd my-test-project
```


**Run the generator**

```
$ yo keystone
```
The generator will ask you a few questions about what features you'd like to include.


The templates question may cause some confusion:
*Would you like to use Pug, Nunjucks, Twig or Handlebars for templates? [pug | nunjucks | twig | hbs] ?  *If you're starting from scratch I recommend [Pug](https://pugjs.org/api/getting-started.html) it's simple and powerful, though if you're porting an existing project, [Handlebars](http://handlebarsjs.com/) (hbs) may be easier because you can use the existing html without any change.

I also strongly advice saying yes to extra comments in the code question.


Once finished your project will have an admin UI and all the dependencies installed.


So you just created your first KeystoneJS project, congrats!

To run your project you first have to connect to a database, so let's continue :)


## Getting started


**Project Structure**


At first the amount of folders and files may seem overwhelming so let's break it.


```
|--models
|  Your application's database models
|--public
|  Static files (css, js, images, etc.) that are publicly available
|--routes
|  |--views
|  |  Your application's view controllers
|  |--index.js
|  |  Initialises your application's routes and views
|  |--middleware.js
|  |  Custom middleware for your routes
|--templates
|  |--layouts
|  |  Base .pug layouts go in here
|  |--mixins
|  |  Common .pug mixins go in here
|  |--views
|  |  Your application's view templates
|--updates
|  Data population and migration scripts
|--package.json
|  Project configuration for npm
|--keystone.js
|  Main script that starts your application
```

Don't worry we'll explore more in depth some of these files later.


**Database set-up (the tricky part)**


The first thing you want to do is connect to a database. To make it easier I advise using a Database-as-a-Service product such as [mlab](https://mlab.com/).


After creating a free account create a new db by clicking on "+ Create new" then the Sandbox option (free), select Europe as the region and give it a name.


Now you have to setup the admin user of your newly created db. So click on the db and then on the tab Users and create a new user.


Next, copy the database URI which is at the top of the page (ex: mongodb://<dbuser>:<dbpassword>@ds157853.mlab.com:57835/yournewdb) to the bottom of the .env file at the root of your project, like so:
undefinedMONGO_URI=mongodb:<dbuser>:<dbpassword>@ds157853.mlab.com:57835/yournewdb
(The .env should never be uploaded to your github repository!).


Notice that I had the 'MONGO_URI=' to the beginning. Also don't forget to change <dbuser> and <dbpassword> to the user and password you've just created at mLab.


Finally let's see our project running, in your terminal type:

```
$ node keystone
```
Then open http://localhost:3000 to view it in your browser and voylá!


Now you may be wondering, that's nice but what just happened?


Well when launching for the first time the script in /updates/0.0.1-admin.js will create your first User. Which is an admin. So if you sign-in in your website with the credentials you set up in the generator you'll be able to manage all your website's database related data such as users and blog posts. Admins have access to the keystone back office while regular Users may only have access to certain parts of your website, that's up to you.
##
## ***Now you have a working web app and may feel free to explore the rest by yourself !***


There's more to it so in the next part I will explain a bit more some of the features of the framework.


## The next part (aka. Understanding KeystoneJS)


**Models**
So let's start with the models folder, here we define our database models, such as the users or the blog posts in this example.
In User.js define the user and add properties such as name, email, password and so on and set several properties. For example you can make the e-mail be unique and say which fields are required. You can get a more in depth understanding [here](http://keystonejs.com/docs/database/).


**keystone.js**
This is your app's starting point. And the one of the two places where you can customize the Admin UI (aka back office).


The init function you'll have something like:

```
keystone.init({
	'name': 'NIAEFEUP Website',
	'brand': 'NIAEFEUP Website',
	'less': 'public',
	'static': 'public',
	'favicon': 'public/favicon.ico',
	'views': 'templates/views',
	'view engine': 'pug',
	'auto update': true,
	'session': true,
	'auth': true,
	'user model': 'User',
});
```


If you want to change the keystone logo you have to append to the function the following line:

```
'signin logo': ['../../images/logo_aefeup.png', 230, 100],

```
I also usually add the following lines in order to redirect to the home page when signing out/in and override the '/keystone' in the url bar.

```
	'signin redirect' : '/',
	'signout redirect' : '/',
	'signin url' : '/signin',
	'signout url' : '/signout',
```


That's it about keystone.js everything else you'll find well explained in the code comments or in the documentation.


**Routes**
This is probably the most important part. Here you define the behaviour of your webapp.
The middleware.js is a starting point to some features in your app such as flash messages, enforcing user auth and so on. I'll focus on the initLoclas function. Here you set the nav links of your page and to which view they'll point.


But where the magic happens is in the index.js. Here you define the routes to your website, but what are the routes?
Simply the links in the url bar that point to the desired page.


You can pass non mandatory arguments like this:

```
app.get('/blog/:category?', routes.views.blog);
```
or mandatory arguments such:

```
app.get('/blog/post/:post', routes.views.post);
```
or even none:

```
app.get('/', routes.views.index);
```


You can also enforce the user to be authenticated by adding one more parameter ('middleware.requireUser'):

```
app.get('/profile', middleware.requireUser, routes.views.profile);
```


In the **views** folder is where you actually define the logic for each page and render the desired template.


**Templates**


First you have the 'layouts' folder, this is where you define the default layout of your website, the html header, body and footer.
So this is the general layout and inside the body there's only the header. The body content is defined in the views folder and will be displayed accordingly to url you've typed as defined in the routes.
The views folder contains the body content for each page in our website. You can also implement your own version of errors page such as 404 and 500 by adding these templates in the errors folder.


**Public Assets**
Keystone will serve any static assets you place in the public directory. For example your css files or your js scripts and libs (bootstrap, jquery, etc) This path is specified in keystone.js by the static option.
