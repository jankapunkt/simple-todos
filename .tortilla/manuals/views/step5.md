# Step 5: Checking off and deleting tasks

[//]: # (head-end)


Until now, we have only interacted with a collection by inserting documents. Now, we will learn how to update and remove them.

Let's work on our `task` template---starting by moving it to its own file, with some new features, a checkbox and a delete button:

[{]: <helper> (diffStep 5.1 noTitle=true)

##### Added imports&#x2F;ui&#x2F;task.html
```diff
@@ -0,0 +1,11 @@
+┊  ┊ 1┊<template name="task">
+┊  ┊ 2┊    <li class="{{#if checked}}checked{{/if}}">
+┊  ┊ 3┊      <button class="delete">&times;</button>
+┊  ┊ 4┊
+┊  ┊ 5┊      <input type="checkbox" checked="{{checked}}" class="toggle-checked" />
+┊  ┊ 6┊
+┊  ┊ 7┊      <span class="text">{{text}}</span>
+┊  ┊ 8┊    </li>
+┊  ┊ 9┊  </template>
+┊  ┊10┊
+┊  ┊11┊  🚫↵
```

[}]: #

**5.2** Remove the previous `task` template

Since we've added a new `task` template here, we need to remove the old `task` template we had before.  Open the `imports/ui/body.html` file and remove the entire `<template name="task">...</template>` section from the end.

Now we've added UI elements, but they don't do anything yet! We should add some event handlers:

[{]: <helper> (diffStep 5.2 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.html
```diff
@@ -14,8 +14,4 @@
 ┊14┊14┊    </ul>
 ┊15┊15┊
 ┊16┊16┊  </div>
-┊17┊  ┊</body>
-┊18┊  ┊
-┊19┊  ┊<template name="task">
-┊20┊  ┊  <li>{{text}}</li>
-┊21┊  ┊</template>
+┊  ┊17┊</body>🚫↵
```

##### Added imports&#x2F;ui&#x2F;task.js
```diff
@@ -0,0 +1,17 @@
+┊  ┊ 1┊import { Template } from 'meteor/templating';
+┊  ┊ 2┊
+┊  ┊ 3┊import { Tasks } from '../api/tasks.js';
+┊  ┊ 4┊
+┊  ┊ 5┊import './task.html';
+┊  ┊ 6┊
+┊  ┊ 7┊Template.task.events({
+┊  ┊ 8┊  'click .toggle-checked'() {
+┊  ┊ 9┊    // Set the checked property to the opposite of its current value
+┊  ┊10┊    Tasks.update(this._id, {
+┊  ┊11┊      $set: { checked: !this.checked }
+┊  ┊12┊    });
+┊  ┊13┊  },
+┊  ┊14┊  'click .delete'() {
+┊  ┊15┊    Tasks.remove(this._id);
+┊  ┊16┊  }
+┊  ┊17┊});
```

[}]: #

The `body` template uses the `task` template, so we need to import it as well:

[{]: <helper> (diffStep 5.4 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.js
```diff
@@ -2,6 +2,7 @@
 ┊2┊2┊
 ┊3┊3┊import { Tasks } from '../api/tasks.js';
 ┊4┊4┊
+┊ ┊5┊import './task.js';
 ┊5┊6┊import './body.html';
 ┊6┊7┊
 ┊7┊8┊Template.body.helpers({
```

[}]: #


### Getting data in event handlers

Inside the event handlers, `this` refers to an individual task object. In a collection, every inserted document has a unique `_id` field that can be used to refer to that specific document. We can get the `_id` of the current task with `this._id`. Once we have the `_id`, we can use `update` and `remove` to modify the relevant task.

### Update

The `update` function on a collection takes two arguments. The first is a selector that identifies a subset of the collection, and the second is an update parameter that specifies what should be done to the matched objects.

In this case, the selector is just the `_id` of the relevant task. The update parameter uses `$set` to toggle the `checked` field, which will represent whether the task has been completed.

### Remove

The `remove` function takes one argument, a selector that determines which item to remove from the collection.

### Using object properties or helpers to add/remove classes

If you try checking off some tasks after adding all of the above code, you will see that checked off tasks have a line through them. This is enabled by the following snippet:

```html
<li class="#if checked}}checked/if}}">
```

With this code, if the `checked` property of a task is `true`, the `checked` class is added to our list item. Using this class, we can make checked-off tasks look different in our CSS.


[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](step4.md) | [Next Step >](step6.md) |
|:--------------------------------|--------------------------------:|

[}]: #
