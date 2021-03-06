node-function-class
===================
### Easy `Function` subclasses
[![Build Status](https://secure.travis-ci.org/TooTallNate/node-function-class.svg)](http://travis-ci.org/TooTallNate/node-function-class)

Creates a function instance with specified `name` and `length` and `prototype`.
Essentially, it's everything that you need to subclass the ECMAscript `Function`
built-in type.

You use the `function-class/invoke` Symbol to define the logic that will be
executed when the created function instance is invoked.

Installation
------------

Install with `npm`:

``` bash
$ npm install function-class
```


Examples
--------

One-off:

``` js
var createFunction = require('function-class');
var invoke = require('function-class/invoke');

var fn = createFunction('theName', 6);
assert.equal(fn.name, 'theName');
assert.equal(fn.length, 6);

fn[invoke] = function (arg) {
  return this.name + ' ' + this.length + ' ' + arg;
};

assert.equal('theName 6 foo', fn('foo'));
```

Subclass:

``` js
var inherits = require('util').inherits;
var createFunction = require('function-class');
var invoke = require('function-class/invoke');

function FunctionSubclass (foo) {
  if (typeof this !== 'function')
    return createFunction('subclass', 0, FunctionSubclass, arguments);

  this.foo = foo;
  assert.equal(typeof this, 'function');
  assert.ok(this instanceof FunctionSubclass);
}
inherits(FunctionSubclass, Function);

FunctionSubclass.prototype[invoke] = function () {
  return this.foo;
}

// usage
var f = new FunctionSubclass('foo');
assert.equal(f.name, 'subclass');
assert.equal(f.length, 0);
assert.equal(f(), 'foo');
```


API
---

The main export is the `createFunction()` function, which returns a new Function
instance.

## createFunction(name, arity, constructor, arguments) → Function<Type>

 * name - `String` - the `name` to set for the created function, must be a string (even containing spaces, unicode, emoji, or otherwise invalid JS identifier characters)
 * arity - `Number` - the `length` to set for the created function, must be a positive integer
 * constructor - `Function` - the class constructor to use invoke on the created instance, and inherit from the `prototype` of
 * arguments - `Array` - array of values to invoke `constructor` with

All arguments are optional. You must specify the `invoke` function to execute on
the created function instance.

``` js
var createFunction = require('function-class');

// inherits from Function
var fn = createFunction('foo', 3);

// inherits from FunctionSubclass with constructors invoked with args
var args = [ 'foo', 'bar' ];
var fn = createFunction('name', 0, FunctionSubclass, args);
```

### invoke → Symbol

You use the `invoke` Symbol to define the function to execute when a created
function instance is invoked.

Note that `this` in the invoke function is bound to the created function instance
when invoked without a receiver (i.e. "globally"). Otherwise, `this` respects the
left-hand receiver, or target when used with `.call()`/`.apply()`.

``` js
var invoke = require('function-class/invoke');

fn[invoke] = function () {
  console.log('fn has been invoked!');
  return this;
};

fn();
// fn has been invoked!

assert.equal(fn(), fn);
assert.equal(fn.call(global), global);
```
