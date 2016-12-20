<!--1:30 10 minutes -->

<!--Hook: So in the last unit we talked a lot about routing between the back end and the front end.  Basically, we would define a path and a method to match on either side.  This was such a revolutionary idea when it came out.  A little while later, web developers started looking around and thinking...wait, if we don't need a database or a back-end...can we just do that on the front end now?  No more waiting for HTTP responses, just immediately switch to a new view, with new HTML, CSS, and JS code.  That would be cool.  Yes, it would be cool.  And that's what we'll learn today. -->

<!--Also worth noting, we have shown ng-controller and ng-model, but the only one of the trio we haven't yet shown is ng-view --we'll show that today! -->

## Angular Routing Lab

| Objectives |
| :--- |
| **Explore** Routing in Single Page Apps |
| **Create** route-specific view templates and controllers. |
| **Create** RESTful Index and Show routes for a (Wine) resource. |
 
In this lab we will be working with templates and routing in angular.

## Introduction

What is Routing in AngularJS?

If you want to navigate to different pages in your application, but you also want the application to be a SPA (Single Page Application), with no page reloading, you can use the ngRoute module.

The ngRoute module routes your application to different pages without reloading the entire application.

### Goal

When a user goes to `/` they should see a list of wines (`wines#index`).
When a user goes to `/wines/:id` they should see a single wine (`wines#show`).

Our data (a list of wines) lives at the bottom of `app.js`. Eventually we will use AJAX to retrieve this so we can perform all CRUD operations, but for now it's hardcoded.

<!--1:40 10 minutes -->

### Step 1. Setup
* Clone this repo. You should `git checkout -b in-progress` or something similar to work there.
* **Make sure to `bower install`.**
* Note: We will need to run a local server once we start playing with routing.
    - In the application directory run `python -m SimpleHTTPServer`.
    - Then open your browser to "localhost:8000" (or similar).
    - The Developer console should display: `Angular is working.`

<!--1:50 20 minutes-->

## Step 2. ng-route
A Single Page App needs a way of responding to user navigation. In order to perform "frontend routing", we need a way to capture URL changes and respond to them. For example, if the user clicks on a link to "/wines/1414", we need our Angular application to know how to respond (what templates, controllers, and resources to use). What we *don't* want to happen is for the request to reach the server.

1. Include `angular-route`:
    * Run `bower install --save angular-route` in your terminal.
      - You may get a prompt like, `? Answer`. If so, you are being asked which version of angular-route you want. I've found that typing `1` and hitting enter usually lets you proceed and you can confirm that `angular-route` has installed properly if the folder appears in your `bower_components` folder.
    * Go to `index.html` and uncomment the angular-route script (index.html line 14).
    * Add an `ng-view` attribute to the `div` on `index.html`, line 23.

### ng-view

The `ng-view` directive is the heart of all our front-end routing.  Applications can only have one ng-view directive, and this will be the placeholder for all views provided by the route.  In other words, wherever our ng-view directive is, this is will be replaced by whatever we choose to render from our routes (usually an HTML template).

2. Configure your routes:
    * In `app.js`, we need to modify line 1 to add the `ngRoute` module:

        ``` javascript
            var app = angular.module('wineApp', ['ngRoute'])
        ```

    * Next, we need to add our first route:

        ``` javascript
            app.config(function($routeProvider){
              $routeProvider
                .when('/', {
                  template: 'Home!'
                });
            });
        ```
3. Fire up your server:
    * Make sure you are still running `python -m SimpleHTTPServer`.
    * Then open your browser to "localhost:8000" (or similar).
    * You should see "Home!"

4. Use a template file instead of a string:
    * Change `template: 'Home!'` to `templateUrl: '/templates/wines-index.html'`
    * Refresh, you should see the content of `/templates/wines-index.html`.

5. Set up a controller:
    * It's time to attach a template to a specific controller, all we have to do is modify our route so that it looks like this:

        ``` javascript
            app.config(function($routeProvider){
              $routeProvider
                .when('/', {
                  // template: 'Home!',
                  templateUrl: '/templates/wines-index.html',
                  controller: 'WinesIndexController'
                });
            });
        ```

    * Now let's add a single value to `WinesIndexController` (in `app.js`):

        ``` javascript
            function WinesIndexController($scope){
              console.log("Wine Index");
              $scope.hello = "wine index controller is working!";
            });
        ```
    * And update our template to include:
        - `{{hello}}`
    * When you refresh you should see: "wine index controller is working!"

<!--2:10 20 minutes -->

### Step 3. Wine List Challenge

#### Factories

**Factories** are a type of **Provider** in Angular.  They are a great way to _DRY out our code_ and _separate concerns_.  Specifically, if we notice we are using the same method in two different controllers, it's probably a good idea to break that out into its own factory so we don't repeat ourselves.  Also, if we are communicating with a Database or a 3rd Party API, it is a good idea to isolate that code from the display of the data on our front end so we can troubleshoot and improve our program more easily.

Can you display a list of all the wine names and prices on the wines index page? (Start by using the mock data object called `ALL_WINES` at the bottom of `app.js`).

What directive would you use to loop through a list of wines?

Can you get it working using the `WineFactory`, without using `ALL_WINES` directly?
- How would you inject the `WineFactory` into `WinesIndexController`?
- How would you query *all* of the wines?

<!--2:30 15 minutes -->

### Step 4. Wine Show Challenge
To setup a `wines#show` route, we need to first figure out how to capture the id parameter in the URL.

For each of your wines on the `wines#index` page, let's add a link:
``` html
    <h5><a href="#/wines/{{wine.id}}">{{wine.name}}</a></h5>
```

When a user navigates to `/wines/:id` we want to display the wine with the matching id!

First, update the route:

``` javascript
$routeProvider
  .when('/', {
    templateUrl: 'templates/wines-index.html',
    controller: 'WinesIndexController'
  })
  .when('/wines/:id', { // the "id" parameter 
    templateUrl: 'templates/wines-show.html',
    controller: 'WinesShowController'
  })
```

Next, we need to inject a new module into `WinesShowController` called `$routeParams`:

``` javascript
WinesShowController.$inject = ['$scope', 'WineFactory', '$routeParams'];

function WinesShowController($scope, WineFactory, $routeParams) {
    console.log($routeParams.id);
});
```

Can you get it working now that you know how to grab the correct `id`? How would you display only that individual wine?

Get your `wines-show.html` template to show at least the following information: wine name, wine price, and wine description.

<!--2:45 5 minutes -->

### Step 5. HTML5 Mode
Add the following in your route configuration so that we don't have to use the query hash for navigation:
``` javascript
    $locationProvider.html5Mode({
      enabled: true,
      requireBase: false
    });
```

Now instead of linking to `#/wines/1424` in our `wines-index.html` template, we can link to "/wines/1424".

<!--Team independence can use this time to look at Sensus code with ui-router -->

<!--3:00 till end of day -->

### Independent Work: Prettify
Go crazy. Use Bootstrap to make a fancy index and show page, listing out all the wine info, and showing an image for each of them.

Here are some of the wine fields we have to work with:

``` json
{
    "id": 1429,
    "created_at": "2015-10-13T01:30:28.631Z",
    "updated_at": "2015-10-13T01:30:28.631Z",
    "name": "CHATEAU LE DOYENNE",
    "year": "2005",
    "grapes": "Merlot",
    "country": "France",
    "region": "Bordeaux",
    "price": 12,
    "description": "Though dense and chewy, this wine does not overpower with its finely balanced depth and structure. It is a truly luxurious experience for the senses.",
    "picture": "http://s3-us-west-2.amazonaws.com/sandboxapi/le_doyenne.jpg"
}
```

### Independent Work:  AJAX
Take a look at the $http lab from yesterday. Can you remove the ALL_WINES array in the WineFactory and replace with an $http call instead?

The endpoint is here: `http://daretoexplore.herokuapp.com/wines/`

The solution will be published on a separate branch at the end of this lab.

### Resources

- More on [Angular Routing](http://www.w3schools.com/angular/angular_routing.asp)
- More about [Providers](https://docs.angularjs.org/guide/providers) like factories.

## Licensing
All content is licensed under a CC­BY­NC­SA 4.0 license.
All software code is licensed under GNU GPLv3. For commercial use or alternative licensing, please contact legal@ga.co.
