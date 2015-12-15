---
layout: post
title: "An Ng2 Timezone Picker - Part 2.3: The TimeZonePicker Component"
date: 2015-12-14 22:11:04 -0700
comments: true
categories: [angular2, timezonepicker]
---

Now that we have our basic project scaffolding in place, it's time for us to get started writing our components.

So let's open up our root component, the TimeZonePicker component (located in the timezonepicker.ts file).  So the first thing to remember is that with Angular2, we are writing TypeScript.  Lucky for us, the basics of TypeScript look pretty similiar to ES6 (or ES2015, or whatever it's called now).  So let's start off by defining a class.

``` javascript timezonepicker.ts
export class TimeZonePicker {}
```

Boom!  Done!  We've got our TimeZonePicker class defined and exported.  Webpack refreshes our browser for us and...  yep we're good.  Alright, now let's make this a component.

First, we'll import the necessary Angular2 dependencies from the angular2 library.  Then, we'll use annotations to mark the class as an Angular2 component.

What are annotations, you ask?  I'm glad you brought that up.  I tend to use the term "annotations" instead of "decorators" because of my background with other languages (Java specifically).  However, in many other languages, (including JavaScript) "decorators" is the correct term.  Either way, think of annotations/decorators as a way of specifying metadata about your code that can be used at compile or run time to modify behavior.  If you are interested in learning more, you can read the [ES7 proposal for decorators](https://github.com/wycats/javascript-decorators).

> Author Note:
> As far as I can tell Typescript technically implements the ES7 decorator spec, so "decorator" is the right term in the context of JavaScript.  But old habits die hard.  So if you see me reference then as annotations, just smirk and move on.  In reality, there are a couple of [differences between the two](http://blog.thoughtram.io/angular/2015/05/03/the-difference-between-annotations-and-decorators.html), but for our purposes here, we don't really care.

So let's add to our code!

``` javascript timezonepicker.ts
import {Component} from 'angular2/angular2';

@Component({
    selector: "timezone-picker",
    template : `
        <div class="picker">
            This is our picker!
            <timezone-map (change)="onChange($event)" [tz-id]="tzId"></timezone-map>
        </div>
    `
})
export class TimeZonePicker {}
```

Alright, now we getting into the fun stuff.  So let's take a quick look at that code.  A couple of important explanations:

1. We import the Component decorator on line 1.  As we expand our component definition, we'll import more things from angular2
2. Line 3 has our `@Component` annotation
3. Line 4 specifies the selector for our component.  Most CSS2 selectors are supported here (excluding psuedo-selectors)
4. Line 5 begins our template definition.  Just like AngularJS, we can define the template inline, or provide a templateUrl.  With TypeScript though, we getting the added benefit of using  [ES6 template strings](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/template_strings).  A major improvement over concatenated strings!
5. Line 8 references our TimeZoneMap component, which doesn't exist yet.  That's fine.  The component just won't evaluate until it does.  Note that anything surrounded with `()` is considered an "output" of that component, and anything surrounded with `[]` is an input.  We'll see this again soon.

To test our new component, let's make sure that it is rendered when we run our app.  Open up `app.ts` and let's modify a few things.

``` javascript app.ts
import {Component} from 'angular2/angular2';
import {TimeZonePicker} from "./timezonepicker/timezonepicker";

@Component({
	selector: 'app',
	template: `
		<div>Hello Angular2!</div>
		<timezone-picker></timezone-picker>
	`,
	directives: [TimeZonePicker]
})
export class App {}
```

Alright, let's talk about our changes.

1. On line 2, we added a second import statement that imports our TimeZonePicker
2. On line 8, we modified the template to include an element that matches the TimeZonePicker selector
3. On line 10, we added a `directives` property to our `@Component` defintion.  Unlike AngularJS, Angular2 will not automatically evaluate all components/directives.  By explicitly listing what directives are used, Angular2 gets a major perf boost, since it doesn't have to check for all of them.

And that's it for our code changes to `app.ts`!

While we have `app.ts` open though, let's do a fun experiment.  If you are using an IDE with built-in TypeScript/ES2015 support, try deleting line 2 and see what happens.  Here's what happens for me:

<div style="text-align:center; margin-bottom:20px;">
    <img src="/images/timezone2-3/autocomplete.png">
</div>

Wait.. is that... autocompletion of imports?!?  Having written ES5 for so long, I forgot what helpful front end autocomplete was.  Yes, I realize autocomplete exists for ES5.  And no, it's not normally very helpful.  Coding in TypeScript/ES2015, however, feels great.  Okay, end of tangent.

Let's fire up our app and see if everything worked.

<div style="text-align:center; margin-bottom:20px;">
    <img src="/images/timezone2-3/result.png">
</div>

And that looks pretty good!  We see our "This is our picker!" text, and the DOM shows that our template has been rendered correctly.  We've built the first component of our picker!  Success!

In our next post, we'll give the TimeZoneMap component and TimeZone directive the same treatment as our TimeZonePicker.