# SPOTLIGHT DOCUMENTATION #

1. [What is Spotlight?](#1)
2. [Modes](#2)
3. [Navigation](#3)
4. [Containers](#4)
5. [Nesting](#5)
6. [Events](#6)
	1. [Spotlight events list](#6.1)
	2. [Preventing/Allowing default DOM events](#6.2)
	3. [Sequence of Spotlight events](#6.3)
	4. [Accelerated keydown events](#6.4)
	5. [Scroll events](#6.5)
7. [Extending Spotlight](#7)
	1. [Spotlight Decorators](#7.1)
	2. [Extending controls](#7.2)

<a name="1"></a>
## 1. WHAT IS SPOTLIGHT? ##

Spotlight is an extensible utility that enables users to navigate Enyo
applications using a keyboard or television remote control.  Responding to input
from the **UP**, **DOWN**, **LEFT**, **RIGHT**, and **RETURN** keys, Spotlight
provides a navigation experience that compares favorably to that of a mouse on a
computer.

In addition, Spotlight includes support for Point-and-Click events, so all bases
are covered.

To add **[Spotlight](kind.Spotlight.js)** to an application, simply include the
`Spotlight` package in your `package.js` file and add a Spotlight control as a
top-level component in the app kind:

    enyo.kind({
        name: 'App',
        kind: 'FittableRows',   
        components:[   
            {kind: 'enyo.Spotlight', defaultControl: 'list1'},
            ...
        ],
        ...
    });

<a name="2"></a>
## 2. MODES ##

Spotlight operates in two mutually exclusive modes: **5-way mode** and **Pointer
mode**.  By default, it is configured to switch between these modes whenever
suitable input is received--i.e., it switches to pointer mode on `mousemove`,
and back to 5-way mode on `keydown`.

The Spotlight API also provides a way to explicitly perform the switch:

    `Enyo.Spotlight.setPointerMode([BOOLEAN]);`

<a name="3"></a>
## 3. NAVIGATION ##

**[Spotlight](kind.Spotlight.js)** allows navigation between Enyo controls by
assigning focus to one control at a time.  When a control is focused, it takes
on the CSS class `.spotlight`, which allows focused controls to be styled on a
per-kind basis using `.<kindClass>.spotlight` selectors.

In order to make a control focusable (or **"spottable"**) with Spotlight, simply
set its `spotlight` property to `TRUE`, like so:

    {name: 'mybutton', tag: 'button', spotlight: true}
	
When the application loads, Spotlight searches for a control with the name
specified in its `defaultControl` property; if `defaultControl` is not
specified, Spotlight assigns focus to the first available spottable control.

In 5-way mode, Spotlight uses the
[Nearest Neighbor Algorithm](kind.Spotlight.NearestNeighbor.js) to determine
which spottable control is the nearest in the direction of navigation.  The
coordinates of a spottable control are derived from its actual position on the
screen.

It's worth noting that spottable controls may be found on different hierarchical
levels of an Enyo component tree.  Spotlight facilitates seamless navigation
between the topmost spottable components found in the tree.

<a name="4"></a>
## 4. CONTAINERS ##

In order to organize controls into navigation groups, we have created Spotlight
containers. 

A good example use case for containers is a set of radio buttons that must be
navigable separate from the rest of the app's controls.

When a Spotlight container is focused, it passes the focus to its own hierarchy
of spottable child controls--namely, to the last spottable child to hold focus
before the focus moved outside of the container.  If the container in question
has never been focused, it passes focus to its first spottable child.

To define a container, set a control's `spotlight` property to `"container"`:

    {name: 'mycontainer', spotlight: 'container', components: [<A LIST OF spotlight:true CONTROLS>]}

In a way, containers may be thought of as the branches--and `spotlight:true`
controls as the leaves--of the Spotlight navigation tree.

<a name="5"></a>
## 5. NESTING ##

Spotlight containers may be nested.  The inner containers may be remembered as
"last focused children" of the outer ones; they act as conduits of focus passed
by the outer containers.

We have not found it useful to nest `spotlight:true` controls.  For now, these
controls act as the leaves of the spottable tree and do not conduct focus;
however, this behavior may be overridden on a per-control basis.

<a name="6"></a>
## 6. EVENTS ##

All Spotlight events are dispatched directly to the currently spotted control,
which may prevent bubbling to define custom behavior. (See [Extending Spotlight](#7).)

If Spotlight events are allowed to bubble to the application level, the app
responds to them with default behavior.

<br />
<a name="6_1"></a>
### 6.1. List of Spotlight Events ###

The following events are dispatched by the main Spotlight module:

#### 5-way mode

- **onSpotlightKeyDown**: Dispatched in response to `keydown`
- **onSpotlightKeyUp** Dispatched in response to `keyup`
- **onSpotlightLeft**: Dispatched in response to `onSpotlightKeyDown` event's
    bubbling to app level with keyCode 37
- **onSpotlightRight**: Dispatched in response to `onSpotlightKeyDown` event's
    bubbling to app level with keyCode 39
- **onSpotlightUp**: Dispatched in response to `onSpotlightKeyDown` event's
    bubbling to app level with keyCode 38
- **onSpotlightDown**: Dispatched in response to `onSpotlightKeyDown` event's
    bubbling to app level with keyCode 40
- **onSpotlightSelect**: Dispatched in response to `onSpotlightKeyDown` event's
    bubbling to app level with keyCode 13
- **onSpotlightFocus**: Dispatched when focus is transferred to a new control
    in response to events 3 through 7.
- **onSpotlightBlur**: Dispatched when focus is transferred away from a control
- **onSpotlightFocused**: Dispatched in response to `onSpotlightFocus` event's
    bubbling to app level right after its originator is set as current
- **onSpotlightScrollUp**: Dispatched when mousewheel event delta exceeds enyo.Spotlight.Scrolling.frequency (Default: 40)  
- **onSpotlightScrollDown**: Dispatched when mousewheel event delta exceeds -enyo.Spotlight.Scrolling.frequency (Default: 40)


#### Pointer mode

- **onSpotlightPoint**: Dispatched in response to `mousemove`, with coordinates
    matching the bounds of the spottable control

<br />     

<a name="6_2"></a>
### 6.2. Preventing or Allowing Default DOM events ###

By default, if a `keydown` event carries a 5-way keyCode (13, 37, 38, 39 or 40),
it will be prevented from bubbling and triggering default browser behavior.
This is done to disable default browser scrolling, because in the presence of
Spotlight, scrolling is handled using components such as Scroller.

Of course, there are some cases where you may want to allow the default browser
behavior.  For example, in text inputs, you may want to allow the cursor to move
to the next character when the right arrow key is pressed, without any
interference from JavaScript. 

For such cases, we have included an **Allow DOM Default** feature.  The events
`onSpotlightKeyDown`, `onSpotlightLeft`, `onSpotlightRight`, `onSpotlightUp`,
`onSpotlightDown` and `onSpotlightSelect` pass their handlers an event object
with an added `allowDomDefault` method:

    onSpotlightKeyDown: function(oSender, oEvent) {
        oEvent.allowDomDefault();
    }

In the above handler, if the Spotlight event is allowed to propagate, it will
allow the original DOM `keydown` to trigger default browser behavior. (See
[Figure A](#A)).
   
<br />  

<a name="6.3"></a>
### 6.3. Sequence of Spotlight Events ###

<a name="A"></a>
Figure A
--------
![Spotlight keyboard events](docs/chart_spotlight_5way_events.jpg)

[Figure A](#A) illustrates the sequence of events in Spotlight's 5-way mode.  At
each step, the sequence may be modified (bubbling may be prevented) on the level
of the currently focused control.  (See [Extending Spotlight](#7).)

For instance, when `onSpotlightKeyDown` is dispatched to the focused control,
the control may choose to prevent it from ever reaching the app level (where it
would be handled by Spotlight) and replace the default behavior with its own
custom handling.

If, however, `onSpotlightKeydown` is allowed to propagate and Spotlight
recognizes its keyCode as one of 5-way key codes, it dispatches an
`onSpotlight<5-Way Direction>` event back to the focused control.  At this point,
the control has, yet again, the option of overriding default behavior.

If `the onSpotlight<5-Way Direction>` event bubbles up to the app level,
Spotlight employs its [Nearest Neighbor Algorithm](kind.Spotlight.NearestNeighbor.js)
to figure out which spottable control is closest in the `<5-Way Direction>`.  It
then dispatches an `onSpotlightBlur` event to the current control (while also
removing the CSS *.spotlight* class), and an `onSpotlightFocus` event to the
nearest neighbor (while also adding the CSS *.spotlight* class).

If `onSpotlightFocus` is allowed to bubble from the newly focused control,
Spotlight sets its originator as `current`, and it officially becomes the
focused control.  In recognition of this fact, Spotlight dispatches
`onSpotlightFocused` to the control.

<br />

<a name="6.4"></a>
### 6.4. Accelerated keydown Events ###

<a name="B"></a>
Figure B
--------
![Accelerated keydown sequence](docs/chart_spotlight_accelerator_events.jpg)

While a key is depressed, the browser dispatches `keydown` events at equal (or
nearly equal) intervals.

Looking at [Figure B](#B), we can see that not all of these events affect the
application.  The function of the [Spotlight Accelerator](kind.Spotlight.Accelerator.js)
is to distribute events over time (according to its configuration).

Spotlight Accelerator may be configured via its array property,
`enyo.Spotlight.Accelerator.frequency`.  The default configuration is as
follows:

    //* Firing configuration. At n-th second use every frequency[n] subsequent keydown event
    frequency : [3, 3, 3, 2, 2, 2, 1], ...

This tells the Accelerator to do the following:

* In the first, second, and third seconds after the key is depressed, only let
    through every third `keydown` event.
* In the fourth, fifth, and sixth seconds, allow every second `keydown` event.
* In the seventh second and later, allow every `keydown` event.

This causes Spotlight focus to move across the screen with apparent acceleration
while a 5-way key is depressed.

<br />

<a name="6.5"></a>
### 6.5. Scroll Events ###

In response to browser's `mousewheel` events, spotlight dispatches `onSpotlightScrollUp` and `onSpotlightScrollDown`.    
However, there is a difference in how these are dispatched:  
The `mousewheel` event has a `wheelDeltaY` property which is translatable to degree of wheel rotation.
[Spotlight Scrolling](kind.Spotlight.Scrolling.js) accumulates `wheelDeltaY` values in a direction of rotation.
Once their cumulative value exceeds `enyo.Spotlight.Scrolling.frequency`, the `onSpotlightScrollUp` or `onSpotlightScrollDown` are
then dispatched and cumulative value is set back to 0.

This way Spotlight scrolling events are made to behave more like repeating keydown events, 
which is useful for controls like pickers and list scrollers that don't use smooth scrolling, instead animating to their next item.

Note: when Spotlight is in [Pointer Mode](#2), it treats first scroll event as first keyboard event, using it to come back from 
pointer mode and re-spot the item that was spotted previously.

<a name="7"></a>
## 7. EXTENDING SPOTLIGHT ##

There are two ways to extend spotlight functionality: using [Spotlight Decorators](#7.1) or [Extending existing controls](#7.2).

Each of the two has it's own pros and cons. Historically, we needed pros of both, so both were implemented.

<br />
<a name="7.1"></a>
### 7.1. Spotlight Decorators ###

Spotlight decorators is a way to add spotlight functionality to a control without altering it's code. 
This approach is useful when one or more of the following is true:

* You don't have access to alter control's code
* You prefer not to extend control
* You want to share same spotlight-related code among multiple kinds

<br />
**Steps to create and apply a decorator to a kind:**

1. Create a *`kind.Spotlight.Decorator.<My-Decorated-Kind>.js`* file  
2. Place this file into */decorators* folder inside of *spotlight* package
3. Add a line with your new filename to */decorators/package.js* file to create a dependency
4. In the file create a decorator kind:  
> `name: enyo.Spotlight.Decorator.<My-Decorated-Kind>,`  
5. Add `statics` block to write the rest of your code in it
6. Add static property *decorates* :   
> `decorates: My-Decorated-Kind,`  

7. Add event handler function with exactly the same name as a Spotlight event you wish to handle:
> `onSpotlightFocused: function(oSender, oEvent) { console.log(oSender, oEvent); },`   
8. Around your decorator you may add an *if* statement checking for *My-Decorated-Kind* being present in scope to avoid null reference exceptions:
> `if (window.onyx && onyx.<My-Decorated-Kind>) { /* decorator code */ }`


<br /><br />
When you run your code and focus your desired control, you should see console entries with an instance of the control (oSender) and `onSpotlightFocused` event instance (oEvent)

<a name="C"></a>
Figure C
--------
![Accelerated keydown sequence](docs/chart_spotlight_decorators.jpg)

Having a reference of control inside of event handler's scope allows you to access it's API and change it's state as desired in response to the event.
Just like in a regular event handler, returning TRUEish values will prevent it's default behavior defined in **[Spotlight](kind.Spotlight.js)**

If you need to persist a state of control between event handler calls you can use `oSender._spotlight` property object which is added by **[Spotlight](kind.Spotlight.js)** to every control it decorates:
> `oSender._spotlight.focusedTimes ++;`  

Please refer to **[existing Spotlight decorators](decorators)** package for samples.

<br />
<a name="7.2"></a>
### 7.2. Extending controls ###

Extending enyo controls to use **[Spotlight](kind.Spotlight.js)** functionality does not require any knowledge beyond enyo inheritance patterns and all described above in this document.
To do so, you simply subkind the kind you want to extend and define event handlers to handle spotlight events.
Please refer to **[Moonraker](https://github.com/enyojs/moonraker)** package for samples.

