### 1. Creating an Meteor App

### 2. Templates

1.  **HTML files in Meteor define templates:**
2.  Meteor parses HTML files and identifies three top-level tags:
    *head*, *body*, and *template*.
3.  Everything inside *template* tags is compiled into Meteor templates,
    which can be included inside HTML with {{\> templateName}} or
    referenced in your JavaScript with Template.templateName.
4.  the body section can be referenced in your JavaScript with
    Template.body.
5.  **Adding logic and data to templates**
6.  All of the code in your HTML files is compiled with Meteor's
    Spacebars compiler.
7.  Spacebars uses statements surrounded by double curly braces such as
    {{\#each}} and {{\#if}} to let you add logic and data to your views.
8.  `                     {{#each tasks}}                            {{> task}}                     {{/each}}                 `
9.  You can pass data into templates from your JavaScript code by
    defining helpers.

### 3. Storing tasks in a collection

1.  Collections are Meteor's way of storing persistent data.
2.  Collections can be accessed from both server and client, making it
    easy to write view logic without having to write a lot of server
    code.
3.  **Creating a new collection**
4.  `MyCollection = new Mongo.Collection("my-collection");`
5.  On the server, this sets up a MongoDB collection called
    my-collection.
6.  On the client, this creates a cache connected to the server
    collection.
7.  To create the collection, we define a new tasks modole that creates
    a Mongo collection and exports it.
8.  Create tasks collection in imports/api/tasks.js
9.  `import { Mongo } from 'meteor/mongo';`\
     `export const Tasks = new Mongo.Collection('tasks');`
10. We need to import that module on the server (this creates the
    MongoDB collection and sets up the plumbing to get the data to the
    client)
11. Load tasks collection on the server in the file server/main.js
12. `import '../imports/api/tasks.js';`

### 4. Forms and events

1.  Add form for new tasks in imports/ui/body.html
2.  Add JS code to add to listen to the submit event on the form in the
    body.js file.

        Template.body.events({
          'submit .new-task'(event) {
            // Prevent default browser form submit
            event.preventDefault();
         
            // Get value from form element
            const target = event.target;
            const text = target.text.value;
         
            // Insert a task into the collection
            Tasks.insert({
              text,
              createdAt: new Date(), // current time
            });
         
            // Clear form
            target.text.value = '';
          },
        });
                        

3.  #### Attaching events to templates

    -   Event listeners are added to templates by calling
        Template.templateName.events(...) with a dictionary. The keys
        describe the event to listen for, and the values are event
        handlers that are called when the event happens.
    -   In our case above, we are listening to the submit event on any
        element that matches the CSS selector .new-task
    -   The event handler gets an argument called event
    -   In this case event.target is our form element, and we can get
        the value of our input with event.target.text.value

4.  #### Inserting into a collection:

    we are adding a task to the tasks collection by calling
    Tasks.insert()

### 5. Checking off and deleting tasks: Update & Remove

1.  Add buttons to Task component in imports/ui/task.html
2.  Now we're making a seperate html template for tasks. Remove the task
    template definition from body.html.
3.  Adding event handlers for the newly created task UI elements in new
    file ui/tasks.js
4.  THe body template uses the task template so import it in body.js
5.  In the event handlers this.\_id refers to the id of the current
    task. Once we have the \_id, we can use update and remove to modify
    the relevant task.
6.  #### Update

    -   The update function on a collection takes two arguments.
    -   i) a selector that identifies a subset of the collection
    -   ii) an update parameter that specifies what should be done to
        the matched objects
    -   In this case, the selector is just the \_id of the relevant
        task. The update parameter uses \$set to toggle the checked
        field, which will represent whether the task has been completed.

7.  #### Remove

    The remove function takes one argument, a selector that determines
    which item to remove from the collection.

### 6. Storing temporary UI state in a Reactive Dictionary

1.  we'll add a client-side data filtering feature to our app, so that
    users can check a box to only see incomplete tasks. We're going to
    learn how to use a ReactiveDict to store temporary reactive state on
    the client. A ReactiveDict is like a normal JS object with keys and
    values, but with built-in reactivity.
2.  Add hide-completed checkbox to HTML in imports/ui/body.html
3.  Then we need to set up a new ReactiveDict and attach it to the body
    template instance (as this is where we'll store the checkbox's
    state) when it is first created
4.  Add state dictionary to the body in imports/ui/body.js
5.  Add this before the helpers and events

        Template.body.onCreated(function bodyOnCreated() {
          this.state = new ReactiveDict();
        });
                    

6.  Then, we need an event handler to update the ReactiveDict variable.

        'change .hide-completed input'(event, instance) {
            instance.state.set('hideCompleted', event.target.checked);
        },
                    

7.  Then we need to update the Template.body.helpers which will only
    show the unchecked tasks if chosen so

        const instance = Template.instance();
        if (instance.state.get('hideCompleted')) {
          // If hide completed is checked, filter tasks
          return Tasks.find({ checked: { $ne: true } }, { sort: { createdAt: -1 } });
        }
        // Otherwise, return all of the tasks
                    

8.  ReactiveDict is the same way, but is not synced with the server like
    collections are. This makes a ReactiveDict a convenient place to
    store temporary UI state like the checkbox above.

### 7. Adding user accounts

1.  {{\> loginButtons}} adds a login dropdown.
2.  Make a new file imports/startup/accounts-config.js to add accounts
    settings. Code below configures the accounts UI to use usernames
    instead of email.

        import { Accounts } from 'meteor/accounts-base';

        Accounts.ui.config({
        passwordSignupFields: 'USERNAME_ONLY',
        });
                        

3.  `import '../imports/startup/accounts-config.js';` Add this to
    main.js for above code to work.
4.  We will add two new fields to the tasks collection.
    -   owner - the \_id of the user that created the task
    -   username - the username of the user that created the task. We
        will save the username directly in the task object so that we
        don't have to look up the user every time we display the task.

5.  Changes to body.js: `import { Meteor } from 'meteor/meteor';` And
    add this to Tasks.insert():\
     `owner: Meteor.userId(),`\
     `username: Meteor.user().username,`
6.  Use these to hide stuff that should be visible to only logged in
    users:\

    `                 {{#if currentUser}}                     .........                 {{/if}}             `

