<!DOCTYPE html>
<html>
<head>
	<title>Simple Todos Notes</title>
	<link rel="stylesheet" type="text/css" href="bootstrap.min.css">
</head>

<style type="text/css">
	section{
		margin-bottom: 10px;
		border-bottom: 2px solid grey;
		margin-left: 8px;
	}
</style>

<body>
	<div class="container">
	<div class="row">
	<div class="col-lg-10 col-offset-lg-1">

	<section id="one">
		<h3>1. Creating an Meteor App</h3>
		<p></p>
	</section>
	<section id="two">
		<h3>2. Templates</h3>
		<p>
			<ol>
				<li><strong>HTML files in Meteor define templates: </strong></li>
				<li>
				Meteor parses HTML files and identifies three top-level tags: <em>head</em>, <em>body</em>, and <em>template</em>.
				</li>
				<li>
					Everything inside <em>template</em> tags is compiled into Meteor templates, which can be included inside HTML with {{> templateName}} or referenced in your JavaScript with Template.templateName.
				</li>
				<li>the body section can be referenced in your JavaScript with Template.body. </li>

				<li><strong>Adding logic and data to templates</strong></li>
				<li>All of the code in your HTML files is compiled with Meteor's Spacebars compiler. </li>
				<li>Spacebars uses statements surrounded by double curly braces such as {{#each}} and {{#if}} to let you add logic and data to your views.</li>
				<li><code>
					{{#each tasks}}<br>
						&nbsp;&nbsp;&nbsp;{{> task}}<br>
					{{/each}}<br>
				</code></li>
				<li>You can pass data into templates from your JavaScript code by defining helpers.</li>
			</ol>
		</p>
	</section>
	
	<section id="three">
		<h3>3. Storing tasks in a collection</h3>
		<p>
			<ol>
				<li>Collections are Meteor's way of storing persistent data.</li>
				<li>Collections can be accessed from both server and client, making it easy to write view logic without having to write a lot of server code.</li>
				<li><strong>Creating a new collection</strong></li>
				<li><code>MyCollection = new Mongo.Collection("my-collection");</code></li>
				<li>On the server, this sets up a MongoDB collection called my-collection.</li>
				<li>On the client, this creates a cache connected to the server collection.</li>

				<li>To create the collection, we define a new tasks modole that creates a Mongo collection and exports it.</li>
				<li>Create tasks collection in imports/api/tasks.js</li>
				<li><code>import { Mongo } from 'meteor/mongo';</code><br>
				<code>export const Tasks = new Mongo.Collection('tasks');</code></li>
				<li>We need to import that module on the server (this creates the MongoDB collection and sets up the plumbing to get the data to the client)</li>
				<li>Load tasks collection on the server in the file server/main.js</li>
				<li><code>import '../imports/api/tasks.js';</code></li>

			</ol>
		</p>
	</section>

	<section id="four">
		<h3>4. Forms and events</h3>
		<p>
			<ol>
				<li>Add form for new tasks in imports/ui/body.html</li>
				<li>Add JS code to add to listen to the submit event on the form in the body.js file.<pre><code>
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
				</code></pre></li>
				<li><h4>Attaching events to templates</h4>
				<ul>
					<li>Event listeners are added to templates by calling Template.templateName.events(...) with a dictionary. The keys describe the event to listen for, and the values are event handlers that are called when the event happens.</li>
					<li>In our case above, we are listening to the submit event on any element that matches the CSS selector .new-task</li>
					<li>The event handler gets an argument called event</li>
					<li>In this case event.target is our form element, and we can get the value of our input with event.target.text.value</li>	
				</ul></li>
				<li><h4>Inserting into a collection: </h4>
				we are adding a task to the tasks collection by calling Tasks.insert()</li>
				
			</ol>
		</p>
	</section>
	<section id="five">
		<h3>5. Checking off and deleting tasks: Update & Remove</h3>
		<p>
			<ol>
				<li>Add buttons to Task component in imports/ui/task.html</li>
				<pre><code>
&lt;template name="task"&gt;
  &lt;li class="{{#if checked}}checked{{/if}}"&gt;
    &lt;button class="delete">&times;&lt;/button&gt;
 
    &lt;input type="checkbox" checked="{{checked}}" class="toggle-checked" /&gt;
 
    &lt;span class="text"&gt;{{text}}&lt;/span&gt;
  &lt;/li&gt;
&lt;/template&gt;					
				</code></pre>
				<li>Now we're making a seperate html template for tasks. Remove the task template definition from body.html.</li>
				<li>Adding event handlers for the newly created task UI elements in new file ui/tasks.js</li>
				<pre><code>
import { Template } from 'meteor/templating';

import { Tasks } from '../api/tasks.js';
 
import './task.html';
 
Template.task.events({
  'click .toggle-checked'() {
    // Set the checked property to the opposite of its current value
    Tasks.update(this._id, {
      $set: { checked: ! this.checked },
    });
  },
  'click .delete'() {
    Tasks.remove(this._id);
  },
});
				</code></pre>
				<li>THe body template uses the task template so import it in body.js</li>
				<li>In the event handlers this._id refers to the id of the current task. Once we have the _id, we can use update and remove to modify the relevant task.</li>
				<li><h4>Update</h4>
					<ul>
						<li>The update function on a collection takes two arguments.</li>
						<li>i) a selector that identifies a subset of the collection</li>
						<li>ii) an update parameter that specifies what should be done to the matched objects</li>
						<li>In this case, the selector is just the _id of the relevant task. The update parameter uses $set to toggle the checked field, which will represent whether the task has been completed.</li>
					</ul>
				</li>
				<li><h4>Remove</h4>
				The remove function takes one argument, a selector that determines which item to remove from the collection.
				</li>
			</ol>
		</p>
	</section>
	<section id="six">
		<h3>6. Storing temporary UI state in a Reactive Dictionary</h3>
		<ol>
			<li>
				we'll add a client-side data filtering feature to our app, so that users can check a box to only see incomplete tasks. We're going to learn how to use a ReactiveDict to store temporary reactive state on the client. A ReactiveDict is like a normal JS object with keys and values, but with built-in reactivity.
			</li>
			<li>
				Add hide-completed checkbox to HTML in imports/ui/body.html
			</li>
			<li>Then we need to set up a new ReactiveDict and attach it to the body template instance (as this is where we'll store the checkbox's state) when it is first created</li>
			<li>Add state dictionary to the body in imports/ui/body.js</li>
			<li>Add this before the helpers and events
			<pre><code>
Template.body.onCreated(function bodyOnCreated() {
  this.state = new ReactiveDict();
});
			</code></pre></li>
			<li>Then, we need an event handler to update the ReactiveDict variable.
			<pre><code>
'change .hide-completed input'(event, instance) {
	instance.state.set('hideCompleted', event.target.checked);
},
			</code></pre>
			</li>
			<li>Then we need to update the Template.body.helpers which will only show the unchecked tasks if chosen so
			<pre><code>
const instance = Template.instance();
if (instance.state.get('hideCompleted')) {
  // If hide completed is checked, filter tasks
  return Tasks.find({ checked: { $ne: true } }, { sort: { createdAt: -1 } });
}
// Otherwise, return all of the tasks
			</code></pre></li>
			<li class="bg-info">ReactiveDict is the same way, but is not synced with the server like collections are. This makes a ReactiveDict a convenient place to store temporary UI state like the checkbox above.</li>

		</ol>
	</section>
	<section id="seven">
		<h3>7. Adding user accounts</h3>
		<ol>
			<li>
				{{> loginButtons}} adds a login dropdown.
			</li>
			<li>
				Make a new file imports/startup/accounts-config.js to add accounts settings. Code below configures the accounts UI to use usernames instead of email.
				<pre><code>
import { Accounts } from 'meteor/accounts-base';

Accounts.ui.config({
passwordSignupFields: 'USERNAME_ONLY',
});
				</code></pre>
			</li>
			<li><code>import '../imports/startup/accounts-config.js';</code>
			Add this to main.js for above code to work.</li>
			<li>We will add two new fields to the tasks collection.
			<ul>
				<li>owner - the _id of the user that created the task</li>
				<li>username - the username of the user that created the task. We will save the username directly in the task object so that we don't have to look up the user every time we display the task.</li>
			</ul></li>
			<li>Changes to body.js:
			<code>import { Meteor } from 'meteor/meteor';</code>
			And add this to Tasks.insert():<br>
			<code>owner: Meteor.userId(),</code><br>
			<code>username: Meteor.user().username,</code></li>
			<li>Use these to hide stuff that should be visible to only logged in users:<br>
			<code>
				{{#if currentUser}}
        			.........
      			{{/if}}
			</code></li>
		</ol>
	</section>		
	</div>
	</div>
	</div>
</body>
</html>