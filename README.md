## *BeCSS* :: A lazy CSS code pattern for UI components

_**Behavioural CSS**_ is a very simple pattern for writing and maintaining CSS style declarations specifically for modular UI components.

One very well known software design pattern for writing abstracted, modular code is having a clear [separation of concerns](https://en.wikipedia.org/wiki/Separation_of_concerns).  Yet almost everywhere, we see JavaScript code on websites or web applications modifying a DOM elements' ```class``` attribute just to affect how it is rendered or presented, yet this clearly breaks a core premise of [SoC](https://en.wikipedia.org/wiki/Separation_of_concerns) as we have tied our underlying JavaScript implementation to a CSS visual representation of our UI component.

Whilst there are are number of CSS patterns that attempt to address the issues with CSS class overuse and naming conventions, none of them really address the primary problem.

### The pattern
**Behavioural CSS** addresses the overuse and naming issues around CSS classes as well their lack of semantic relevance, and ensures a clear separation of concerns in our code.  The core premise is simply to be as lazy as possible and maximise the use of the browsers CSS engine by following two simple rule:


### The rules
This in practice, leads us to the two very simple rules in this pattern.
1)  *JavaScript code should only ever describe the **behaviour and state** of the UI not its' visual appearance.*
2)	*JavaScript code should never manipulate the class attribute on DOM elements*

### Describing UI state & behaviour
So the obvious question then becomes how do we apply CSS selectors to DOM elements without using CSS classes - the simple answer is we use DOM attributes instead.

At its' simplest, code like this:

```css
button.disabled {
	border: none;	
}
```
Becomes:
```css
button[disabled] {
	border: none;	
}
```
So why would we do this?

Because applying a CSS class of ```.disabled``` has no semantic meaning, but the ```disabled``` DOM attribute has very a specific semantic meaning, that other parts of your application can infer information about.  If someone else wanted to disable the button, they don't need to know anything about your CSS classes, as well as now, we automatically prevent any uneeded ```click``` events.

How about a toggle button that has a depressed state?

Many developers would add/remove a class probably called ```.toggle``` or something similar, but why don't we make it easy on ourselves and use a semantic attribute that might be written as:
```css
button[aria-pressed="true"] {
	background: #CCCCCC;
}
```
Now all our JavaScript button code does is to set the attribute as to whether it is in the ```pressed``` state or not, anyone maintaining our code in the months or years to come will have a very clear idea of the intent of our code, yet we haven't described anything about that buttons' visual representation.  The real +1 is that properly written, now we get [Accessibility](https://developer.mozilla.org/en-US/docs/Web/Accessibility/ARIA) support for free, because ```aria-pressed``` has a clearly defined semantic meaning.

Let's take this idea further.

What do you do when you have a story in sprint that says that your UX designer wants a red border around text input fields that are invalid, as long as it is not a readonly field?  Do you breakout your JS editor, check the state, toggle your CSS classes, run a build on all your JS code and re-deploy?  Well you could... or you could just say:
```css
input[type="text"]:not([readonly]):invalid {
	border : 1px solid red;
}
```
No JavaScript code was harmed in the deployment of this requirement, and in 6 months time, everyone reading that will know what you meant, and we've let the CSS engine do the heavy lifting for us.

### Custom attributes
Ok so what about when there is no global DOM attribute, element attribute or ARIA attribute that works for you?  Simple, you make one up (yes just like your CSS classes but...), our custom attributes go into the HTML5 ```data-``` namespace.  We can access all of these directly via: 

```javascript
element.dataset.myCustomAttribute = something;
```

Notice as well, by using an attribute we have two parameters to describe our elements' behaviour and state, the attribute name and its' value.  By using a set of well named semantic attributes that perhaps we have agreed with the whole team, we now have a rich way to express application state throughout our entire app.

```css
#calendar[data-selecteddate="today"] {
	background: red;
}
```
Or maybe even:
```css
#calendar[data-selecteddate$="2017"] {
	background: blue;
}
```
or perhaps:
```css
#price[data-currency="GBP"] :: before {
	content : '£';
}
```

The list of custom attributes and their values can be endless - whatever fits your needs.

### Look Ma - no hands!
What about something a little more juicy & complex?  How about an input range element that has a numbered badge showing the value multiplied by two?  By expressing the behavioural state of our widget, we use a custom attribute and few lines of behavoural css.

```html
<input type="range" value="0" min="0" step="1" max="10" data-doubled="0">
```

```css
input[type="range"] {
	margin-top: 40px;
	position: relative;
}

input[type="range"]::after {
    content: attr(data-doubled);
    position: absolute;
    width: 28px;
    height: 28px;
    top: -35px;
    right: 0px;
    font-size: 20px;
    text-align: center;
    border: 1px solid black;
    border-radius: 40px;
}

```
```javascript
document.querySelector('input').addEventListener('input', function(event){
	event.target.dataset.doubled = event.target.valueAsNumber *2;
}, false);
```

Notice how we have not said anything about how the ```doubled``` value is rendered?  Tomorrow the UI team could render it as a speech bubble popup, none of our code needs to change.  We've expressed the state of the element and left the CSS/presentation layer to do its' thing.


### *How I Learned to Stop Worrying and Love the DOM*
A great many people, spend huge amounts of time worrying about storing, updating and then restoring the states of their apps and components.  When the states and current behaviours in each component are stored in your custom DOM attributes, guess what...?
```javascript
JSON.stringify(element.dataset);
```
You'll get back a neat keyed JSON string of all your attributes and values that you can put where you want, and get back when you need to.


### Become 'Classless'
The only people that should be using CSS classes are the UI team applying actual themes, and whose class selectors rarely if ever encounter any app states or behaviour - except perhaps one as the root selector that might be:
```css
body[data-theme="mytheme"] ...
```

So go ahead and try it, freeing yourself from the pain and anguish of messing around with CSS in your actual code is liberating, it forces you to consider the states and behaviours that you actually need to expose and ensures UI decisions today are as far as possible abstracted away from the underlying behaviour and state of the component itself.
