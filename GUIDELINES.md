# Projects Coding & Testing Guidelines

This guide is covering the coding guidelines common to all ibm-js hosted projects.

## JSHint
 
All projects MUST comply with the directives in this [jshintrc](.jshintrc)
and MUST enforce it through a [TravisCI](https://travis-ci.org/) check.

JSHint max complexity is set to 10 by default but CAN be temporarily put up to 15 for a given method
if the committer feels this is not hurting the code readability.


## JavaScript & CSS Coding Guidelines

### JavaScript

* Getters/Setters MUST use ES5 getter/setter and not getProp/setProp methods.
* Calling mycomponent.myproperty = value for ES5-like setters SHOULD refresh the component either directly
  or using a simple delayed refresh mechanism provided by delite/Invalidating.
  If that is not the case it must be clearly documented
* All widget/custom elements MUST be enabled for globalization (including translatability & bidi support)
* All widget/custom elements MUST be enabled for accessibility:
    * keyboard support
    * high contrast support (IE and FF on Windows, with High Contrast mode enabled in the os settings)
    * zoom (full zoom, not text only zoom, up to 200%)
    * screen reader accessibility (supported readers are JAWS on Windows and VoiceOver on iOS)
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

### CSS

* JavaScript code SHOULD NOT hard code style but rely on stylesheets
* !important MUST NOT be used
* Stylesheets MUST clearly document which properties we don't support changing. It SHOULD layout properties and rules
 to clearly separate what is not supposed to be changed.
* Widgets styling SHOULD NOT use float style property.
* Widgets Samples SHOULD use delite/themes/defaultapp.css when:
    * laying out element with borders (defaultapp.css set border-box on everything)
    * using an element that uses the whole height of the page (height:100% is needed on ascendants, including body)
    * in need of getting rid of the default margin/padding defined by browsers
   
### Inheritance Guidelines

* Visual components (aka widgets) MUST extend delite/Widget
* Non visual components SHOULD extend delite/Stateful

### Naming Guidelines

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

#### CSS class naming

* component specific CSS classes MUST be prefixed by the component tag name,
  e.g. d-button-whatever, d-treemap-whatever etc.. 
* cross-components CSS classes MUST be prefixed by d-, e.g. d-selected etc…
  and shared across the various projects / components.
* private CSS classes MUST be prefixed by “-” and follow the practices above (so -d-button-whatever)

## Testing Guidelines

All tests MUST be written using [Intern](http://theintern.io/).

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

## Security Guidelines

Use of possible evil functions like `eval()` or equivalents SHOULD be avoided as much as possible.

When they cannot be avoided, each use of an evil function MUST be hinted by jshint in the code. 

The code MUST also contain a comment explaining why we need that call, when it will be triggered.
From this we will be able to extract a security guide listing all the potentially harmful call
and recommendation for the user.
