# Step 10: Filtering data with publish and subscribe

[//]: # (head-end)


Now that we have moved all of our app's sensitive code into methods, we need to learn about the other half of Meteor's security story. Until now, we have worked assuming the entire database is present on the client, meaning if we call `Tasks.find()` we will get every task in the collection. That's not good if users of our application want to store privacy-sensitive data. We need a way of controlling which data Meteor sends to the client-side database.

Just like with `insecure` in the last step, all new Meteor apps start with the `autopublish` package. Let's remove it and see what happens:

```bash
meteor remove autopublish
```

When the app refreshes, the task list will be empty. Without the `autopublish` package, we will have to specify explicitly what the server sends to the client. The functions in Meteor that do this are `Meteor.publish` and `Meteor.subscribe`.

First lets add a publication for all tasks:

[{]: <helper> (diffStep 10.2 noTitle=true)

##### Changed imports&#x2F;api&#x2F;tasks.js
```diff
@@ -6,6 +6,13 @@
 ┊ 6┊ 6┊
 ┊ 7┊ 7┊export const Tasks = new Mongo.Collection('tasks');
 ┊ 8┊ 8┊
+┊  ┊ 9┊if (Meteor.isServer) {
+┊  ┊10┊  // This code only runs on the server
+┊  ┊11┊  Meteor.publish('tasks', function tasksPublication() {
+┊  ┊12┊    return Tasks.find();
+┊  ┊13┊  });
+┊  ┊14┊}
+┊  ┊15┊
 ┊ 9┊16┊Meteor.methods({
 ┊10┊17┊  'tasks.insert'(text) {
 ┊11┊18┊    check(text, String);
```

[}]: #

And then let's subscribe to that publication when the `body` template is created:

[{]: <helper> (diffStep 10.3 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.js
```diff
@@ -9,6 +9,7 @@
 ┊ 9┊ 9┊
 ┊10┊10┊Template.body.onCreated(function bodyOnCreated() {
 ┊11┊11┊  this.state = new ReactiveDict();
+┊  ┊12┊  Meteor.subscribe('tasks');
 ┊12┊13┊});
 ┊13┊14┊
 ┊14┊15┊Template.body.helpers({
```

[}]: #

Once you have added this code, all of the tasks will reappear.

Calling `Meteor.publish` on the server registers a _publication_ named `"tasks"`. When `Meteor.subscribe` is called on the client with the publication name, the client _subscribes_ to all the data from that publication, which in this case is all of the tasks in the database. To truly see the power of the publish/subscribe model, let's implement a feature that allows users to mark tasks as "private" so that no other users can see them.

You can read more about publications in the [Data Loading article](http://guide.meteor.com/data-loading.html) of the Meteor Guide.

### Implementing private tasks

First, let's add another property to tasks called "private" and a button for users to mark a task as private. This button should only show up for the owner of a task. It will display the current state of the item.

[{]: <helper> (diffStep 10.4 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;task.html
```diff
@@ -4,6 +4,16 @@
 ┊ 4┊ 4┊
 ┊ 5┊ 5┊      <input type="checkbox" checked="{{checked}}" class="toggle-checked" />
 ┊ 6┊ 6┊
+┊  ┊ 7┊      {{#if isOwner}}
+┊  ┊ 8┊      <button class="toggle-private">
+┊  ┊ 9┊        {{#if private}}
+┊  ┊10┊          Private
+┊  ┊11┊        {{else}}
+┊  ┊12┊          Public
+┊  ┊13┊        {{/if}}
+┊  ┊14┊      </button>
+┊  ┊15┊    {{/if}}
+┊  ┊16┊
 ┊ 7┊17┊      <span class="text"><strong>{{username}}</strong> - {{text}}</span>
 ┊ 8┊18┊    </li>
 ┊ 9┊19┊  </template>
```

[}]: #

Let's make sure our task has a special class if it is marked private:

[{]: <helper> (diffStep 10.5 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;task.html
```diff
@@ -1,5 +1,5 @@
 ┊1┊1┊<template name="task">
-┊2┊ ┊    <li class="{{#if checked}}checked{{/if}}">
+┊ ┊2┊    <li class="{{#if checked}}checked{{/if}} {{#if private}}private{{/if}}">
 ┊3┊3┊      <button class="delete">&times;</button>
 ┊4┊4┊
 ┊5┊5┊      <input type="checkbox" checked="{{checked}}" class="toggle-checked" />
```

[}]: #

We need to modify our JavaScript code in three places:

[{]: <helper> (diffStep 10.6 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;task.js
```diff
@@ -5,6 +5,12 @@
 ┊ 5┊ 5┊
 ┊ 6┊ 6┊import './task.html';
 ┊ 7┊ 7┊
+┊  ┊ 8┊Template.task.helpers({
+┊  ┊ 9┊  isOwner() {
+┊  ┊10┊    return this.owner === Meteor.userId();
+┊  ┊11┊  }
+┊  ┊12┊});
+┊  ┊13┊
 ┊ 8┊14┊Template.task.events({
 ┊ 9┊15┊  'click .toggle-checked'() {
 ┊10┊16┊    // Set the checked property to the opposite of its current value
```

[}]: #

[{]: <helper> (diffStep 10.7 noTitle=true)

##### Changed imports&#x2F;api&#x2F;tasks.js
```diff
@@ -42,5 +42,18 @@
 ┊42┊42┊    check(setChecked, Boolean);
 ┊43┊43┊
 ┊44┊44┊    Tasks.update(taskId, { $set: { checked: setChecked } });
+┊  ┊45┊  },
+┊  ┊46┊  'tasks.setPrivate'(taskId, setToPrivate) {
+┊  ┊47┊    check(taskId, String);
+┊  ┊48┊    check(setToPrivate, Boolean);
+┊  ┊49┊
+┊  ┊50┊    const task = Tasks.findOne(taskId);
+┊  ┊51┊
+┊  ┊52┊    // Make sure only the task owner can make a task private
+┊  ┊53┊    if (task.owner !== Meteor.userId()) {
+┊  ┊54┊      throw new Meteor.Error('not-authorized');
+┊  ┊55┊    }
+┊  ┊56┊
+┊  ┊57┊    Tasks.update(taskId, { $set: { private: setToPrivate } });
 ┊45┊58┊  }
 ┊46┊59┊});
```

[}]: #

[{]: <helper> (diffStep 10.8 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;task.js
```diff
@@ -18,5 +18,8 @@
 ┊18┊18┊  },
 ┊19┊19┊  'click .delete'() {
 ┊20┊20┊    Meteor.call('tasks.remove', this._id);
-┊21┊  ┊  }
+┊  ┊21┊  },
+┊  ┊22┊  'click .toggle-private'() {
+┊  ┊23┊    Meteor.call('tasks.setPrivate', this._id, !this.private);
+┊  ┊24┊  },
 ┊22┊25┊});
```

[}]: #

### Selectively publishing tasks based on privacy status

Now that we have a way of setting which tasks are private, we should modify our
publication function to only send the tasks that a user is authorized to see:

[{]: <helper> (diffStep 10.9 noTitle=true)

##### Changed imports&#x2F;api&#x2F;tasks.js
```diff
@@ -8,8 +8,11 @@
 ┊ 8┊ 8┊
 ┊ 9┊ 9┊if (Meteor.isServer) {
 ┊10┊10┊  // This code only runs on the server
+┊  ┊11┊  // Only publish tasks that are public or belong to the current user
 ┊11┊12┊  Meteor.publish('tasks', function tasksPublication() {
-┊12┊  ┊    return Tasks.find();
+┊  ┊13┊    return Tasks.find({
+┊  ┊14┊      $or: [{ private: { $ne: true } }, { owner: this.userId }]
+┊  ┊15┊    });
 ┊13┊16┊  });
 ┊14┊17┊}
```

[}]: #

To test that this functionality works, you can use your browser's private browsing mode to log in as a different user. Put the two windows side by side and mark a task private to confirm that the other user can't see it. Now make it public again and it will reappear!

### Extra method security

In order to finish up our private task feature, we need to add checks to our `deleteTask` and `setChecked` methods to make sure only the task owner can delete or check off a private task:

[{]: <helper> (diffStep 10.1 noTitle=true)

##### Changed .meteor&#x2F;packages
```diff
@@ -19,7 +19,6 @@
 ┊19┊19┊typescript@3.7.2              # Enable TypeScript syntax in .ts and .tsx modules
 ┊20┊20┊shell-server@0.4.0            # Server-side component of the `meteor shell` command
 ┊21┊21┊
-┊22┊  ┊autopublish@1.0.7             # Publish all data to the clients (for prototyping)
 ┊23┊22┊reactive-dict
 ┊24┊23┊accounts-ui
 ┊25┊24┊accounts-password🚫↵
```

##### Changed .meteor&#x2F;versions
```diff
@@ -3,7 +3,6 @@
 ┊3┊3┊accounts-ui@1.3.1
 ┊4┊4┊accounts-ui-unstyled@1.4.2
 ┊5┊5┊allow-deny@1.1.0
-┊6┊ ┊autopublish@1.0.7
 ┊7┊6┊autoupdate@1.6.0
 ┊8┊7┊babel-compiler@7.5.1
 ┊9┊8┊babel-runtime@1.5.0
```

[}]: #

> Notice that with this code anyone can delete any public task. With some small modifications to the code, you should be able to make it so that only the owner can delete their tasks.

We're done with our private task feature! Now our app is secure from attackers trying to view or modify someone's private tasks.


[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](step9.md) | [Next Step >](step11.md) |
|:--------------------------------|--------------------------------:|

[}]: #
