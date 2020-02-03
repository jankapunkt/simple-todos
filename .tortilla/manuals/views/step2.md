# Step 2: Defining views with templates

[//]: # (head-end)


To start working on our todo list app, let's replace the code of the default starter app with the code below. Then we'll talk about what it does.

First, let's remove the body from our HTML entry-point (leaving just the `<head>` tag):

[{]: <helper> (diffStep 2.1 noTitle=true files="client/main.html")

##### Changed client&#x2F;main.html
```diff
@@ -1,25 +1,3 @@
 ┊ 1┊ 1┊<head>
 ┊ 2┊ 2┊  <title>simple-todos</title>
-┊ 3┊  ┊</head>
-┊ 4┊  ┊
-┊ 5┊  ┊<body>
-┊ 6┊  ┊  <h1>Welcome to Meteor!</h1>
-┊ 7┊  ┊
-┊ 8┊  ┊  {{> hello}}
-┊ 9┊  ┊  {{> info}}
-┊10┊  ┊</body>
-┊11┊  ┊
-┊12┊  ┊<template name="hello">
-┊13┊  ┊  <button>Click Me</button>
-┊14┊  ┊  <p>You've pressed the button {{counter}} times.</p>
-┊15┊  ┊</template>
-┊16┊  ┊
-┊17┊  ┊<template name="info">
-┊18┊  ┊  <h2>Learn Meteor!</h2>
-┊19┊  ┊  <ul>
-┊20┊  ┊    <li><a href="https://www.meteor.com/try" target="_blank">Do the Tutorial</a></li>
-┊21┊  ┊    <li><a href="http://guide.meteor.com" target="_blank">Follow the Guide</a></li>
-┊22┊  ┊    <li><a href="https://docs.meteor.com" target="_blank">Read the Docs</a></li>
-┊23┊  ┊    <li><a href="https://forums.meteor.com" target="_blank">Discussions</a></li>
-┊24┊  ┊  </ul>
-┊25┊  ┊</template>
+┊  ┊ 3┊</head>🚫↵
```

[}]: #

Create a new directory with the name `imports` inside `simple-todos` folder. Then we create some new files in the `imports/` directory:

[{]: <helper> (diffStep 2.2 noTitle=true)

##### Added simple-todos&#x2F;imports&#x2F;ui&#x2F;body.html
```diff
@@ -0,0 +1,15 @@
+┊  ┊ 1┊<body>
+┊  ┊ 2┊  <div class="container">
+┊  ┊ 3┊    <header>
+┊  ┊ 4┊      <h1>Todo List</h1>
+┊  ┊ 5┊    </header>
+┊  ┊ 6┊
+┊  ┊ 7┊    <ul>
+┊  ┊ 8┊      {{#each tasks}} {{> task}} {{/each}}
+┊  ┊ 9┊    </ul>
+┊  ┊10┊  </div>
+┊  ┊11┊</body>
+┊  ┊12┊
+┊  ┊13┊<template name="task">
+┊  ┊14┊  <li>{{text}}</li>
+┊  ┊15┊</template>
```

[}]: #

[{]: <helper> (diffStep 2.3 noTitle=true)

##### Renamed from simple-todos&#x2F;imports&#x2F;ui&#x2F;body.html to imports&#x2F;ui&#x2F;body.html


##### Added imports&#x2F;ui&#x2F;body.js
```diff
@@ -0,0 +1,11 @@
+┊  ┊ 1┊import { Template } from 'meteor/templating';
+┊  ┊ 2┊
+┊  ┊ 3┊import './body.html';
+┊  ┊ 4┊
+┊  ┊ 5┊Template.body.helpers({
+┊  ┊ 6┊  tasks: [
+┊  ┊ 7┊    { text: 'This is task 1' },
+┊  ┊ 8┊    { text: 'This is task 2' },
+┊  ┊ 9┊    { text: 'This is task 3' }
+┊  ┊10┊  ]
+┊  ┊11┊});
```

[}]: #

Inside our front-end JavaScript entry-point file, `client/main.js`, we'll _remove_ the rest of the code and import `imports/ui/body.js`:

[{]: <helper> (diffStep 2.4 noTitle=true)

##### Changed client&#x2F;main.js
```diff
@@ -0,0 +1 @@
+┊ ┊1┊import '../imports/ui/body.js';🚫↵
```

[}]: #

You can read more about how imports work and how to structure your code in the [Application Structure article](http://guide.meteor.com/structure.html) of the Meteor Guide.

In our browser, the app will now look much like this:

> #### Todo List
> - This is task 1
> - This is task 2
> - This is task 3

Now let's find out what all these bits of code are doing!

### HTML files in Meteor define templates

Meteor parses HTML files and identifies three top-level tags: **&lt;head>**, **&lt;body>**, and **&lt;template>**.

Everything inside any &lt;head> tags is added to the `head` section of the HTML sent to the client, and everything inside &lt;body> tags is added to the `body` section, just like in a regular HTML file.

Everything inside &lt;template> tags is compiled into Meteor _templates_, which can be included inside HTML with `> templateName}}` or referenced in your JavaScript with `Template.templateName`.

Also, the `body` section can be referenced in your JavaScript with `Template.body`. Think of it as a special "parent" template, that can include the other child templates.

### Adding logic and data to templates

All of the code in your HTML files is compiled with [Meteor's Spacebars compiler](http://blazejs.org/api/spacebars.html). Spacebars uses statements surrounded by double curly braces such as `#each}}` and `#if}}` to let you add logic and data to your views.

You can pass data into templates from your JavaScript code by defining _helpers_. In the code above, we defined a helper called `tasks` on `Template.body` that returns an array. Inside the body tag of the HTML, we can use `#each tasks}}` to iterate over the array and insert a `task` template for each value. Inside the `#each` block, we can display the `text` property of each array item using `text}}`.

In the next step, we will see how we can use helpers to make our templates display dynamic data from a database collection.

### Styling with CSS

To have a better experience while following the tutorial we suggest you copy-paste the following CSS code into your app:

[{]: <helper> (diffStep 2.5 noTitle=true)

##### Changed client&#x2F;main.css
```diff
@@ -1,4 +1,125 @@
+┊   ┊  1┊/* CSS declarations go here */
+┊   ┊  2┊
 ┊  1┊  3┊body {
-┊  2┊   ┊  padding: 10px;
 ┊  3┊  4┊  font-family: sans-serif;
+┊   ┊  5┊  background-color: #315481;
+┊   ┊  6┊  background-image: linear-gradient(to bottom, #315481, #918e82 100%);
+┊   ┊  7┊  background-attachment: fixed;
+┊   ┊  8┊  position: absolute;
+┊   ┊  9┊  top: 0;
+┊   ┊ 10┊  bottom: 0;
+┊   ┊ 11┊  left: 0;
+┊   ┊ 12┊  right: 0;
+┊   ┊ 13┊  padding: 0;
+┊   ┊ 14┊  margin: 0;
+┊   ┊ 15┊  font-size: 14px;
+┊   ┊ 16┊}
+┊   ┊ 17┊
+┊   ┊ 18┊.container {
+┊   ┊ 19┊  max-width: 600px;
+┊   ┊ 20┊  margin: 0 auto;
+┊   ┊ 21┊  min-height: 100%;
+┊   ┊ 22┊  background: white;
+┊   ┊ 23┊}
+┊   ┊ 24┊
+┊   ┊ 25┊header {
+┊   ┊ 26┊  background: #d2edf4;
+┊   ┊ 27┊  background-image: linear-gradient(to bottom, #d0edf5, #e1e5f0 100%);
+┊   ┊ 28┊  padding: 20px 15px 15px 15px;
+┊   ┊ 29┊  position: relative;
+┊   ┊ 30┊}
+┊   ┊ 31┊
+┊   ┊ 32┊#login-buttons {
+┊   ┊ 33┊  display: block;
+┊   ┊ 34┊}
+┊   ┊ 35┊
+┊   ┊ 36┊h1 {
+┊   ┊ 37┊  font-size: 1.5em;
+┊   ┊ 38┊  margin: 0;
+┊   ┊ 39┊  margin-bottom: 10px;
+┊   ┊ 40┊  display: inline-block;
+┊   ┊ 41┊  margin-right: 1em;
+┊   ┊ 42┊}
+┊   ┊ 43┊
+┊   ┊ 44┊form {
+┊   ┊ 45┊  margin-top: 10px;
+┊   ┊ 46┊  margin-bottom: -10px;
+┊   ┊ 47┊  position: relative;
+┊   ┊ 48┊}
+┊   ┊ 49┊
+┊   ┊ 50┊.new-task input {
+┊   ┊ 51┊  box-sizing: border-box;
+┊   ┊ 52┊  padding: 10px 0;
+┊   ┊ 53┊  background: transparent;
+┊   ┊ 54┊  border: none;
+┊   ┊ 55┊  width: 100%;
+┊   ┊ 56┊  padding-right: 80px;
+┊   ┊ 57┊  font-size: 1em;
+┊   ┊ 58┊}
+┊   ┊ 59┊
+┊   ┊ 60┊.new-task input:focus {
+┊   ┊ 61┊  outline: 0;
+┊   ┊ 62┊}
+┊   ┊ 63┊
+┊   ┊ 64┊ul {
+┊   ┊ 65┊  margin: 0;
+┊   ┊ 66┊  padding: 0;
+┊   ┊ 67┊  background: white;
+┊   ┊ 68┊}
+┊   ┊ 69┊
+┊   ┊ 70┊.delete {
+┊   ┊ 71┊  float: right;
+┊   ┊ 72┊  font-weight: bold;
+┊   ┊ 73┊  background: none;
+┊   ┊ 74┊  font-size: 1em;
+┊   ┊ 75┊  border: none;
+┊   ┊ 76┊  position: relative;
+┊   ┊ 77┊}
+┊   ┊ 78┊
+┊   ┊ 79┊li {
+┊   ┊ 80┊  position: relative;
+┊   ┊ 81┊  list-style: none;
+┊   ┊ 82┊  padding: 15px;
+┊   ┊ 83┊  border-bottom: #eee solid 1px;
+┊   ┊ 84┊}
+┊   ┊ 85┊
+┊   ┊ 86┊li .text {
+┊   ┊ 87┊  margin-left: 10px;
+┊   ┊ 88┊}
+┊   ┊ 89┊
+┊   ┊ 90┊li.checked {
+┊   ┊ 91┊  color: #888;
+┊   ┊ 92┊}
+┊   ┊ 93┊
+┊   ┊ 94┊li.checked .text {
+┊   ┊ 95┊  text-decoration: line-through;
+┊   ┊ 96┊}
+┊   ┊ 97┊
+┊   ┊ 98┊li.private {
+┊   ┊ 99┊  background: #eee;
+┊   ┊100┊
+┊   ┊101┊  border-color: #ddd;
+┊   ┊102┊}
+┊   ┊103┊
+┊   ┊104┊header .hide-completed {
+┊   ┊105┊  float: right;
+┊   ┊106┊}
+┊   ┊107┊
+┊   ┊108┊.toggle-private {
+┊   ┊109┊  margin-left: 5px;
+┊   ┊110┊}
+┊   ┊111┊
+┊   ┊112┊@media (max-width: 600px) {
+┊   ┊113┊  li {
+┊   ┊114┊    padding: 12px 15px;
+┊   ┊115┊  }
+┊   ┊116┊
+┊   ┊117┊  .search {
+┊   ┊118┊    width: 150px;
+┊   ┊119┊    clear: both;
+┊   ┊120┊  }
+┊   ┊121┊
+┊   ┊122┊  .new-task input {
+┊   ┊123┊    padding-bottom: 5px;
+┊   ┊124┊  }
 ┊  4┊125┊}
```

[}]: #


[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](step1.md) | [Next Step >](step3.md) |
|:--------------------------------|--------------------------------:|

[}]: #
