# Mini Chat

Adapted from google's ../codelab.

Deployed at <https://chat-client-4c5eb.firebaseapp.com>

## 1. Create a Firebase project

See <https://codelabs.developers.google.com/codelabs/firebase-web/#3>

Create an `index.html` file and save the script tag generated to it.

## 2. Change Firebase database

On the Firebase console click on Database on the sidebar. Then click on Rules at
the top.

Change `"auth != null"` to `true` for both read and write:

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

Note, this give anyone read and write access to your database.

## 3. Layout HTML for Chat App

Add at the top of your `index.html` somewhere for the messages to go and the
form to input them:

```html
<!DOCTYPE html>
<div>
  <ul id="messages"></ul>
  <form id="message-form">
    <input type="text" id="message">
    <button type="submit">Send</button>
  </form>
</div>
```

Then there should be the code copied from your firebase project:

```html
<script src="https://www.gstatic.com/firebasejs/3.4.1/firebase.js"></script>
<script>
  ...
```

Finally make sure at the end of your html page you include your javascript file:

```html
<script src="main.js"></script>
```


## 4. Create `main.js`

Create your javascript file and set up a class that will hold the chat
application logic.

```js
// Initializes Chat
function Chat() {
  if (!window.firebase || !(firebase.app instanceof Function) || !window.config) {
    window.alert('You have not configured and imported the Firebase SDK.');
    return;
  }

  // Initialize Firebase database connection.
  this.database = firebase.database();

  // Shortcuts to DOM Elements.
  this.messageList = document.getElementById('messages');
  this.messageForm = document.getElementById('message-form');
  this.messageInput = document.getElementById('message');

  // Saves message on form submit.
  this.messageForm.addEventListener('submit', this.saveMessage.bind(this));

  // Load previous chat messages.
  this.loadMessages();

  // Focus on the input
  this.messageInput.focus();
}
```

Then make sure to initialize your Chat class when the page is loaded.

```js
window.onload = function() {
  new Chat();
};
```

## 5. Load previous messages

```js
// Loads chat messages history and listens for upcoming ones.
Chat.prototype.loadMessages = function() {
  // Reference to the /messages/ database path.
  this.messagesRef = this.database.ref('messages');
  // Make sure we remove all previous listeners.
  this.messagesRef.off();

  // Loads the last 12 messages and listen for new ones.
  var setMessage = function(data) {
    var val = data.val();
    this.displayMessage(data.key, val.text);
  }.bind(this);
  this.messagesRef.limitToLast(12).on('child_added', setMessage);
  this.messagesRef.limitToLast(12).on('child_changed', setMessage);
};
```

## 6. Display messages on the page

```js
// Displays a Message in the UI.
Chat.prototype.displayMessage = function(key, text) {
  var msg = document.getElementById(key);
  // If an element for that message does not exists yet we create it.
  if (!msg) {
    var msg = document.createElement('li');
    msg.innerHTML = text;
    msg.setAttribute('id', key);
    this.messageList.appendChild(msg);
  }
};
```

## 7. Enable the user to save a message

```js
// Saves a new message on the Firebase DB.
Chat.prototype.saveMessage = function(e) {
  e.preventDefault();
  // Check that the user entered a message.
  if (this.messageInput.value) {
    // Add a new message entry to the Firebase Database.
    this.messagesRef.push({text: this.messageInput.value}).then(function() {
      // Clear message text field and focus on it.
      this.messageInput.value = '';
      this.messageInput.focus();
    }.bind(this)).catch(function(error) {
      console.error('Error writing new message to Firebase Database', error);
    });
  }
};
```

### 8. Test it out!

Open up index.html in one or more tabs and try it out.

![Screen shot of chat working](screenshot.png)
