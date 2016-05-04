##Craneo
### A Minimalist Bot Framework for node.js

### What it is
The base command parsing structure for any small to medium sized bot you intend to make.
Craneo serves as a minimalistic back-end bot framework that specifically reads commands and triggers responses with the use of regular expressions. 

### What it’s not
Craneo is not an out of the box intelligence for your commands, it’s a structure for parsing a command. In other word’s - Here’s the skull, you supply the brain.


###Install
If you have node.js installed you can simply use npm to download it.
```bash 
npm install craneo
```

###Quick Start
Here’s the quickest way to get a bot going with the least amount of steps. 
</br>
###Create a response
Create a `hello-world.js` file inside the main directory.</br>
This function will serve as the first response your bot will have.
```js
var helloWorld = function(context){ 
  console.log( 'Hello world' );
  return false
}
```
</br>

###Create a list for your responses
In order for the bot to read your response you'll need to create a `response-list.js` file.
Inside `response-list.js` create the main response list array and require the `hello-world.js` response function into this file. It will serve as your root response list for all of your bot’s commands and responses.  
```js
var helloWorld = require('./hello-world.js');

var responseList = [
  {
    name: 'helloWorld',
    commands: [ /^(.*?(\Hello\b)[^$]*)$/i ],
    response: hello,
  }
]; 

module.exports = responseList;
```
</br>

### Create a commands list for your response
The `commands:` inside a response object must be supplied using regular expressions. </br>Craneo uses regular expressions to match any command supplied inside the command list with the received message.
</br> By adding the regular expresion `/^(.*?(\Hello\b)[^$]*)$/i` inside our command list, we set up our bot to respond with our hello world function to any commands that include the string 'Hello'.

### Listen to messages
In your main app folder you can require Craneo and your `response-list.js` file.
```js
var Craneo = require('craneo');
var responseList = require('./response-list');
```
Initiate your own instance of a Craneo with by passing it your responseList array inside of an object. 
```js
var bot = Craneo({ responseList: responseList }); 
```
Now you're ready to listen to any message by calling the listen method inside your bot and passing it a message and any arguments that you would like to be passed down to your response.
```js
bot.listen('hello');
```

### Passing down a context
All responses will get passed two context objects. The `context.bot` object will pass down the contents necessary for the bot to function; Mainly the message content `context.bot.message` and a responseList array `context.bot.responseList`.  The `context.bot` object will pass down whatever contents you pass to the bot’s `listen` method.
```js

var responseArgs  = {
  userId : ‘rx-78g’, 
  name:  ‘Amuro Ray’
  type: ‘Gundam’
}
bot.listen( ‘hello’, responseArgs );
```

## Response Types

### Basic Response
As shown in our `hello-world.js` function, a basic response has a series of commands that can match to trigger a basic response function. These function types usually return false. This tells the bot to use the default `responseList` array for the following responses. As with all other response types the a basic response will be passed a context argument with the objects bot and client.

### Response Chain
Think of a response chain as a conversation, once a response matches it returns a list or possible responses that are used instead of the default response.  This closes the bots context for a specific set of actions.  In order to end the response chain, provide a function with `return false`, this will tell Craneo to go back to using the default responseList.
```js
var responseChain = function( context ){ 

  // do your response’s action
  console.log(‘response chain started’ ); 
  
  // Return the following response objects to the bot in the same array format as the response list
  return [
    { 
      name: ‘foo’,
      response: function(){ console.log(‘foo’); }, 
      commands: […]
    },
    { 
      name: ‘bar’,
      response: function(){ console.log(‘bar’); }, 
      commands: […]
    }
  ]
} 
```

### Read Chain
A Read Chain is used when you want to parse a message in detail. By returning the string `’read’` inside of any response property along with a `responseList` property inside that same object the bot will read the message supplied a second time. This time using the responseList supplied inside of the matching object. This can be done in multiple sequences in order to get the most accurate reading of a message. 


```js 
var responseList = [
  {
    name: “show”,
    response: ‘read’, 
    command: [ … ],
    responseList: 
    [ 
      { 
	name: ‘Gundam Wing’
	response: function(){ console.log(‘if we weren’t idiots, we wouldn’t be soldiers.’); },
	command: [ … ],
      },
      { 
	name: ‘MSG’
	response: function(){ console.log(’The OG’); 
	command: […], 
      }
    ]
]

### Important Note
The response list is read in order, repeating a command will result (regex formatting and context) in a matching of the first command of that type only. If there is a need for a command that reads `’Show Gundam Wing’` and a command in a different object that reads `’Show MSG’` then use a read chain with a command of `’show’` with a responseList property that holds the `Gundam Wing` response object and the `MSG` response object; As shown in the example above.


###Command Not Found
Craneo’s parser expects a response named `’commandNotFound’` to be found inside any response list provided. This allows you to supply a custom function that will be triggered whenever the user adds a command that does not match with your response list’s options.


