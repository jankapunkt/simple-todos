# Step 7: Storing temporary UI state in a Reactive Dictionary

[//]: # (head-end)


In this step, we'll add a client-side data filtering feature to our app, so that users can check a box to only see incomplete tasks. We're going to learn how to use a [`ReactiveDict`](https://atmospherejs.com/meteor/reactive-dict) to store temporary reactive state on the client. A `ReactiveDict` is like a normal JS object with keys and values, but with built-in reactivity.

First, we need to add a checkbox to our HTML:

[{]: <helper> (diffStep 7.1 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.html
```diff
@@ -2,6 +2,10 @@
 ┊ 2┊ 2┊  <div class="container">
 ┊ 3┊ 3┊    <header>
 ┊ 4┊ 4┊      <h1>Todo List</h1>
+┊  ┊ 5┊      <label class="hide-completed">
+┊  ┊ 6┊        <input type="checkbox" />
+┊  ┊ 7┊        Hide Completed Tasks
+┊  ┊ 8┊      </label>
 ┊ 5┊ 9┊    </header>
 ┊ 6┊10┊
 ┊ 7┊11┊    <form class="new-task">
```

[}]: #

Now we need to add the `reactive-dict` package:

```bash
meteor add reactive-dict
```

Then we need to set up a new `ReactiveDict` and attach it to the body template instance (as this is where we'll store the checkbox's state) when it is first created:

[{]: <helper> (diffStep 7.3 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.js
```diff
@@ -1,10 +1,15 @@
 ┊ 1┊ 1┊import { Template } from 'meteor/templating';
+┊  ┊ 2┊import { ReactiveDict } from 'meteor/reactive-dict';
 ┊ 2┊ 3┊
 ┊ 3┊ 4┊import { Tasks } from '../api/tasks.js';
 ┊ 4┊ 5┊
 ┊ 5┊ 6┊import './task.js';
 ┊ 6┊ 7┊import './body.html';
 ┊ 7┊ 8┊
+┊  ┊ 9┊Template.body.onCreated(function bodyOnCreated() {
+┊  ┊10┊  this.state = new ReactiveDict();
+┊  ┊11┊});
+┊  ┊12┊
 ┊ 8┊13┊Template.body.helpers({
 ┊ 9┊14┊  tasks() {
 ┊10┊15┊    // Show newest tasks at the top
```

[}]: #

Then, we need an event handler to update the `ReactiveDict` variable when the checkbox
is checked or unchecked. An event handler takes two arguments, the second of which is the same template instance which was `this` in the `onCreated` callback:

[{]: <helper> (diffStep 7.4 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.js
```diff
@@ -34,5 +34,8 @@
 ┊34┊34┊
 ┊35┊35┊    // Clear form
 ┊36┊36┊    target.text.value = '';
-┊37┊  ┊  }
+┊  ┊37┊  },
+┊  ┊38┊  'change .hide-completed input'(event, instance) {
+┊  ┊39┊    instance.state.set('hideCompleted', event.target.checked);
+┊  ┊40┊  },
 ┊38┊41┊});
```

[}]: #

Now, we need to update `Template.body.helpers`. The code below has a new if
block to filter the tasks if the checkbox is checked:

[{]: <helper> (diffStep 7.5 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.js
```diff
@@ -12,9 +12,14 @@
 ┊12┊12┊
 ┊13┊13┊Template.body.helpers({
 ┊14┊14┊  tasks() {
-┊15┊  ┊    // Show newest tasks at the top
+┊  ┊15┊    const instance = Template.instance();
+┊  ┊16┊    if (instance.state.get('hideCompleted')) {
+┊  ┊17┊      // If hide completed is checked, filter tasks
+┊  ┊18┊      return Tasks.find({ checked: { $ne: true } }, { sort: { createdAt: -1 } });
+┊  ┊19┊    }
+┊  ┊20┊    // Otherwise, return all of the tasks
 ┊16┊21┊    return Tasks.find({}, { sort: { createdAt: -1 } });
-┊17┊  ┊  }
+┊  ┊22┊  },
 ┊18┊23┊});
 ┊19┊24┊
 ┊20┊25┊Template.body.events({
```

[}]: #

Now if you check the box, the task list will only show tasks that haven't been completed.

### ReactiveDicts are reactive data stores for the client

Until now, we have stored all of our state in collections, and the view updated automatically when we modified the data inside these collections. This is because Mongo.Collection is recognized by Meteor as a _reactive data source_, meaning Meteor knows when the data inside has changed. `ReactiveDict` is the same way, but is not synced with the server like collections are. This makes a `ReactiveDict` a convenient place to store temporary UI state like the checkbox above. Just like with collections, we don't have to write any extra code for the template to update when the `ReactiveDict` variable changes &mdash; just calling `instance.state.get(...)` inside the helper is enough.

You can read more about patterns for writing components in the Blaze documentation on [writing reusable components](http://blazejs.org/guide/reusable-components.html) and on [writing smart components](http://blazejs.org/guide/smart-components.html).


### One more feature: Showing a count of incomplete tasks

Now that we have written a query that filters out completed tasks, we can use the same query to display a count of the tasks that haven't been checked off. To do this we need to add a helper and change one line of the HTML.

[{]: <helper> (diffStep 7.6 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.js
```diff
@@ -20,6 +20,9 @@
 ┊20┊20┊    // Otherwise, return all of the tasks
 ┊21┊21┊    return Tasks.find({}, { sort: { createdAt: -1 } });
 ┊22┊22┊  },
+┊  ┊23┊  incompleteCount() {
+┊  ┊24┊    return Tasks.find({ checked: { $ne: true } }).count();
+┊  ┊25┊  },
 ┊23┊26┊});
 ┊24┊27┊
 ┊25┊28┊Template.body.events({
```

[}]: #

[{]: <helper> (diffStep 7.7 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.html
```diff
@@ -1,7 +1,8 @@
 ┊1┊1┊<body>
 ┊2┊2┊  <div class="container">
 ┊3┊3┊    <header>
-┊4┊ ┊      <h1>Todo List</h1>
+┊ ┊4┊      <h1>Todo List ({{incompleteCount}})</h1>
+┊ ┊5┊
 ┊5┊6┊      <label class="hide-completed">
 ┊6┊7┊        <input type="checkbox" />
 ┊7┊8┊        Hide Completed Tasks
```

[}]: #

[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](step6.md) | [Next Step >](step8.md) |
|:--------------------------------|--------------------------------:|

[}]: #
