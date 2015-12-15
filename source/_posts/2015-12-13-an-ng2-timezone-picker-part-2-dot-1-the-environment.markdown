---
layout: post
title: "An Ng2 Timezone Picker - Part 2.1: The Environment"
date: 2015-12-13 23:25:18 -0700
comments: true
categories: [angular2, timezonepicker]
---


First things first, we are going to get our Angular2 project environment setup.  Feel free to use your favorite IDE or text editor for this series.  I'll be using Webstorm because I'm a fake developer that doesn't understand VIM key bindings.

The starting point for our our timezone picker is going to be the [angular2-seed](https://github.com/angular/angular2-seed) project found in the official angular repository.  I'm a big fan of reusing what's there.  Plus, this saves us from rewriting a bunch of boilerplate.


#### Getting the seed setup

Once you have node.js installed, just clone the [angular2-seed](https://github.com/angular/angular2-seed) repository and run `npm install` to get setup.  If needed, use the steps below.

``` bash
#go to your source dir
cd /my_cool_source_dir

#clone the seed project
git clone https://github.com/angular/angular2-seed ng2timezonepicker

#go into the seed directory
cd ng2timezonepicker

#install dependencies
npm install
```

The angular2-seed project provides us with a good starting point for our timezonepicker.  It comes preconfigured with the following technologies:

1. [Angular2](https://angular.io/) for building awesome web stuff
2. [Webpack](https://webpack.github.io/) for module bundling
3. [TypeScript compiler](http://www.typescriptlang.org/) for typed Javascript (and more)
4. Associated Node packages

If you aren't familiar with Webpack or Typescript, don't fret too much.  A deep knowledge of Webpack will not be needed for this series.  However, it's becoming quite prevelant in the JS community, so you should probably dedicate some time to it in the future.

If you've never used TypeScript before, you're not alone.  We won't use most of the language features in this series.  And the ones we do, I'll try and explain.  However, if you see something that looks unfamiliar, go ahead and head over to the [TypeScript](http://www.typescriptlang.org/) site to see what's up.

#### Running the project

The `package.json` file also has the `npm start` command configured to run the [webpack-dev-server](https://webpack.github.io/docs/webpack-dev-server.html) on port 8080.  So once you've got everything setup, run `npm start` and open your browser to localhost:8080.  We're ready to get coding!



