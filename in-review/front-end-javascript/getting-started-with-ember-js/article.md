# Installation and initial steps

## Installing node.js

Node.js is a JavaScript runtime built on Chrome's V8 JavaScript engine for developing server-side applications. It also has a built in package ecosystem called **node package manager** (npm), which allows you to install a variety of packages and plugins (such as Ember CLI) to increase your development efficiency.

### nvm

While node has a great installer, I highly recommend installing node via **node version manager** (nvm), which allows you to install and run multiple versions of node alongside one another. The nvm serves as a simple interface that manages the different versions of the node on our system and allows us to easily switch between versions when we jump between different projects.

#### OS X Installation

**To install nvm on OS X**, we can use the fantastic (Homebrew package manager)[http://brew.sh/]. Once you have it installed, simply run `brew install nvm`. Once it's been installed we have a few housekeeping items to address to make sure that it's properly configured. First, create a working directory for the tool by calling `mkdir ~/.nvm`. This tells our system to "make a directory" (`mkdir`) in our "home" folder (`~`), called `.nvm`. The file is preceeded by a `.` to make it a "hidden" folder, ensuring that it won't show up in your typical file-system browsing activities (because you shouldn't typically be directly modifying its contents).

Next, we need to add some environment configuration to our shell's configuration (`.zshrc` or `.bashrc`). At the time of this tutorial's creation, nvm requires the following configuration:
```
export NVM_DIR=~/.nvm
. $(brew --prefix nvm)/nvm.sh
```
Finally, load the configuration by "sourcing" your `rc` file - (`source ~/.zshrc` or `source ~/.bashrc`). You can run `nvm help` to ensure everything is working at this point.

## Installing Ember CLI

With node.js and nvm successfully setup, simply run the command `npm install -g ember-cli`. Before we move on, though, let's look at what this command does. `npm` invokes the node package manager executable. This command involves a variety of subcommands such as:

- `install` - install a package
- `help` - lookup help on subcommands or the npm command itself
- `update` - update a package
- `ls` - list installed packages

We invoke the `install` subcommand with a "flag" (a letter preceded by a dash such as `-g` or `-l`). The `-g` flag tells npm to install this package globally. The package will now be accessible to anything that has access to your node modules, instead of just the particular project you are working on. Finally `ember-cli` is the name of the package we install. When installed it exposes an `ember` executable for managing Ember CLI-based projects.

## Generate a project

Generating a project in Ember CLI is simple. First, make sure you're in the directory where you'd like to store the project (for example, `~/Projects` or `~/dev`). Once you're in an appropriate directory, we'll use the `ember new` command to start a new project by passing it a project name. We'll be developing a simple to-do list application, so we can run `ember new todos` to create our project folder. After the command completes, run `cd todos` (or assuming you named it `todos`) to move to your new project directory.

## Viewing the project

Ember CLI gives us a fully functional ember application straight "out of the box." Let's jump right in by running `ember serve` in our new projects directory. 

The project should be viewable at http://localhost:4200. While our project doesn't look like much at this point, we've actually got a full webserver running with live-reloading capabilities and more with very little effort. Furthermore, the Ember CLI tool has a full compliment of generators,  a solid "add-on" ecosystem, and a production-ready build tool to assist in deploying our apps.

## The Ember Inspector

Now that we've got our "app" up and running, it's time to take a quick detour and look at another powerful Ember development tool - **[the Ember Inspector]**(https://chrome.google.com/webstore/detail/ember-inspector/bmdblncegkenkacieihfhpjfppoconhi?hl=en). The Ember Inspector gives you a full suite of debugging and exploration capabilities right in the browser.

# The application route and index route

One of Ember's core differentiators is it's focus on routing as a first-class citizen. Unlike other JavaScript frameworks that view routing as an afterthought or something to be handled by a plugin, _Ember has full fledged support for dynamic routing all handled from the client-side_. As a result, Ember compliments the features of modern browsers rather than compete against them.

Out of the box, every Ember application has a root-level route known as the "application" route. Let's take a look at what this means, why it's necessary, and how it affects the rest of our application.

## Customizing the application route

The application route is basically a "wrapper" for all other routes that get rendered in your app.

### `app/routes/application.js`

### `app/templates/application.hbs`

## Customing the index route

Each "parent" route in an app (including the application route) will, by default, render an index route when it is entered.

### `app/routes/index.js`

### `app/templates/index.js`

# Our first custom route

Now that we have a decent understanding of the routes that Ember builds by default, let's take a crack at building our own route.

## `app/router.js`

## `routes/about.js`

## `templates/about.hbs`

# Pods

# Components

# Addons
