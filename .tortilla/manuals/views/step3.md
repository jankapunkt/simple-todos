# Step 3: Storing tasks in a collection

[//]: # (head-end)


Collections are Meteor's way of storing persistent data. The special thing about collections in Meteor is that they can be accessed from both the server and the client, making it easy to write view logic without having to write a lot of server code. They also update themselves automatically, so a view component backed by a collection will automatically display the most up-to-date data.

You can read more about collections in the [Collections article](http://guide.meteor.com/collections.html) of the Meteor Guide.

Creating a new collection is as easy as calling `new Mongo.Collection("my-collection")` in your JavaScript. On the server, this sets up a MongoDB collection called `my-collection`; on the client, it creates a "minimongo" collection connected to the server collection. Think of it as a local light version of a MongoDB collection that acts as a cache on the client. We'll learn more about the client/server divide in step 12, but for now we can write our code with the assumption that the entire database is present on the client.

To create the collection, we define a new **`imports/api/tasks.js`** module that creates a Mongo collection and exports it:

[{]: <helper> (diffStep 3.1 noTitle=true)

##### Added imports&#x2F;api&#x2F;tasks.js
```diff
@@ -0,0 +1,3 @@
+┊ ┊1┊import { Mongo } from 'meteor/mongo';
+┊ ┊2┊
+┊ ┊3┊export const Tasks = new Mongo.Collection('tasks');
```

[}]: #

Notice that we place this file in a new `imports/api` directory. This is a sensible place to store API-related files for the application. We will start by putting "collections" here and later we will add "publications" that read from them and "methods" that write to them. You can read more about how to structure your code in the [Application Structure article](http://guide.meteor.com/structure.html) of the Meteor Guide.

We now need to import the `imports/api/tasks.js` module into `server/main.js`:

[{]: <helper> (diffStep 3.2 noTitle=true)

##### Changed server&#x2F;main.js
```diff
@@ -1,5 +1 @@
-┊1┊ ┊import { Meteor } from 'meteor/meteor';
-┊2┊ ┊
-┊3┊ ┊Meteor.startup(() => {
-┊4┊ ┊  // code to run on server at startup
-┊5┊ ┊});
+┊ ┊1┊import '../imports/api/tasks.js';🚫↵
```

[}]: #

Importing imports/api/tasks.js on the server creates the MongoDB collection and sets up the plumbing to get the data to the client.

Let's update our client-side JavaScript code to get our tasks from a collection instead of a static array:

[{]: <helper> (diffStep 3.3 noTitle=true)

##### Changed imports&#x2F;ui&#x2F;body.js
```diff
@@ -1,11 +1,11 @@
 ┊ 1┊ 1┊import { Template } from 'meteor/templating';
 ┊ 2┊ 2┊
+┊  ┊ 3┊import { Tasks } from '../api/tasks.js';
+┊  ┊ 4┊
 ┊ 3┊ 5┊import './body.html';
 ┊ 4┊ 6┊
 ┊ 5┊ 7┊Template.body.helpers({
-┊ 6┊  ┊  tasks: [
-┊ 7┊  ┊    { text: 'This is task 1' },
-┊ 8┊  ┊    { text: 'This is task 2' },
-┊ 9┊  ┊    { text: 'This is task 3' }
-┊10┊  ┊  ]
+┊  ┊ 8┊  tasks() {
+┊  ┊ 9┊    return Tasks.find({});
+┊  ┊10┊  },
 ┊11┊11┊});
```

[}]: #

When you make these changes to the code, you'll notice that the tasks that used to be in the todo list have disappeared. That's because our database is currently empty — we need to insert some tasks!

### Inserting tasks from the server-side database console

Items inside collections are called _documents_. Let's use the server database console to insert some documents into our collection. While your app is running, in a new terminal tab, go to your app directory and type:

```bash
meteor mongo
```

This opens a console into your app's local development database. Into the prompt, type:

```js
db.tasks.insert({ text: "Hello world!", createdAt: new Date() });
```

In your web browser, you will see the UI of your app immediately update to show the new task. You can see that we didn't have to write any code to connect the server-side database to our front-end code—it just happened automatically.

Insert a few more tasks from the database console with different text. In the next step, we'll see how to add functionality to our app's UI so that we can add tasks without using the database console.

[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](step2.md) | [Next Step >](step4.md) |
|:--------------------------------|--------------------------------:|

[}]: #
