# Step 4: Adding tasks with a form

[//]: # (head-end)


In this step, we'll add an input field for users to add tasks to the list.

First, let's add a form to our HTML:

[{]: <helper> (diffStep 4.1 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.html
```diff
@@ -2,11 +2,17 @@
 ┊ 2┊ 2┊  <div class="container">
 ┊ 3┊ 3┊    <header>
 ┊ 4┊ 4┊      <h1>Todo List</h1>
+┊  ┊ 5┊
+┊  ┊ 6┊      <form class="new-task">
+┊  ┊ 7┊        <input type="text" name="text" placeholder="Type to add new tasks" />
+┊  ┊ 8┊      </form>
+┊  ┊ 9┊
 ┊ 5┊10┊    </header>
 ┊ 6┊11┊
 ┊ 7┊12┊    <ul>
 ┊ 8┊13┊      {{#each tasks}} {{> task}} {{/each}}
 ┊ 9┊14┊    </ul>
+┊  ┊15┊
 ┊10┊16┊  </div>
 ┊11┊17┊</body>
```

[}]: #

Here's the JavaScript code we need to add to listen to the `submit` event on the form:

[{]: <helper> (diffStep 4.2 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.js
```diff
@@ -9,3 +9,23 @@
 ┊ 9┊ 9┊    return Tasks.find({});
 ┊10┊10┊  },
 ┊11┊11┊});
+┊  ┊12┊
+┊  ┊13┊Template.body.events({
+┊  ┊14┊  'submit .new-task'(event) {
+┊  ┊15┊    // Prevent default browser form submit
+┊  ┊16┊    event.preventDefault();
+┊  ┊17┊
+┊  ┊18┊    // Get value from form element
+┊  ┊19┊    const target = event.target;
+┊  ┊20┊    const text = target.text.value;
+┊  ┊21┊
+┊  ┊22┊    // Insert a task into the collection
+┊  ┊23┊    Tasks.insert({
+┊  ┊24┊      text,
+┊  ┊25┊      createdAt: new Date(), // current time
+┊  ┊26┊    });
+┊  ┊27┊
+┊  ┊28┊    // Clear form
+┊  ┊29┊    target.text.value = '';
+┊  ┊30┊  },
+┊  ┊31┊});
```

[}]: #

Now your app has a new input field. To add a task, just type into the input field and hit enter. If you open a new browser window and open the app again, you'll see that the list is automatically synchronized between all clients.

### Attaching events to templates

Event listeners are added to templates in much the same way as helpers are: by calling `Template.templateName.events(...)` with a dictionary. The keys describe the event to listen for, and the values are _event handlers_ that are called when the event happens.

In our case above, we are listening to the `submit` event on any element that matches the CSS selector `.new-task`. When this event is triggered by the user pressing enter inside the input field, our event handler function is called.

The event handler gets an argument called `event` that has some information about the event that was triggered. In this case `event.target` is our form element, and we can get the value of our input with `event.target.text.value`. You can see all of the other properties of the `event` object by adding a `console.log(event)` and inspecting the object in your browser console.

Finally, in the last line of the event handler, we clear the input to prepare for another new task.

If you want to find out more about Template events you may consult the [Blaze documentation on Event Maps](http://blazejs.org/api/templates.html#Event-Maps).

### Inserting into a collection

Inside the event handler, we are adding a task to the `tasks` collection by calling `Tasks.insert()`. We can assign any properties to the task object, such as the time created, since we don't ever have to define a schema for the collection.

Being able to insert anything into the database from the client isn't secure at all, but it's okay for now. In step 10 we'll learn how we can make our app secure and restrict how data is inserted into the database.

### Sorting our tasks

Currently, our code displays all new tasks at the bottom of the list. That's not very good for a task list, because we want to see the newest tasks first.

We can solve this by sorting the results using the `createdAt` field that is automatically added by our new code. Just add a sort option to the `find` call inside the `tasks` helper:

[{]: <helper> (diffStep 4.3 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.js
```diff
@@ -6,8 +6,9 @@
 ┊ 6┊ 6┊
 ┊ 7┊ 7┊Template.body.helpers({
 ┊ 8┊ 8┊  tasks() {
-┊ 9┊  ┊    return Tasks.find({});
-┊10┊  ┊  },
+┊  ┊ 9┊    // Show newest tasks at the top
+┊  ┊10┊    return Tasks.find({}, { sort: { createdAt: -1 } });
+┊  ┊11┊  }
 ┊11┊12┊});
 ┊12┊13┊
 ┊13┊14┊Template.body.events({
```
```diff
@@ -22,10 +23,10 @@
 ┊22┊23┊    // Insert a task into the collection
 ┊23┊24┊    Tasks.insert({
 ┊24┊25┊      text,
-┊25┊  ┊      createdAt: new Date(), // current time
+┊  ┊26┊      createdAt: new Date() // current time
 ┊26┊27┊    });
 ┊27┊28┊
 ┊28┊29┊    // Clear form
 ┊29┊30┊    target.text.value = '';
-┊30┊  ┊  },
+┊  ┊31┊  }
 ┊31┊32┊});
```

[}]: #

In the next step, we'll add some very important todo list functions: checking off and deleting tasks.


[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](step3.md) | [Next Step >](step5.md) |
|:--------------------------------|--------------------------------:|

[}]: #
