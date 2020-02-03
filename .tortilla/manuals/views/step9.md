# Step 9: Security with methods

[//]: # (head-end)


Before this step, any user of the app could edit any part of the database. This might be okay for very small internal apps or demos, but any real application needs to control permissions for its data. In Meteor, the best way to do this is by declaring _methods_. Instead of the client code directly calling `insert`, `update`, and `remove`, it will instead call methods that will check if the user is authorized to complete the action and then make any changes to the database on the client's behalf.

### Removing `insecure`

Every newly created Meteor project has the `insecure` package added by default. This is the package that allows us to edit the database from the client. It's useful when prototyping, but now we are taking off the training wheels. To remove this package, go to your app directory and run:

```bash
meteor remove insecure
```

If you try to use the app after removing this package, you will notice that none of the inputs or buttons work anymore. This is because all client-side database permissions have been revoked. Now we need to rewrite some parts of our app to use methods.

### Defining methods

First, we need to define some methods. We need one method for each database operation we want to perform on the client. Methods should be defined in code that is executed on the client and the server - we will discuss this a bit later in the section titled _Optimistic UI_.

[{]: <helper> (diffStep 9.2 noTitle=true)

##### Changed imports&#x2F;api&#x2F;tasks.js
```diff
@@ -1,3 +1,39 @@
+┊  ┊ 1┊import { Meteor } from 'meteor/meteor';
+┊  ┊ 2┊
 ┊ 1┊ 3┊import { Mongo } from 'meteor/mongo';
 ┊ 2┊ 4┊
+┊  ┊ 5┊import { check } from 'meteor/check';
+┊  ┊ 6┊
 ┊ 3┊ 7┊export const Tasks = new Mongo.Collection('tasks');
+┊  ┊ 8┊
+┊  ┊ 9┊Meteor.methods({
+┊  ┊10┊  'tasks.insert'(text) {
+┊  ┊11┊    check(text, String);
+┊  ┊12┊
+┊  ┊13┊    // Make sure the user is logged in before inserting a task
+┊  ┊14┊
+┊  ┊15┊    if (!Meteor.userId()) {
+┊  ┊16┊      throw new Meteor.Error('not-authorized');
+┊  ┊17┊    }
+┊  ┊18┊
+┊  ┊19┊    Tasks.insert({
+┊  ┊20┊      text,
+┊  ┊21┊      createdAt: new Date(),
+┊  ┊22┊      owner: Meteor.userId(),
+┊  ┊23┊      username: Meteor.user().username
+┊  ┊24┊    });
+┊  ┊25┊  },
+┊  ┊26┊
+┊  ┊27┊  'tasks.remove'(taskId) {
+┊  ┊28┊    check(taskId, String);
+┊  ┊29┊
+┊  ┊30┊    Tasks.remove(taskId);
+┊  ┊31┊  },
+┊  ┊32┊
+┊  ┊33┊  'tasks.setChecked'(taskId, setChecked) {
+┊  ┊34┊    check(taskId, String);
+┊  ┊35┊    check(setChecked, Boolean);
+┊  ┊36┊
+┊  ┊37┊    Tasks.update(taskId, { $set: { checked: setChecked } });
+┊  ┊38┊  }
+┊  ┊39┊});
```

[}]: #

Since methods are not restricted to manipulation of MongoDB collection data but also  allow to call external APIs or execute process-level code it is highly recommended to read the [article on methods security](https://guide.meteor.com/security.html#methods).
Now that we have defined our methods, we need to update the places we were operating on the collection to use the methods instead:

[{]: <helper> (diffStep 9.3 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.js
```diff
@@ -36,12 +36,7 @@
 ┊36┊36┊    const text = target.text.value;
 ┊37┊37┊
 ┊38┊38┊    // Insert a task into the collection
-┊39┊  ┊    Tasks.insert({
-┊40┊  ┊      text,
-┊41┊  ┊      createdAt: new Date(), // current time
-┊42┊  ┊      owner: Meteor.userId(),
-┊43┊  ┊      username: Meteor.user().username,
-┊44┊  ┊    });
+┊  ┊39┊    Meteor.call('tasks.insert', text);
 ┊45┊40┊
 ┊46┊41┊    // Clear form
 ┊47┊42┊    target.text.value = '';
```

[}]: #

We do the same on this file, but also remove the `import` of the `Tasks` collection since it's no longer necessary:

[{]: <helper> (diffStep 9.4 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;task.js
```diff
@@ -1,3 +1,4 @@
+┊ ┊1┊import { Meteor } from 'meteor/meteor';
 ┊1┊2┊import { Template } from 'meteor/templating';
 ┊2┊3┊
 ┊3┊4┊import { Tasks } from '../api/tasks.js';
```
```diff
@@ -7,11 +8,9 @@
 ┊ 7┊ 8┊Template.task.events({
 ┊ 8┊ 9┊  'click .toggle-checked'() {
 ┊ 9┊10┊    // Set the checked property to the opposite of its current value
-┊10┊  ┊    Tasks.update(this._id, {
-┊11┊  ┊      $set: { checked: !this.checked }
-┊12┊  ┊    });
+┊  ┊11┊    Meteor.call('tasks.setChecked', this._id, !this.checked);
 ┊13┊12┊  },
 ┊14┊13┊  'click .delete'() {
-┊15┊  ┊    Tasks.remove(this._id);
+┊  ┊14┊    Meteor.call('tasks.remove', this._id);
 ┊16┊15┊  }
 ┊17┊16┊});
```

[}]: #

Now all of our inputs and buttons will start working again. What did we gain from all of this work?

1. When we insert tasks into the database, we can now securely verify that the user is logged in, that the `createdAt` field is correct, and that the `owner` and `username` fields are correct and the user isn't impersonating anyone.
2. We can add extra validation logic to `setChecked` and `remove` in later steps when users can make tasks private.
3. Our client code is now more separated from our database logic. Instead of a lot of stuff happening inside our event handlers, we now have methods that can be called from anywhere.



[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](step8.md) | [Next Step >](step10.md) |
|:--------------------------------|--------------------------------:|

[}]: #
