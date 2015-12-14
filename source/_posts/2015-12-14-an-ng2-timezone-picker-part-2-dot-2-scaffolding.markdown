---
layout: post
title: "An Ng2 Timezone Picker - Part 2.2: Scaffolding"
date: 2015-12-14 13:02:12 -0700
comments: true
categories: 
---

In this post, we'll get the scaffolding of our project setup.

Our timezone picker will consist of four different components/directives.

1. App - the entry point into our Angular2 application
2. TimeZonePicker - the root component for our time zone picker
3. TimeZoneMap - the component that renders our SVG map
4. TimeZone - the directive that maps SVG elements to classes with attributes

By default, our project structure from the seed project looks like so:

<div style="text-align:center; margin-bottom:20px;">
  <img src="/images/timezone2-2/project_base.png">
</div>

Let's add a bit of scaffolding and some placholder files so that we can flesh out our components in the future.

1. Create the `src/components/timezone` directory
2. Create the `src/components/timezonemap` directory
3. Create the `src/components/timezonepicker` directory
4. Create the `src/assets` directory
5. In `src/components/timezone`, create `timezone.ts`
6. In `src/components/timezonemap`, create `timezonemap.ts`
7. In `src/components/timezonepicker`, create `timezonepicker.ts`
8. In `src/assets`, create `styles.css`
9. In `src/assets`, copy in the output map from our Kartograph.py script

In case you need them, here are the commands for steps 1-8.  Step 9 you'll still need to do by hand.

``` bash
mkdir src/app/components/timezone
mkdir src/app/components/timezonemap
mkdir src/app/components/timezonepicker
mkdir src/assets
touch src/app/components/timezone/timezone.ts
touch src/app/components/timezonemap/timezonemap.ts
touch src/app/components/timezonepicker/timezonepicker.ts
touch src/assets/styles.css
```

Once setup, our project structure should look like this:

<div style="text-align:center; margin-bottom:20px;">
    <img src="/images/timezone2-2/project_with_files.png">
</div>

In the next part of our series, we will start to write our Angular2 components.