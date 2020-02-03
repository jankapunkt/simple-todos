# Step 11: Testing

[//]: # (head-end)


Now that we've created a few features for our application, let's add a test to ensure that we don't regress and that it works the way we expect.

We'll write a test that exercises one of our Methods (which form the "write" part of our app's API), and verifies it works correctly.

To do so, we'll add a [test driver](http://guide.meteor.com/testing.html#test-driver) for the [Mocha](https://mochajs.org) JavaScript test framework, along with a test assertion library:

```bash
meteor add meteortesting:mocha
meteor npm install --save-dev chai
```

> **New in Meteor 1.7+**: While the `meteor test ...` command does still work as specified below, the release of Meteor 1.7 includes a new feature which allows you to specify the location of the test module in `package.json` with a property named `"testModule"` within the `"meteor"` object, which you can read more about [in the changelog](https://docs.meteor.com/changelog.html#v1720180528). In order to get the expected behavior, such that the `meteor test ...` command only uses files whose names match the format `*.test[s].*` or `*.spec[s].*`, you should either remove the line `"testModule": "tests/main.js"` from your `package.json` file, or change it to an appropriate value, before running `meteor test ...`.

We can now run our app in "test mode" by calling out a special command and specifying to use the driver (you'll need to stop the regular app from running, or specify an alternate port with `--port XYZ`):

```bash
TEST_WATCH=1 meteor test --driver-package meteortesting:mocha
```

If you do so, you should see an empty test results page in your browser window.

Let's add a simple test (that doesn't do anything yet):

[{]: <helper> (diffStep 11.2 files="imports/api/tasks.tests.js" noTitle=true)

##### Added imports&#x2F;api&#x2F;tasks.tests.js
```diff
@@ -0,0 +1,11 @@
+┊  ┊ 1┊/* eslint-env mocha */
+┊  ┊ 2┊
+┊  ┊ 3┊import { Meteor } from 'meteor/meteor';
+┊  ┊ 4┊
+┊  ┊ 5┊if (Meteor.isServer) {
+┊  ┊ 6┊  describe('Tasks', () => {
+┊  ┊ 7┊    describe('methods', () => {
+┊  ┊ 8┊      it('can delete owned task', () => {});
+┊  ┊ 9┊    });
+┊  ┊10┊  });
+┊  ┊11┊}
```

[}]: #

In any test we need to ensure the database is in the state we expect before beginning. We can use Mocha's `beforeEach` construct to do that easily:

[{]: <helper> (diffStep 11.3 noTitle=true)

##### Changed imports&#x2F;api&#x2F;tasks.tests.js
```diff
@@ -1,10 +1,28 @@
 ┊ 1┊ 1┊/* eslint-env mocha */
 ┊ 2┊ 2┊
 ┊ 3┊ 3┊import { Meteor } from 'meteor/meteor';
+┊  ┊ 4┊import { Random } from 'meteor/random';
+┊  ┊ 5┊
+┊  ┊ 6┊import { Tasks } from './tasks.js';
 ┊ 4┊ 7┊
 ┊ 5┊ 8┊if (Meteor.isServer) {
 ┊ 6┊ 9┊  describe('Tasks', () => {
 ┊ 7┊10┊    describe('methods', () => {
+┊  ┊11┊      const userId = Random.id();
+┊  ┊12┊
+┊  ┊13┊      let taskId;
+┊  ┊14┊
+┊  ┊15┊      beforeEach(() => {
+┊  ┊16┊        Tasks.remove({});
+┊  ┊17┊
+┊  ┊18┊        taskId = Tasks.insert({
+┊  ┊19┊          text: 'test task',
+┊  ┊20┊          createdAt: new Date(),
+┊  ┊21┊          owner: userId,
+┊  ┊22┊          username: 'tmeasday'
+┊  ┊23┊        });
+┊  ┊24┊      });
+┊  ┊25┊
 ┊ 8┊26┊      it('can delete owned task', () => {});
 ┊ 9┊27┊    });
 ┊10┊28┊  });
```

[}]: #

Here we create a single task that's associated with a random `userId` that'll be different for each test run.

Now we can write the test to call the `tasks.remove` method "as" that user and verify the task is deleted:

[{]: <helper> (diffStep 11.4 noTitle=true)

##### Changed imports&#x2F;api&#x2F;tasks.tests.js
```diff
@@ -2,6 +2,7 @@
 ┊2┊2┊
 ┊3┊3┊import { Meteor } from 'meteor/meteor';
 ┊4┊4┊import { Random } from 'meteor/random';
+┊ ┊5┊import { assert } from 'chai';
 ┊5┊6┊
 ┊6┊7┊import { Tasks } from './tasks.js';
 ┊7┊8┊
```
```diff
@@ -9,12 +10,10 @@
 ┊ 9┊10┊  describe('Tasks', () => {
 ┊10┊11┊    describe('methods', () => {
 ┊11┊12┊      const userId = Random.id();
-┊12┊  ┊
 ┊13┊13┊      let taskId;
 ┊14┊14┊
 ┊15┊15┊      beforeEach(() => {
 ┊16┊16┊        Tasks.remove({});
-┊17┊  ┊
 ┊18┊17┊        taskId = Tasks.insert({
 ┊19┊18┊          text: 'test task',
 ┊20┊19┊          createdAt: new Date(),
```
```diff
@@ -23,7 +22,20 @@
 ┊23┊22┊        });
 ┊24┊23┊      });
 ┊25┊24┊
-┊26┊  ┊      it('can delete owned task', () => {});
+┊  ┊25┊      it('can delete owned task', () => {
+┊  ┊26┊        // Find the internal implementation of the task method so we can
+┊  ┊27┊        // test it in isolation
+┊  ┊28┊        const deleteTask = Meteor.server.method_handlers['tasks.remove'];
+┊  ┊29┊
+┊  ┊30┊        // Set up a fake method invocation that looks like what the method expects
+┊  ┊31┊        const invocation = { userId };
+┊  ┊32┊
+┊  ┊33┊        // Run the method with `this` set to the fake invocation
+┊  ┊34┊        deleteTask.apply(invocation, [taskId]);
+┊  ┊35┊
+┊  ┊36┊        // Verify that the method does what we expected
+┊  ┊37┊        assert.equal(Tasks.find().count(), 0);
+┊  ┊38┊      });
 ┊27┊39┊    });
 ┊28┊40┊  });
 ┊29┊41┊}
```

[}]: #

And finally, don't forget to import our `tasks.tests` file in `tests/main.js` so that it'd run when during our test command.

[{]: <helper> (diffStep 11.5 noTitle=true)

##### Changed tests&#x2F;main.js
```diff
@@ -1,20 +1 @@
-┊ 1┊  ┊import assert from "assert";
-┊ 2┊  ┊
-┊ 3┊  ┊describe("simple-todos", function () {
-┊ 4┊  ┊  it("package.json has correct name", async function () {
-┊ 5┊  ┊    const { name } = await import("../package.json");
-┊ 6┊  ┊    assert.strictEqual(name, "simple-todos");
-┊ 7┊  ┊  });
-┊ 8┊  ┊
-┊ 9┊  ┊  if (Meteor.isClient) {
-┊10┊  ┊    it("client is not server", function () {
-┊11┊  ┊      assert.strictEqual(Meteor.isServer, false);
-┊12┊  ┊    });
-┊13┊  ┊  }
-┊14┊  ┊
-┊15┊  ┊  if (Meteor.isServer) {
-┊16┊  ┊    it("server is not client", function () {
-┊17┊  ┊      assert.strictEqual(Meteor.isClient, false);
-┊18┊  ┊    });
-┊19┊  ┊  }
-┊20┊  ┊});
+┊  ┊ 1┊import '../imports/api/tasks.tests';🚫↵
```

[}]: #

There's a lot more you can do in a Meteor test! You can read more about it in the Meteor Guide [article on testing](http://guide.meteor.com/testing.html).


[//]: # (foot-start)

[{]: <helper> (navStep)

| [< Previous Step](step10.md) | [Next Step >](step12.md) |
|:--------------------------------|--------------------------------:|

[}]: #
