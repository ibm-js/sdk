# Projects Coding & Testing Guidelines

This guide covers the coding guidelines common to all ibm-js hosted projects.

## End User Requirements

* All widget/custom elements MUST be enabled for globalization (including translatability & bidi support)
* All widget/custom elements MUST be enabled for accessibility:
    * keyboard support
    * zoom (full zoom, not text only zoom, up to 200%)
    * screen reader accessibility (supported readers are JAWS on Windows and VoiceOver on iOS)
    * must support high contrast (IE and FF on Windows, with high contrast mode enabled in the OS settings);
      device specific themes (currently iOS and holodark) are exempt from this rule
* The projects will only support modern browser/platforms: FF31+, Chrome latest, IE9/10+, Safari 7+,
  Android 4.1+, iOS6+, WindowsPhone8+ 

See https://docs.google.com/document/d/1Sa0rn4udqO_ea20o4xxogRP6P6ZLgHPMEj4A2V4g7hM
for more details.

## Javascript Coding Guidelines

Our components are written in Javascript rather than a derivative like Typescript or CoffeeScript.

### JSHint
 
All projects MUST comply with the directives in this [jshintrc](.jshintrc)
and MUST enforce it through a [TravisCI](https://travis-ci.org/) check.

JSHint max complexity is set to 10 by default but CAN be temporarily put up to 15 for a given method
if the committer feels this is not hurting the code readability.

### AMD

For projects to be used in the browser (as opposed to Node.js specific projects),
modules are wrapped in [AMD](http://requirejs.org/docs/whyamd.html#amd) rather than CommonJS or UMD format.
Rationale: this is mainly for the benefit of the build tool.

[RequireJS](http://requirejs.org/) is our main AMD loader.  Plugins and builds are tailored for RequireJS.

We standardize on using the following plugins:

* [requirejs-domReady!](https://github.com/requirejs/domReady) - to detect when the DOM has finished loading
* [requirejs-text!](https://github.com/requirejs/text) - to load non JS files (ex: HTML)
* [requirejs-dplugins/i18n!](https://github.com/ibm-js/requirejs-dplugins/blob/master/i18n.js) - for loading
  message files (translated into different languages).
* [requirejs-dplugins/has!](https://github.com/ibm-js/requirejs-dplugins/blob/master/has.js) -
  Feature and browser sniffing is done via
  `has.add()` and `has()`, so that a build can strip unneeded code.

### API documentation

Public methods and properties should use JSDoc format, as detailed in
https://docs.google.com/document/d/1fGBLuDAJRHFuSE4_NM2YfbRcCeYV40KWDXCuVyiSV_U/edit#heading=h.mrkbtrfyd36f


### Other JavaScript Coding Guidelines

* Getters/Setters MUST use ES5 getter/setter and not getProp/setProp methods.
* Calling mycomponent.myproperty = value for ES5-like setters SHOULD refresh the component either directly
  or using a simple delayed refresh mechanism provided by decor/Invalidating.
  If that is not the case it must be clearly documented
* Variables SHOULD be declared as close to the point where they are first used as it helps to cluster groups of code
  together instead of spreading them throughout a function.
  Variables that are assigned in multiple places of a function (e.g. loop counters)
  CAN be declared at the beginning of the function.
* In order to associate an object to be used as `this` when calling a function:
    * If the called function is public _and_ documented the code MUST use a closure construct with a `self` variable to preserve AOP:
    ```js
    var self = this;
    this.on("DOMNodeInserted",  function(){ self.publicmethod(); });
    ```
    * If the called function is either private _or_ not documented the core MUST use Function.prototype.bind():
    ```js
    this.on("DOMNodeInserted",  this._privatemethod.bind(this));
    ```
* By default, code SHOULD NOT check properties or parameters for validity, if there's a particular case where it's especially 
helpful to the user to check, then code SHOULD: 
    * throw an exception rather than using console.log() or console.error()
    * add comment to the code about why you throwing the exception
* Widget Template MUST have a top level tag of type ```<template>```.

### OO

If you need an OO framework to declare classes (with multiple inheritance, advice, etc.), we use
[dcl](http://dcljs.org).  However, it's also acceptable to declare classes native by specifying a constructor
and prototype.

For classes defined by [dcl](http://dcljs.org) (or delite/register), a method in a subclass should call the
superclass' method (of the same name) via `dcl.superCall()`, or its alias `delite/register#superCall()`.  For example:

```js
appendChild: dcl.superCall(function (sup) {
	return function (child) {
		var res = sup.call(this.containerNode, child);
		this.onAddChild(child);
	};
})
```

`dcl.after()`/`dcl.before()`/`dcl.around()` should
be reserved for when the superclass needs to execute some code before/after all the subclasses' methods run.
For example, `delite/Widget#attachedCallback()` sets `this.attached = true` after the `attachedCallback()` methods
in all the subclasses run.

Note that we sometimes automatically chain class methods using `dcl.chainAfter()` / `dcl.chainBefore()`, for example
delite/Widget does:

```js
dcl.chainAfter(Widget, "postRender");
```

In this case, the `dcl.superCall()` is implied, and should not done explicitly, for example,
a widget's `postRender()` method would be:

```js
postRender: function () {
	// code here will be executed after superclasses' postRender() methods run
}
```

### Inheritance Guidelines

* Visual components (aka widgets) MUST extend delite/Widget
* Non visual components SHOULD extend decor/Stateful

### Libraries

We've standardized on a number of support libraries, both in and outside of the ibm-js account:

* [has](https://github.com/ibm-js/requirejs-dplugins/blob/master/has.js) - Feature and browser sniffing is done via
  `has.add()` and `has()`, so that a build can strip unneeded code. We use the `has()` implementation in
   `ibm-js/requirejs-dplugins/has`.

See also the "AMD" and "OO" sections above.

#### Properties/functions naming

* Properties and functions that are not meant to be used by the application developer
  (either private or only used internally) MUST either:
    * be closure-bound in the AMD module without underscore
    * start with underscore
* Mixin private methods CAN be postfixed or marked by the Mixin name to avoid potential clash

#### Properties naming

* Properties having similar effects on components MUST share the same naming. Examples:
    * property to set/get data store MUST be called store
    * property to set/get selected items MUST be called selectedItems
    * …
* Properties that are used to bind a component property to a data for a property xxx MUST be called
  (StoreMap mixin is enforcing that in delite):
    * xxxAttr for binding by data attribute (like labelAttr for label property)
    * xxxFunc for binding by data function (like labelFunc for label property)

#### Events naming

* component/mixin-specific events SHOULD be prefixed by the component/mixin AMD module name,
  e.g. button-whatever, selection-change etc..
* project-specific events that can be used cross-components MUST be prefixed by the project name
  e.g. dapp-eventname, delite-display...
* event topic names MUST be namespaced by the project name e.g.delite/topicname
* event handlers passed to on, addEventListener SHOULD be named eventNameHandler or
  _eventNameHandler depending on whether they are public or private.

#### Class/Modules naming

* Modules that return an instantiable object MUST start with capital letter
* Modules that return an object that can’t be used as a constructor MUST start with lowercase letter
* Modules whose interface (aka methods) is not used by the application developer
  (either because it is fully private or because it is only used by component developer) MUST start with underscore.
  That means totally private classes but also mixins on public classes that are here for purely internal reasons.
* Modules whose interface (aka methods) is used by the application developer MUST NOT start with underscore
  even if the application developer does not instantiate it.
  That means any class or mixin that has at least a property or method the application developer needs to call.
* Public classes and that must not be instantiated because they are just parent classes for implementations SHOULD be
  suffixed by Base (e.g. CalendarBase) only when they need to differentiate themselves from the actual user accessible
  implementation (e.g. Calendar).
* Mixins SHOULD NOT be postfixed by Mixin (ala dgrid or dtreemap, so Selection not SelectionMixin for example)

## Security Guidelines

Use of possible evil functions like `eval()` or equivalents SHOULD be avoided as much as possible.

When they cannot be avoided, each use of an evil function MUST be hinted by jshint in the code.

The code MUST also contain a comment explaining why we need that call, when it will be triggered.
From this we will be able to extract a security guide listing all the potentially harmful call
and recommendation for the user.


## CSS

* CSS is generated from Less files (i.e. we are using Less rather than Stylus)
* JavaScript code SHOULD NOT hard code style but rely on stylesheets
* !important MUST NOT be used
* Stylesheets MUST clearly document which properties we don't support changing. It SHOULD layout properties and rules
 to clearly separate what is not supposed to be changed.
* Widgets styling SHOULD NOT use float style property.
* Widgets Samples SHOULD use delite/themes/defaultapp.css when:
    * laying out element with borders (defaultapp.css set border-box on everything)
    * using an element that uses the whole height of the page (height:100% is needed on ascendants, including body)
    * in need of getting rid of the default margin/padding defined by browsers
* bootstrap theme will use font-icons rather than CSS sprites, in order to support high-contrast mode and
  also theme customization (i.e. changing the colors of the icons)


#### CSS class naming

* component specific CSS classes MUST be prefixed by the component tag name,
  e.g. d-button-whatever, d-treemap-whatever etc.. 
* cross-components CSS classes MUST be prefixed by d-, e.g. d-selected etc…
  and shared across the various projects / components.
* private CSS classes MUST be prefixed by “-” and follow the practices above (so -d-button-whatever)


## Testing Guidelines

### API For Writing Tests

All tests MUST be written using [Intern](http://theintern.io/).
Tests should be written using  `intern!object`, which provides the `registerSuite()` method and also
the API for functional tests.

For assertions, we use the `intern/chai!assert` module.
We prefer `assert.strictEqual()` to `assert.equal()`, just like we prefer `===` to `==`.
Also, we add hints to each assertion so if there's a failure we know which assert fired.  For example:

```
assert.strictEqual(width, "100px", "width");
assert.strictEqual(height, "100px", "height");
```

#### Functional tests

For functional tests, we are using the
[Intern Promised Based Webdriver API](https://github.com/theintern/intern/blob/1.7/lib/wd.js), where
the test is written as a promise chain, and that chain is returned, ex:

```js
return this.remote
	.elementById("stub-for-blurring")
		.click()
		.end()
	.wait(500)
	.elementById("choiceDropDown")
		.isDisplayed(function (err, displayed) {
			assert.isFalse(displayed, "choiceDropDown popup not visible");
		})
		.end();
```

#### Unit tests

When practical, asynchronous unit tests should be written as chained promises too, for example:

```
this.timeout = 10000;		// (if necessary; default is 1000ms)
return myWidget.open().then(function () {
    return myWidget.close();
}.then(function() {
    var cs = getComputedStyle(myWidget);
    assert.strictEqual(cs.width, "100px", "myWidget width");
});
```

When it's not possible to make a promise chain,
we use Intern's `this.async(timeout)` method, along with `d.rejectOnError()` and `d.callback()`, for example:


```
var dfd = this.async(10000);
myWidget.open();
setTimeout(d.rejectOnError(function () {
   myWidget.close();
   setTimeout(d.callback(function () {
     var cs = getComputedStyle(myWidget);
     assert.strictEqual(cs.width, "0px", "myWidget width");
   }, 100);
}), 100);
```

Note how the final asynchronous callback must be wrapped by `d.callback()`,
and all other (intermediate) asynchronous callbacks are wrapped by `d.rejectOnError()`.


### Widget Testing

#### Unit tests SHOULD cover
 
* checking the widget state including default properties values after programmatic creation
* checking the widget state including default properties values after markup creation
* setting & checking dynamically each property (except the ones clearly documented as not being mutable)
  on a created instance
* calling & checking each widget public method on a created instance
* checking that each event the widget is firing on method calls / property change is correctly received
* checking the widget resources are properly released on destroy

#### Functional tests SHOULD cover

* checking the state of the widget during and after the typical user interaction scenarios for the widget
* checking that each event the widget is firing on user interaction is correctly received
