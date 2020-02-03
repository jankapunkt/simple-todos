# Step 8: Adding user accounts

[//]: # (head-end)


Meteor comes with an accounts system and a drop-in login user interface that lets you add multi-user functionality to your app in minutes.

To enable the accounts system and UI, we need to add the relevant packages. In your app directory, run the following command:

```bash
meteor add accounts-ui accounts-password
```

In the HTML, right under the checkbox, include the following code to add a login dropdown:

[{]: <helper> (diffStep 8.2 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.html
```diff
@@ -7,6 +7,9 @@
 ┊ 7┊ 7┊        <input type="checkbox" />
 ┊ 8┊ 8┊        Hide Completed Tasks
 ┊ 9┊ 9┊      </label>
+┊  ┊10┊
+┊  ┊11┊      {{> loginButtons}}
+┊  ┊12┊
 ┊10┊13┊    </header>
 ┊11┊14┊
 ┊12┊15┊    <form class="new-task">
```

[}]: #

Then, in your JavaScript, add the following code to configure the accounts UI to use usernames instead of email addresses:

[{]: <helper> (diffStep 8.3 noTitle=true)

##### Added imports&#x2F;startup&#x2F;accounts-config.js
```diff
@@ -0,0 +1,5 @@
+┊ ┊1┊import { Accounts } from 'meteor/accounts-base';
+┊ ┊2┊
+┊ ┊3┊Accounts.ui.config({
+┊ ┊4┊  passwordSignupFields: 'USERNAME_ONLY'
+┊ ┊5┊});
```

[}]: #

We'll need to import that configuration from our *client-side JavaScript entrypoint* also:

[{]: <helper> (diffStep 8.4 noTitle=true)

##### Changed client&#x2F;main.js
```diff
@@ -1 +1,2 @@
+┊ ┊1┊import '../imports/startup/accounts-config.js';
 ┊1┊2┊import '../imports/ui/body.js';🚫↵
```

[}]: #

Now users can create accounts and log into your app! This is very nice, but logging in and out isn't very useful yet. Let's add two functions:

1. Only display the new task input field to logged in users
2. Show which user created each task

To do this, we will add two new fields to the `tasks` collection:

1. `owner` - the `_id` of the user that created the task.
2. `username` - the `username` of the user that created the task. We will save the username directly in the task object so that we don't have to look up the user every time we display the task.

First, let's add some code to save these fields into the `submit .new-task` event handler:

[{]: <helper> (diffStep 8.5 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.js
```diff
@@ -1,3 +1,4 @@
+┊ ┊1┊import { Meteor } from 'meteor/meteor';
 ┊1┊2┊import { Template } from 'meteor/templating';
 ┊2┊3┊import { ReactiveDict } from 'meteor/reactive-dict';
 ┊3┊4┊
```
```diff
@@ -37,7 +38,9 @@
 ┊37┊38┊    // Insert a task into the collection
 ┊38┊39┊    Tasks.insert({
 ┊39┊40┊      text,
-┊40┊  ┊      createdAt: new Date() // current time
+┊  ┊41┊      createdAt: new Date(), // current time
+┊  ┊42┊      owner: Meteor.userId(),
+┊  ┊43┊      username: Meteor.user().username,
 ┊41┊44┊    });
 ┊42┊45┊
 ┊43┊46┊    // Clear form
```

[}]: #

Then, in our HTML, add an `#if` block helper to only show the form when there is a logged in user:

[{]: <helper> (diffStep 8.6 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.html
```diff
@@ -10,6 +10,11 @@
 ┊10┊10┊
 ┊11┊11┊      {{> loginButtons}}
 ┊12┊12┊
+┊  ┊13┊      {{#if currentUser}}
+┊  ┊14┊        <form class="new-task">
+┊  ┊15┊          <input type="text" name="text" placeholder="Type to add new tasks" />
+┊  ┊16┊        </form>
+┊  ┊17┊    {{/if}}
 ┊13┊18┊    </header>
 ┊14┊19┊
 ┊15┊20┊    <form class="new-task">
```

[}]: #

Finally, add a Spacebars statement to display the `username` field on each task right before the text:

[{]: <helper> (diffStep 8.7 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;task.html
```diff
@@ -4,7 +4,7 @@
 ┊4┊4┊
 ┊5┊5┊      <input type="checkbox" checked="{{checked}}" class="toggle-checked" />
 ┊6┊6┊
-┊7┊ ┊      <span class="text">{{text}}</span>
+┊ ┊7┊      <span class="text"><strong>{{username}}</strong> - {{text}}</span>
 ┊8┊8┊    </li>
 ┊9┊9┊  </template>
```

[}]: #

Now, users can log in and we can track which user each task belongs to. Let's look at some of the concepts we just discovered in more detail.

### Automatic accounts UI

If our app has the `accounts-ui` package, all we have to do to add a login dropdown is include the `loginButtons` template with `> loginButtons}}`. This dropdown detects which login methods have been added to the app and displays the appropriate controls. In our case, the only enabled login method is `accounts-password`, so the dropdown displays a password field. If you are adventurous, you can add the `accounts-facebook` package to enable Facebook login in your app - the Facebook button will automatically appear in the dropdown.

### Getting information about the logged-in user

In your HTML, you can use the built-in `currentUser}}` helper to check if a user is logged in and get information about them. For example, `currentUser.username}}` will display the logged in user's username.

In your JavaScript code, you can use `Meteor.userId()` to get the current user's `_id`, or `Meteor.user()` to get the whole user document.

In the next step, we will learn how to make our app more secure by doing all of our data validation on the server instead of the client.

You can read more about using accounts in Meteor in the [Accounts article](http://guide.meteor.com/accounts.html) of the Meteor Guide.

[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](step7.md) | [Next Step >](step9.md) |
|:--------------------------------|--------------------------------:|

[}]: #
