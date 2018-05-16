# Script Hijacking
A list of different methods that allow you to read (sensitive) data from included or injected JavaScript files

## Simple Variables
```javascript
secretVar = 'This is a secret';
doSomething(secretVar);
```
The variable `secretVar` is declared in a global scope. Therefore it can easily be read by any other code on the same page.

```javascript
console.log(secretVar);
```
[Try it!](https://jsfiddle.net/ea6d4bs6/)

To help protect against this kind of leak, you can use IIFEs together with the `"use strict"` statement like so:
```javascript
(function(){
	"use strict";
	let secretVar = 'This is a secret';
	doSomething(secretVar);
})()
```
[Try it!](https://jsfiddle.net/ea6d4bs6/1/)

Please not that activating strict mode is not necessary for the code to work, especially since it's such a short example. It will however help to prevent coding errors that result in the accidental creation of global variables.

## Object Properties
```javascript
(function(){
	"use strict";
	let secretObject = {};
	secretObject.secretString = 'This is a secret!';
	doSomething(secretObject.secretString);
})()
```
[Try it!](https://jsfiddle.net/6ez220zg/)

This code looks like it's secure. We use strict mode, an IIFE and don't declare any global variable. But since we are using an Object here, the secret string can still be read by anyone who includes the script. The key are the `__defineSetter__` and `__defineGetter__` methods.

By using them on the prototype of the `Object` object to define a setter as well as a getter, all Objects that are created afterwards will have them defined as well. This allows us to execute a function every time a property with the name secretString is declared on an object. The best part is, that this function expects an argument - the value of the property that's being set. We can also save this value in another property, which means that the Object still behaves as expected. We use the getter to read the value from its new location and give it back as expected. That may look like this:

```javascript
Object.prototype.__defineSetter__('secretString', function(value) {
	console.log(value);
	this._secretString = value;
});

Object.prototype.__defineGetter__('secretString', function(){
	return this._secretString
})
```
[Try it!](https://jsfiddle.net/hmmqa7La/)
