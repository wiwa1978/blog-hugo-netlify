---
title: Laravel ToDo REST API - client
date: 2020-04-22T17:19:50+01:00
draft: True
categories:
  - Web Development
  - Programming
  - Backend
  - Frontend
tags:
  - Laravel
---

### Introduction

### Installing Vue

```
WAUTERW-M-65P7:~ wauterw$ npm install -g @vue/cli
WAUTERW-M-65P7:~ wauterw$ vue --version
@vue/cli 4.3.1
```

### Creating Vue project

```
WAUTERW-M-65P7:~ wauterw$ vue create laravel-todo-api-client
WAUTERW-M-65P7:laravel-todo-api-client wauterw$ npm run serve
```
![VUE](/images/2020-04-22-1.png)

### Installing Bootstrap 4

```
WAUTERW-M-65P7:laravel-todo-api-client wauterw$ npm install bootstrap jquery popper.js
```
and import it into the main.js script by adding these lines to the top

```
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'
```

### Installing additional packages

```
WAUTERW-M-65P7:laravel-todo-api-client wauterw$ npm install axios vue-router
```


### Reorganizing things a bit

I like to work with a seperate routes file, similar as such to Laravel which stores the routes also in a dedicated folder. Let's change our app to accomodate that.

Create a folder `router` (under src folder). Inside that folder, create a file called `index.js`.

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'

Vue.use(Router)

export default new Router({
   mode: 'history',
   routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    }
  ]
})
```
The `main.js` file also needs to be modified as this is currently containing all the route related information.

```javascript
import Vue from 'vue'
import App from './App.vue'
import 'bootstrap'
import 'bootstrap/dist/css/bootstrap.min.css'
import router from './router'

Vue.config.productionTip = false

new Vue({
  router,
  render: h => h(App),
}).$mount('#app')
```
Make also the following changes to the App file, otherwise it will just always render the HelloWorld component.
```javascript
<template>
  <div id="app">
     <router-view/>
  </div>
</template>

<script>

export default {
  name: 'App',

}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>

```
Once you modified these files, run the Vue application again to verify that everything is still working properly. Note that we did not add functionality or changed anything fundamentally to our app, we have just re-arranged things a little bit.

### Create the Todo component
Now that all our routes are available in a seperate file under the `router` folder, we can make some changes to that file first. First off, we will add a new route for our Todo endpoint.

```javascript
import Vue from 'vue'
import Router from 'vue-router'
import HelloWorld from '@/components/HelloWorld'
import Todos from '@/components/Todos'

Vue.use(Router)

export default new Router({
   mode: 'history',
   routes: [
    {
      path: '/',
      name: 'HelloWorld',
      component: HelloWorld
    },
    {
      path: '/todos',
      name: 'Todos',
      component: Todos
    }
  ]
})
```
In the `components` folder, create a file called `Todos.vue`.

```javascript
<template>
  <div>
     This is the todo page
  </div>
</template>

<script>
export default {
  name: 'Todo',

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>

```
If no mistakes were made, then you will see the following:
![VUE](/images/2020-04-22-1.png)


### REST API

Verb	   Path	                  Action	      Route Name
GET	   /api/cruds	            index	         cruds.index
GET	   /api/cruds/create	      create	      cruds.create
PUT	   /api/cruds/{id}	      update	      cruds.update
DELETE	/api/cruds/{id}	      destroy	      cruds.destroy





