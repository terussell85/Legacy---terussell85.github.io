---
layout: post
title: "The AngularJs Promise Anti-Pattern That Makes Me Cry"
date: 2013-12-23 10:57:37 -0800
comments: true
categories: [angularjs, promises]
---

### TLDR
  * Try to wrap any promise work in services.  Rarely should controllers do anything but consume promises.
  * Expose promises as return values of service calls instead of forcing callbacks as parameters.  This allows for multiple callbacks. (and much simpler code)
  * Setup callbacks as behaviors instead of global "success" or "failure".  This allows for much easier chaining if necessary

<!--more-->

### The anti-pattern

Asynchronous interactions in AngularJs are built on promises.  A [lot](http://andyshora.com/promises-angularjs-explained-as-cartoon.html) [has](http://markdalgleish.com/2013/06/using-promises-in-angularjs-views/) [been](http://johnmunsch.com/2013/07/17/angularjs-services-and-promises/) [said](https://egghead.io/lessons/angularjs-promises) on the subject of promises already.  So I won't cover their purpose extensively here.  Long story short, use them.

Sadly enough, the reasons for using promises are still widely misunderstood.  One pattern, in particular, I still see quite often with new users of promises.  I don't have a cool name for it, but if I did, it would be: "The AngularJs promise anti-pattern that makes me cry."  But that's just because I'm over-dramatic.

In general, I've seen this pattern in the following form:

```javascript

function caller(){
  callee({}, function(){}, function(){});
}

function callee(toSave, success, failure){
  var promise = $http.post("/place", toSave);

  promise.success(function(successData){
    if(success){
      success(successData);
    }
  });

  failure.error(function(errorData){
    if(failure){
      failure(errorData);
    }
  });
}

```

So before I start a flame war on this one, I'll go ahead and say nothing is "wrong" with this code.  In some cases I would even argue that it is perfectly acceptable.  You can see that we are trying to separate concerns, abstract callbacks, and keep things as clean as possible.  However, using promises this way eliminates many of the reasons that promises exist in the first place.  How so?  Let's step through a scenario.


### Our initial code

Let's say that we are working along one day, listening to our favorite Pandora station when our project manager comes and taps us on the shoulder.  "We need a page that saves an item and then moves on to another page.", he says.  No problem.  Being savvy programmers (with little time), we whip up the following controller and service in a few minutes.

```javascript
//Page controller
angular.module("app").controller("testCtrl", function($scope, $location, test){

  /**
   * Responds to click events on the save button
   * Saves the items and advances to the next page if successful
   */
  $scope.onSaveClick = function(){
    test.saveItem($scope.data.item,
      //success callback
      function(){
        $scope.data.backgroundColor = "red";

        //go to next page
        $location.path("/place1");
      },
      //error callback
      function(){
        //go to next page
        $location.path("/place2");
      });
  }
}

//Helper service
angular.module("app").service("test", function(){
  function saveItem(item, success, fail){
    var promise = $http.post("/item", item);

    //wrap the callback in another callback, because that's the cool way
    promise.success(function(itemData){
      if(success){
          success(itemData);
      }
    });

    //do it a different way here.  Because two ways is always better than one!
    promise.error(fail);
  }

  return {
    saveItem : saveItem
  }
})

```

And voila, a page that saves an item on button click.  It works as designed.  I'm sure quite a few readers threw up in their mouths reading that code (I almost did as I wrote it).  We could easily point out a number of issues already with the way this is designed, but that's not the point of this exercise.  Let's step through piece by piece and see why this is bad.  Plus, I'm still surprised how often I will go back and read my own code and find designs like this.  We all have bad days.

### Exposing the issues: Changing requirements

Let's say that our project manager comes back and decides that the requirements have changed.  "Customers are complaining about getting lost," he says.  "We need to add a second button that saves the item but stays on the same page. We need to support both buttons."

Great, no problem.  We'll just make some minor tweaks to our controller to support this functionality.  We take a step back and look at our code.  Lets look at our plan of attack.

* Another event handler for "update in place" button clicks.
* both handlers call the same save code.
* only the first save should route to another page
* both error callbacks do the same thing

So we take a first stab at it.  And after a few iterations of refactoring, come out with the following:

```javascript
//Page controller
angular.module("app").controller("testCtrl", function($scope, $location, test){

  function commonUiLogic(){
    $scope.data.backgroundColor = "red";
  }

  $scope.onSaveClick = function(){
    test.saveItem($scope.data.item, function(){
      commonUiLogic();
      //go to next page
      $location.path("/place1");
    }, function(){
      //go to next page
      $location.path("/place2")
    });
  }

  $scope.onUpdateInPlace = function(){
    test.saveItem($scope.data.item, commonUiLogic, function(){
      //go to next page
      $location.path("/place2")
    });
  }
}

//service left out for brevity

```

All things considered, not too bad.  We have a "commonUiLogic" method that updates our UI for us in both cases.  We call the same service method for saving on both updates.  We are coding masters.

### Starting to feel the pain: Even more changing requirements

But then it happens, our project manager comes over and tells us the requirements have changed again.  "Red is too powerful.  We need more blue.  We need another button that saves, turns the text blue, and then goes to the next page".  Oh boy.  K.  Let's try this out.

```javascript
angular.module("app").controller("testCtrl", function($scope, $location, test){

  function commonUiLogic(){
    $scope.data.backgroundColor = "red";
  }

  function turnTextBlue(){
    $scope.data.textColor = "blue";
  }

  $scope.onSaveClick = function(){
    test.saveItem($scope.data.item, function(){
      commonUiLogic();
      //go to next page
      $location.path("/place1");
    }, function(){
      //go to next page
      $location.path("/place2")
    });
  }

  $scope.onUpdateInPlace = function(){
    test.saveItem($scope.data.item, commonUiLogic, function(){
      //go to next page
      $location.path("/place2")
    });
  }

  $scope.anotherButton = function(){
    test.saveItem($scope.data.item, function(){
      commonUiLogic();
      turnTextBlue();
    }, function(){
      //go to next page
      $location.path("/place2")
    });
  }
}

//service left out for brevity

```
Alright, that was a relatively simple addition.  But we are beginning to see how these callbacks can quickly get out of hand.  Every new difference can cause pretty large refactoring, and it isn't as easy to share common code as it should be.

### There must be a better way

So let's take a step back.  What if we had been a bit smarter in our original design of our controller and service.  Could we have made this implementation even simpler?  For brevity's sake, lets just jump to the final refactor on this one.  We need three buttons, changing background colors, changing font colors, and fancy routing.

```javascript
//Page controller
angular.module("app").controller("testCtrl", function($scope, $location, test){

  function goToPlace1(){
    //go to next page
    $location.path("/place1");
  }

  function changeBackgroundToRed(){
    $scope.data.backgroundColor = "red";
  }

  function changeTextToBlue(){
    $scope.data.backgroundColor = "blue";
  }

  function goToPlace2(){
    //go to next page
    $location.path("/place2")
  }

  $scope.onSaveClick = function(){
    test.saveItem($scope.data.item).then(changeBackgroundToRed)
                                   .then(goToPlace1)
                                   .catch(goToPlace2);
  }

  $scope.onUpdateInPlace = function(){
    test.saveItem($scope.data.item).then(changeBackgroundToRed)
                                   .catch(goToPlace2);
  }

  $scope.anotherButton = function(){
    test.saveItem($scope.data.item).then(changeTextToBlue)
                                   .then(goToPlace1)
                                   .catch(goToPlace2);
  }
}

//Helper service
angular.module("app").service("test", function(){
  function saveItem(item){
    return $http.post("/item", item);
  }

  return {
    saveItem : saveItem
  }
})

```

Awww.  That's better.  Don't you feel lighter and happier?

Let's note the major differences in this original implementation.  First, the service now returns the promise directly, instead of wrapping it by passing in the success and failure callbacks.  We no longer have inline functions as callbacks.  In fact, we don't even categorize callbacks as "success" or "failure."  Instead, they are behaviorally specified.  That way, we can chain each behavior in different combinations in order to get the deserved result per save request.  Overall, the readability of the code is greatly improved.  And with concise behavioral callbacks, unit testing will be a breeze.

### Minor changes, big impact

The differences between the two implementations are subtle, and in many cases won't matter unless you are the third or fourth developer working in the codebase.  But in summary, some best practices around using promises include:

* Try to wrap any promise work in services.  Rarely should controllers do anything but consume promises.
* Expose promises as return values of service calls instead of forcing callbacks.  This allows for multiple callbacks.
* Setup callbacks as behaviors instead of global "success" or "failure".  This allows for much easier chaining if necessary

--T