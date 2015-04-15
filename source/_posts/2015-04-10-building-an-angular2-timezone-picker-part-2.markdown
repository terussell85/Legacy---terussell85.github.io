---
layout: post
title: "An Angular2 Timezone Picker - Part 2: Exploring the World (of ng2)"
date: 2015-04-10 23:22:28 -0600
comments: true
categories: [javascript,angular2]
---

### TLDR

### Recap

In part 1 ...

### Getting Started

Alright!  Let's get started with part 2 of our timezone picker.  Now that we've built a good map to serve as our starting point, the Angular2 code that goes along with it will actually be relatively simple.  We'll use the [Angular2 Yeoman generator](https://github.com/swirlycheetah/generator-angular2) as a starting point.  Again, I'm not sure if this is the best way of doing things, but I personally liked it better than the "Getting Started" on [angular.io](http://www.angular.io).

If you've made it this far into the series, congratulations, you're finally going to see some JavaScript.  Well... kind of.  We're going to be using TypeScript with ES6 structure and modules.  Love it or hate it, it's going to be the way to write Angular2, so we'll try and pick it up now and stay ahead of the game.

Once the generator ran, I made a few small tweaks to the file organization, just for my own sake.  In the end, my file structure looked like the image below.


index.html
```html
<!doctype html>
<html lang="">
  <head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge">
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <title>My Timezone Picker</title>
  </head>
  <body>

    <my-app>Loading...</my-app>

    <script src="lib/system.js"></script>
    <script src="lib/zone.js"></script>

    <script>
      System.config({
        baseURL: 'lib/',
        paths: {
          'angular2/*': 'angular2.js',
          'rx/dist/rx.all': 'rx.all.js',
          'components/*' : "../components/*.js"
        }
      });

      System.import('components/app/main');
    </script>

  </body>
</html>
```

/src/components/app/main.js
```javascript
import {Component, Decorator, Template, bootstrap} from 'angular2/angular2';
import {TimeZonePicker} from 'components/timezonepicker/timezonepicker'


// Annotation section
@Component({
    selector: 'my-app'
})
@Template({
    inline: 'Testing'
})
// Component controller
export class Main {
    constructor() { }
}

bootstrap(Main);
```

### A Quick Note on Angular2

Before we get too far into the weeds, I want to get something out in open...  I'm still an Angular2 n00b.  *Whew*.  I'm glad we got that out of the way.  This article reflects a play-by-play of my own personal adventures as I began learning Angular2 (and continue to learn it).  So if you're still learning Angular2, then we'll be learning together.  If you're an expert, feel free to comment and let me know how to improve.  Feedback is great.

Keep in mind that this means many things that are covered here might be likely to change in the future.  As to be expected of a framework in alpha stages, Angular2 is still the wild wild west of coding. Expect to read source code, design documents, and pull requests.  And expect that things are likely to change in the future.  In fact, since beginning this post, a [few](https://github.com/angular/angular/issues/1244) [open](https://github.com/angular/angular/issues/973) [issues](https://github.com/angular/angular/issues/1239) have already changed some fundamental parts ofAngular2.  Fun.  I will try and update once the changes have been made public in the alpha releases.

Even with all those caveats, I hope you will find that you learn something from my misadventures.  I will attempt to explain some of the concepts that I have learned thus far.  If you feel like my explanation are lacking, and you need more information, [this curated list](https://github.com/timjacobi/angular2-education) is probably a good starting point.


### The Architecture

Because of it's complexity, we already know that our time zone picker is going to consist of a number of different Angular2 components working together.

There is one major constraint that we need to keep in mind as we plan our architecture so that we don't code ourselves into a corner. We need to remember that *our map is generated*.  This turns out to be important.  We want to be able to use our map in it's generated form without modifying the SVG.  This will allow us to regenerate maps with different bounds, timezones, etc without requiring tweaks by hand, or writing some sort of post-processor.

In more specific terms, it means that our Angular2 components need to work with the existing SVG structure of the map.  In traditional directive development, we would be defining the templates.  With our picker, that is almost reversed.  The template is defined, and we are writing directives to assign behavior.  You could do the same thing with AngularJs (v1.x), but it's actually really impressive how easily Angular2 still works with these constraints.

Alright, so with that constraint in mind, we know that we will need at least 3 directives:

1. The Time Zone Picker - this will serve as the container component.  It will provide some behavior, but mostly styling and a wrapper template
2. The Time Zone Map - this is the component that will represent our map.  It will coordinate the state across all time zones.
3. The Time Zone - this component will serve to assign click behaviors to a time zone in our map.

Let's start by creating stubs for each of these directives.

/src/components/timezonepicker.js
```javascript
import {Component, Template} from 'angular2/angular2';
import {TimeZoneMap} from 'components/timezonepicker/timezonemap';

@Component({
    selector: "timezone-picker"
})
@Template({
    inline: "<timezone-map></timezone-map>",
    directives: [TimeZoneMap]
})
export class TimeZonePicker{
    constructor(){
        console.log("TimeZonePicker constructed");
    }
}
```

/src/components/timezonemap.js
```javascript
import {Component, Template} from 'angular2/angular2';
import {TimeZone} from 'components/timezonepicker/timezone';

@Component({
    selector: "timezone-map"
})
@Template({
    url: "components/timezonepicker/timezonemap.svg",
    directives: [TimeZone]
})
export class TimeZoneMap{
    constructor(){
       console.log("TimeZoneMap constructed");
    }
}
```

/src/components/timezone.js
```javascript
import {Decorator} from 'angular2/angular2';
import {PropertySetter} from 'angular2/src/core/annotations/di';

@Decorator({
    selector: '[data-tz-id]'
})
export class TimeZone{
    constructor(){
        console.log("TimeZone constructed");
    }
}
```

### Pause!

If you're new to Angular2, those stubs already contain an awful amount of new information.  Here is a crash course:

* *"import {item} from 'location'* - This is ES6 module syntax for loading dependencies.  Pretty sweet huh?  Do a Google search for "ES6 modules" if you want more information.  Lot's has been written already, I won't duplicate.
* *@Component, @Template, @Decorator* - These fancy things are called "annotations".  They've been a pivotal part of other languages for a while, and a few JavaScript libraries as well.  Angular2 will use then in full force.  Think of them as neat ways to establish metadata.
* *@Component vs @Decorator* - There are [three types of directives in Angular2](https://github.com/angular/angular/blob/master/modules/angular2/docs/core/02_directives.md), Decorator, Components, and Viewports.  In short, decorators are useful for encapsulating behavior, components are useful for encapsulating behavior and visual state.
* *constructor()* - Constructor functions from ES6 classes.  It is calls anytime an instance of this class is created.  Again, Google this one, there is a plethora of information already.

Most of the other items should be pretty self-explanatory.  The properties of the annotations provide configuration for Angular2 to build and find the components we are creating.

You'll also notice that we use the selector "[data-tz-id]" in our TimeZone @Decorator annotation.  When we generated our map, we designated that the timezone id should be added to each time zone SVG path with that attribute.  That makes it really easy to create a directive instance for each time zone.

### Back to the show

Now that we have our stubs, let's make sure it all works.  In `/src/components/app/main.js` let's change our app component to include a TimeZonePicker in its template.
```javascript
//change the template from this
@Template({
    inline: 'Testing'
})

//to this
@Template({
    inline: '<timezone-picker></timezone-picker>'
})
```

Now let's fire up our component using `gulp dev`.

<div style="text-align:center; margin-bottom:20px;">
    <img src="/images/timezone/timezonestub_output.jpg" style="max-height: 250px">
</div>

We see that each of the components is being instantiated!  Perfect.  Let's start fleshing them out.

### Make the Time Zones Clickable

Let's start to add some behavior.  We know that we want the user to be able to click on a time zone and have it be selected.  So let's start there.  In Angular 1.x, we'd add an ng-click with a callback.  In standard Angular2, we'd probably add something like `(click)="doSomething()"`.  But remember how we don't want to change the SVG to add properties?  So how in the world could we assign click behavior to the time zone?

Maybe we could inject the current element and add a click listener by hand, that calls a function in the TimeZone.  That is actually very possible in Angular2, but more work than it's worth.  Luckily, we are provided a shortcut.

If we look at the @Decorator source, you'll notice a property called `events`.

<strong>--Author Injection--</strong> Ugh.  This one literally [changed underneath me](https://github.com/angular/angular/issues/1244) as I was writing this article.  It looks like it is now `hostListeners`.  Expect updates in the future to fix the syntax.

The `events` property allows up to wire up listeners on our host DOM element that are automatically wired to expressions in our directives.  So let's tweak our TimeZone implementation to include the click event and corresponding callback.

```javascript
import {Decorator} from 'angular2/angular2';
import {PropertySetter} from 'angular2/src/core/annotations/di';

@Decorator({
    selector: '[data-tz-id]',
    events: { click: 'onTimeZoneClick()' }
})
export class TimeZone{
    constructor(){
        console.log("TimeZone constructed");
    }

    onTimeZoneClick(){
        console.log("time zone was clicked!");
    }
}
```

Alright, now let's fire up our picker and take a look.  Hmm.  Well, that's weird, why isn't it working?  Ah yes, SVG paths with no fill are not clickable.  We don't have a proper place to style yet, so let's just make a quick tweak to `index.html` to add a path fill.  We'll do proper styling later.

```html
    <style type="text/css">
      path[data-tz-id]{
        fill: transparent;
      }
    </style>
```

Now we're in business.  Our time zone is responding to clicks.  But we still don't know which id corresponds to the time zone shape being clicked.  That information is available in the `data-tz-id` attribute on the DOM element, but not in our Decorator.  Let's head to the documentation (TODO: LINK) one more time.  It looks like directives can specify a `bind` property to get attributes from the DOM elements.  Let's try that.

On a side note, because of the way that Angular2 bindings work, the `bind` property is actually the same for both static property values and dynamic values from parent directives.  So no more `@` and `=` in directive definitions.  Yay!

<strong>--Author Injection--</strong> It looks like `bind` has also been changed. It's new name is `properties` and has changed from an object to a list.  Again, expect an update in the future to fix the code samples once the changes are published.

In our TimeZone directive, let's add the `bind` property.

```javascript
import {Decorator} from 'angular2/angular2';
import {PropertySetter} from 'angular2/src/core/annotations/di';

@Decorator({
    selector: '[data-tz-id]',
    bind: { "id" : "data-tz-id" },
    events: { click: 'onTimeZoneClick()' }
})
export class TimeZone{
    constructor(){
        console.log("TimeZone constructed");
    }

    onTimeZoneClick(){
        console.log("time zone " + this.id + " was clicked!");
        this.active = true;
    }
}
```

<div style="text-align:center; margin-bottom:20px;">
    <img src="/images/timezone/timezoneoutput_clickwithid.jpg" style="max-height: 250px">
</div>

And just like that, we have an `id` property in our TimeZone directive that stores our time zone id.  But we also want to show the visual state to our users. Ideally, when a time zone is selected, it would be highlighted on our map.  The easiest way would be to add a class based on the state of the component.  But again, since we can't modify the SVG template, how can we add a class to the host element?

The answer comes by means of a `PropertySetter`.  A `PropertySetter` is an object that can be injected into our directive instance and allows us to modify a DOM property.  In the case of modifying classes, if we prefix our property name with "class.", we have a neat shortcut that will add/remove the class whenever the property is set to true or false.

```javascript
import {Decorator} from 'angular2/angular2';
import {PropertySetter} from 'angular2/src/core/annotations/di';

@Decorator({
    selector: '[data-tz-id]',
    bind: { "id" : "data-tz-id" },
    events: { click: 'onTimeZoneClick()' }
})
export class TimeZone{
    constructor(@PropertySetter('class.active') setClassProperty:Function){
        console.log("TimeZone constructed");

        this.setClassProperty = setClassProperty;
    }

    onTimeZoneClick(){
        console.log("time zone " + this.id + " was clicked!");

        this.active = !this.active;
        this.setClassProperty(this.active);
    }
}
```

### The Time Zone Map Component



###