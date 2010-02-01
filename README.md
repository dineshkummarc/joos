# joos == "OO in JS"

"joos" is a compact library that provides an elegant way of writing Object-Oriented JavaScript.

joos **is**:

  * Lightweight - &lt; 1.5KB minified and gzipped
  * Library agnostic - Use it with jQuery, Prototype, Ext... whatever.
  * Powerful - Supports OO constructs that aren
  * Elegant - joos-enabled code is more readable, and just *looks* better
  * Cross-platform - Urrr... well... sort of.  I need to spec out what the exact browser support story is.

joos **is not**:

  * A new language
  * A replacement for existing libraries.

## API Definition Objects (APID's)
joos allows you to define your APIs using a single object (referred to in joos as an API Definition, or "APID").  It's no longer necessary to write code that declares the constructor function in one place, assigns the prototype properties in another, the class properties in yet a third place and, finally has some static initialization code in yet another spot.  Just put it all in an API Definition object, and let joos do the rest.

## Killer "super" support
joos allows you to inherit superclass methods by simply calling 'this.\_super' from within subclass methods.  But this isn't limited to just traditional class-based inheritance.  You can use this exact same technique when overriding native methods, and even when mixing methods into object instances.  It all just works.

And the approach joos uses for this is *fast*. (Faster than Prototype's $super!)

## APID Key Syntax and Modifiers
But joos isn't just about letting you lay your code out in a clean, logical manner, and leverage easy-to-use, fast, method inheritance.  joos allows you to prefix your APID keys with special modifiers that provide useful, powerful functionality.  We'll show you how this works below, but for now here's the list of currently supported modifiers:

### **bind**$*name*
Activates binding on the *name* function.

Functions are bound to each instance (cool, right?), meaning regardless of how the function is invoked, 'this' will refer to the object instance.

If used in conjunction with static$, 'this' will refer to the class object.

Note: When using joos.extendClass() to enhance a non-joos class, non-static bind$s will produce an error.

### **get**$*name*
### **set**$*name*
Applies the *name* function using built-in getter/setter support.

Note: get$ and set$ will produce errors on older browsers [TODO: such as?]

### **static**$*name*
### $*name*
Identifies *name* as a static (class) member, as opposed to a prototype member.

This can be abbreviated as just "$".  "$*name*" is identical to "static$*name*"

### **superclass**$
Specifies the superclass to inherit from

Note: Only meaningful in joos.createClass.

### **initialize**$
Identifies a static initializer method.  The supplied function will be invoked immediately before returning from joos.createClass/extendClass/extendObject.

### **initialize**
Specifies the instance initializer method.  joos classes invoke this method as part of "new" object creation.

(This is not technically a modifier, it's just a special property)

## API (w/ examples)
Okay, APIDs and key modifiers sound good, but what does this stuff look like? How's it work?  Let's check out a few examples ...

### joos.createClass()

Classes are created with "joos.createClass(apid)", where 'apid' is an API Definition (APID) object.  The simplest form of this is:

      var MyClass = joos.createClass();

This creates a new, empty class.  But for a slightly more interesting example, let's instead create a class that has some properties and methods by passing in an API Definition object:

    var MyClass = joos.createClass({

      // Define a class variable, "MyClass.SOME_CONSTANT"
      $SOME_CONSTANT: 'a value',

      // Define a class method, "MyClass.find"
      $find: function() {
        // (code goes here)
      }

      // Define a static initializer method. This method is called once, immediately
      // after the class is created.
      initialize$: function() {
        // (code goes here)
      },

      // Define initializer.  This is invoked automatically during new object
      // creation (e.g. "new MyClass()" will call this)
      initialize: function() {
        // (code goes here)
      },

      // Define a getter for the 'this.something' property
      get$something: function() {
        return this._something;
      },

      // Define a setter for the 'this.something' property
      set$something: function(val) {
        this._something = val;
      },

      // Define this.doSomething() method, and tell joos to bind it to each
      // object instance
      bind$doSomething: function() {
        // (code goes here)
      }
    });

#### Subclasses

With MyClass defined, we can create a subclass like this:

    var MySubclass = joos.createClass({
      superclass$: MyClass
    });

Again, not very interesting.  So lets flesh it out a bit:

**Note the use of this.\_super() to invoke superclass methods (for both instance and class methods, no less!)**

    var MySubclass = joos.createClass({

      // Declare MyClass as our superclass
      superclass$: MyClass,

      // static initializer for the subclass
      initialize$: function() {
        // (code goes here)
      },

      // Override the setter for 'something'
      set$something: function(val) {
        // Use this.\_super() to invoke superclass' method!
        this._super(val + ' blah');
      },

      // Override doSomethin()
      // object instance
      bind$doSomething: function() {
        // (function implementation)
      }
    });

### joos.extendClass()
joos.extendClass() allows you to add members to an existing class.  This works for _any_ class, not just those created with joos.createClass.  For example, to extend the native Array class:

    joos.extendClass({
      // Define Array.SOME_CONSTANT
      $SOME_CONSTANT: 'a value',

      // Override super method to do something interesting, like filter results
      $find: function() {
        var results = this._super();
        // (filter results)
        return results;
      }

      // Replace prior initialize() method
      initialize: function() {
        this._super()
        // (finish initializing)
      }
    });

### joos.extendObject()

joos.extendObject() allows you to extend object instances in much the same way you would extend a class.  This allows you to easily leverage the powerful "mixin" pattern.

