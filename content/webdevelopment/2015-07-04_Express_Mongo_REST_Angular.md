---
title: Consume Express API with Angular
date: 2015-07-04T20:19:50+01:00
draft: True
categories:
  - Web Development
  - Programming
  - Frontend
  - All
tags:
  - MEAN
  - Express
  - Mongo
---
> Quick note: the original post dates from 04-07-2015 but got updated at 02-04-2020 with latest versions and it was essentially written from scratch again.
### Introduction
In a previous post, we have created a REST API. In this post, we will build further on it and provide it with a nice user interface developed using Angular. 

To get started, we will first clone the Github repository that contains the server code.

```bash
WAUTERW-M-65P7:Programming wauterw$ git clone https://github.com/wiwa1978/blog-hugo-netlify-code.git
WAUTERW-M-65P7:Programming wauterw$ cd Express_Mongo_REST_API
WAUTERW-M-65P7:Programming wauterw$ npm install
``` 
If all went well, you should be able to go to http://localhost:3000 and see a welcome page. That indicates that our server is running.

### Angular code

Let’s now add the Angular specific code…

By default, Express is using Jade/Pug templates to create client code. When using Angular, we want to prevent that the Jade/Pug code is running. So therefore do the following in the app.js file:

```javascript
var app = express();
// view engine setup
// app.set('views', path.join(__dirname, 'views'));
// app.set('view engine', 'pug');
```
As you can see, we have simply uncommented the lines that informs Express to use the Jade/Pug template. As a consequence, Express won’t look for Jade/Pug files in the ‘views’ directory. We will put the Angular client files in a folder called `public`. Obviously you are free to choose your own folder name as long as you inform Express about it. In the app.js file, you can do it as follows:

```javascript
app.use(express.static(path.join(__dirname, 'public')));
```
Everything is now ready for the actual client code. As Angular is a client framework to build single page applications, we will put all code in a file called `index.html` in the `public` folder we created.

The following snippet is adding Angular to the HTML code, refer to the ng-app=’MyApp’ reference which is part of the html tag. We have also included the reference to the Angular javascript file as well as a reference to a controller.js file which will contain all of our ‘todo item intelligence’. As I’m a big fan of Bootstrap, obviously I’ve included the stylesheet reference as well.

```html
<!DOCTYPE>
<html ng-app='myApp'>
<head>
	<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap.min.css">
	<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.5/css/bootstrap-theme.min.css">
	<title>Todo application</title>
</head>

<body>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.3.16/angular.min.js"></script>
  <script src="controllers/controller.js"></script>
</body>
</html>
```
Let’s now have a look at the body part. Essentially, we want to display a table that contains all todo items retrieved from the server.

```html
<div class="container" ng-controller="AppCtrl">
	<h1>Todo Application</h1>
	<table class="table">

		<thead>
			<tr>
				<th>Content</th>
				<th>Description</th>
				<th>Status</th>
				<th>Created</th>
				<th>Updated</th>
				<th>Action</th>
				
			</tr>
		</thead>
		<tbody>
			<tr>
          	<td><input class="form-control" ng-model="todo.content"></td>
          	<td><input class="form-control" ng-model="todo.description"></td>
          	<td>
          		<div ng-if="todo.edit == true">
    					<div class="form-group">
				         <div class="radio">
				            <label>
				               <input type="radio" name="true" value="true" ng-model="todo.completed">
				                  Completed
				            </label>
				         </div>
				         <div class="radio">
				            <label>
				               <input type="radio" name="false" value="false" ng-model="todo.completed">
				                  Pending
				            </label>
				         </div>
				      </div>
					</div>
		      </td>
          	<td></td>
          	<td></td>
          	<td>
          		<div ng-if="todo.edit == true">
    				   <button class="btn btn-primary" ng-click="updateTodo()">Update Todo</button>
					</div>
					<div ng-if="todo.edit != true">
    					<button class="btn btn-primary" ng-click="addTodo()">Add Todo</button>
					</div>
          	</td>
        	</tr>
        	<tr ng-repeat="todo in todolist">
				<td>{{todo.content}}</td>
				<td>{{todo.description}}</td>
				<td>
					<div ng-if="todo.completed == true">
    					<span class="label label-success">Completed</span>
					</div>
					<div ng-if="todo.completed != true">
    					<span class="label label-warning">Pending</span>
					</div>
				</td>
				<td>{{todo.created_at| date:'dd-MM-yyyy -- HH:mm:ss'}}</td>
				<td>{{todo.updated_at| date:'dd-MM-yyyy -- HH:mm:ss'}}</td>
				<td>
					<button class="btn btn-success" ng-click="editTodo(todo._id)">Edit</button>
					<button class="btn btn-danger" ng-click="removeTodo(todo._id)">Remove</button>
				</td>
			</tr>
		</tbody>	
	</table>
</div>
```
In the above snippet, you see that `ng-controller=”AppCtrl”` binds the code to the controller. We also use an Angular ‘ng-repeat’ statement to loop over all the items and we use the {{}} directives to effectively display the content on screen. In the body part of the table, you can also see that we define two input fields that refer to an ‘ng-model’ and two buttons that refer to an `ng-click` event, respectively addTodo() and updateTodo()

Let’s then have a look at the controller.js file.

```javascript
var myApp = angular.module('myApp', []);
myApp.controller('AppCtrl', ['$scope', '$http', function($scope, $http) {
   
url = 'http://localhost:3000/todos/'

var refresh = function() {
  $http.get(url).success(function(response) {
    $scope.todolist = response;
    $scope.todo = "";
    $scope.todo.edit = false;
  });
};
```
In the above code, you’ll see that the myApp is defined in the controller file and that the controller ‘AppCtrl’ is also defined. We also define the $scope and $http variables. We need $scope to be able to use the info in the HTML file and the $http to be able to make the REST calls to the server.

We can also see that we do a HTTP GET call to the URL http://localhost:3000/todos, which is effectively our server (as we run everything on localhost for now). The JSON response from the server is stored in the $scope.todolist which is used in the `ng-repeat` statement (cfr: ng-repeat=”todo in todolist”).

Let’s look at the addTodo(), updateTodo() and deleteTodo() functions:

```javascript
$scope.addTodo = function() {
  console.log($scope.todo);
  $http.post(url, $scope.todo).success(function(response) {
    console.log(response);
    refresh();
  });
};

$scope.removeTodo = function(id) {
  $http.delete(url + id).success(function(response) {
    console.log("deleting: " + response);
    refresh();
  });
};

$scope.editTodo = function(id) {
  console.log(id);
  $http.get(url + id).success(function(response) {
    $scope.todo = response;
    $scope.todo.edit = true;
  });
};

$scope.updateTodo = function() {
  console.log("Completed" + $scope.todo.completed);
  $http.put(url + $scope.todo._id, $scope.todo).success(function(response) {
     console.log("new updated: " + response.updated_at);
    refresh();
 	})
};
```
In the addTodo() function, we see that we create an HTTP POST and pass it the $scope.todo, which essentially contains all the info from the input fields (in the todo.content and todo.description).

In the UpdateTodo(), we create an HTTP PUT request to the URL and pass it again the $scope.todo parameter, which contains the updated todo items.

Finally, the removeTodo() is using an HTTP delete request to the URL which we pass the id of the todo item.

You can also see that each time the refresh function is called each time. The refresh function is wrapped around the HTTP GET method that retrieves all todo items, so in fact each time we add, update or delete a todo item, we retrieve the whole list of todo items again so the page is updated automatically.

### Testing the application

We are now ready to do some basic functional testing. Go to `http://localhost:3000` and you should see the todo application popping up. Let's add couple of items and see the result:

![Angular](/images/2015-07-04-1.png)

Let's now update an item. I took the 5th todo item and changed content and description of the todo item as well as the status.

![Angular](/images/2015-07-04-2.png)

Next we will delete that 5th todo item.
![Angular](/images/2015-07-04-3.png)

That's it folks. Nothing more for today. We created a rudimentary frontend application that consumes are REST API. Code can be found at my Github repo [here](https://github.com/wiwa1978/blog-hugo-netlify-code/tree/master/Express_Mongo_REST_Angular).