# Sofa Coding Guide

## Table of Contents
1. [Preamble](#preamble)
2. [Commit Message Convention](#commit-message-convention)
3. [Directory and File Structure](#directory-file-and-repository-structure)
4. [Naming Conventions](#naming-conventions)
5. [Code indentation](#code-indentation)
6. [Code alignment](#code-alignment)
7. [Prefer functional code](#prefer-functional-declarative-code-over-imperative-code)
8. [Business logic](#keep-business-logic-in-sofa-services)
9. [Modularize app code](#modularize-app-code)
10. [Services](#services)
11. [Directives](#directives)
12. [Dependency injection syntax](#dependency-injection-syntax)
13. [Tests](#tests)


## Preamble

The idea of this guideline is to be as concise as possible and as explicit as needed.
The ultimate goal is to have a consistent sofa code base.

## Commit Message Convention

We follow the [AngularJS Commit Message Convention](https://github.com/ajoslin/conventional-changelog/blob/master/CONVENTIONS.md) in order to be able to automatically generate nice changelogs from version control history.

## Directory, file and repository structure

### sofa app

Files should be grouped by feature rather than by type. For instance all files related to the product page should be in one `product` directory. Files that contain logic that is shared across large parts should go into a directory called `common`.

```js
/src
    /app
        /cart
            cartController.js
            cart.tpl.html
            cart.scss
        /product
            productController.js
            product.tpl.html
            product.scss
        /trusted-shops
            trustedShopsController.js
            trustedShopsService.js
            trusted-shops.tpl.html
            trusted-shops.scss
        /common
            selectionService.js
    app.js
index.html
```

### sofa components

The same structural rules apply to sofa components with the only difference that there is no `app` folder. Also most components are small enough to have all source files directly in `src` without further grouping. 

Use the [sofa-component-seed](https://github.com/sofa/sofa-component-seed) as a starting point for new sofa components. 

## Naming conventions

All files should start with lower case. JavaScript files should be written in camelCase whereas html and (s)css should use dash-case.


## Code indentation

Use four spaces. Period.

## Code Alignment

Align long code chains at the dot.

```js
// Avoid

getCustomer().getOrders().filter(function(order) { return order.total > 0; }).forEach(function(order) { orderService.archive(order);})

// Recommended

getCustomer()
    .getOrders()
    .filter(function(order) { return order.total > 0; })
    .forEach(function(order) { orderService.archive(order); });
```

## Prefer functional, declarative code over imperative code

As a rule of thumb, the less code one has to read, the easier it's to grasp. Imperative code focusses on *how* something works and makes it much harder to get *what* is being done. Functional code is more declarative and focusses on the *what* rather than the *how*.

```js
// Avoid

var friends = ['Pascal', 'Patrick', 'Christoph', 'Alex'];
var pFriends = [];

for (i = 0; i < friends.length; ++i) {
    if (friends[i].substring(0,1) === 'P') {
        pFriends.push(friends[i]);
    }
}

// Recommended

var friends = ['Pascal', 'Patrick', 'Christoph', 'Alex'];
var pFriends = friends.filter(function(friend) { return friend.substring(0,1) === 'P'; })
```

## Keep business logic in sofa services

Business logic is best placed in sofa services as it enables greater reuse. Keep controller code as little as possible. As a rule of thumb only code directly related to presentation logic is allowed to stay inside controllers.
.

## Modularize app code

Keep the sofa app as dump as possible and create sofa components even for whole pages or group of pages. E.g. create sofa packages for product lists, product pages basket widgets and more. This way code sharing can be maximized across projects.

## Services

Sofa services are defined in plain old JavaScript without direct Angular dependencies. It's allowed to use `$http` and `$q` though. Even if those do look like Angular dependencies, they are defined as sofa services that share exactly the same interface as the equally named Angular services. Depending on if sofa is used within an Angular context or not they will be instantiated as Angular services or pure sofa services.

Let's take a look at how a sofa service should be defined and examine the details afterwards.

```js
'use strict';
/* global sofa */
/**
 * @name StateResolverService
 * @class
 * @namespace sofa.StateResolverService
 *
 * @description
 * `sofa.StateResolverService` is a service to resolve human readable URLs into states that
 * can be dealed with on an application level.
 */
sofa.define('sofa.StateResolverService', function ($q, $http, configService) {
    var self            = {},
        states          = {};

    /**
    * @method registerState
    * @memberof sofa.StateResolverService
    *
    * @description
    * registers a state
    *
    * @example
    * stateResolverService.registerState(state);
    *
    * @param {object} state The state to be registered.
    *
    */
    self.registerState = function (state) {
        states[state.url] = state;

        somePrivateFunction();
    };

    var somePrivateFunction = function () {
        console.log('doing private things');
    };

});
```

1. Make sure to start the service with `'use strict'` at the first line
2. Annotate the service header as shown above
3. Use `sofa.define(name, fn)` to define the service constructor function
    - 3a. Define the service within the `sofa` namespace
    - 3b. Use the `Service` suffix to mark the constructor function as a service
    - 3c. Pass in a function with all dependencies as parameters
4. Define and empty `self` object at the very top inside the constructor function.
5. Attach public methods to the `self` object and annotate them with documentation as seen above
6. Place private functions freely but arrange them
    6a. before all public methods if they are shared across multiple public methods
    6b. directly after the public methods that uses it, if it's not shared widely among public methods

## Directives

### Use an isolate scope for element directives. Use inherited scope for attribute directives.

Using an isolate scope forces you to expose an API by giving the component all the data it needs. This increases reusability and testability. Attribute directives should not have an isolate scope because doing so overwrites the current scope.

```js
// Recommended
angular.module('alertListComponentModule', [])
  .directive('alertList', [
    function alertListDirective() {
      return {
        restrict: 'E',
        scope: {}
      };
    }
  ]);

// Avoid
angular.module('alertListComponentModule', [])
  .directive('alertList', [
    function alertListDirective() {
      return {
        restrict: 'E'
      };
    }
  ]);
```

```js
// Recommended
angular.module('alertListComponentModule', [])
  .directive('alertList', [
    function alertListDirective() {
      return {
        restrict: 'A',
        scope: true
      };
    }
  ]);

// Avoid
angular.module('alertListComponentModule', [])
  .directive('alertList', [
    function alertListDirective() {
      return {
        restrict: 'A'
      };
    }
  ]);
```

### Use `controllerAs` 

Bind to the controller rather than to the `$scope`. [Read up here](http://toddmotto.com/digging-into-angulars-controller-as-syntax/).

### Link function parameters

Name the parameters in the link function `scope`, `element` and `attrs`. We called the second parameter `$element` in the past because it is widely common to prefex jQuery(lite) wrapped DOM elements with `$`. We abandoned this rule to be more aligned with the AngularJS code base though.

## Dependency Injection Syntax

Don't use array notation. Use [ng-annotate](https://github.com/olov/ng-annotate) instead. This applies to the sofa app and all Angular sofa components.

## Tests

Tests should be written with jasmine and be named as the file under test with an additional `.spec` suffix. For instance, if `userService.js` is the file under test, the test file should be called `userService.spec.js`. It should be placed directly next to the file being tested. 