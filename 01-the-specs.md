# A No-Nonsense Guide to Web Components, Part 1: The Specs

This is Part 1 of a 3 part series.

- Part 1: The Specs
- [Part 2: Practical Use (Browser Support and Other Challenges)](02-practical-use.md)


## Introduction

This is a crash course for getting familiar with Web Components. It strives to be concise, rather than exhaustive. There a lot of other great resources available on the topic: check out HTML5 Rocks' [tutorials](http://www.html5rocks.com/en/search?q=web+components) and [this massive list of resources](https://github.com/mateusortiz/webcomponents-the-right-way).


## Why Web components?

**First** &mdash; they're easy to manage &ndash; thay can instantiate themselves and clean up after themselves.

**Second** &mdash; Simple, declarative usage means they're easy to include and configure:
```markup
<link rel="import" href="my-dialog.htm">

<my-dialog heading="A Dialog">Lorem ipsum</my-dialog>
```

**Third** &mdash; they're modular and reusable. A standard format for building, implementing, and interacting with UI components means that it's easy to use them across different frameworks and environments.

**Fourth** &mdash; they can provide encapsulation for components' styles and HTML. So they play nice with other styles and things happening on a page.


## The Specs

Web Components are made up of 4 separate specifications. They go together nicely, but you don't have to use them all &ndash; you can pick and choose based on your situation. **Custom Elements** and **Shadow DOM** are most important; **HTML Imports** and **Templates** are really just handy. 

We're sticking to native code for now (no polyfills), so be sure to use a browser that [supports](http://caniuse.com/) the spec, when looking at the demos.


### Custom Elements

Custom Elements are the heart of Web Components. This API lets you create new elements, add public methods to them, and gives you 4 lifecycle callbacks to manage them.

All you have to do is create an object to be used as your element's prototype, add the callbacks to it, and then register it with a hyphenated name (all custom elements must have a hypen &ndash; that's how you know they're not native elements).

```javascript
// <my-element></my-element>

var myProto = Object.create(HTMLElement.prototype);

// Lifecycle callbacks
myProto.createdCallback = function() {
    // initialize, render templates, etc.
};
myProto.attachedCallback = function() {
    // called when element is inserted into the DOM
    // good place to add event listeners
};
myProto.detachedCallback = function() {
    // called when element is removed from the DOM
    // good place to remove event listeners
};
myProto.attributeChangedCallback = function(name, oldVal, newVal) {
    // make changes based on attribute changes
};

// Add a public method
myProto.doSomething = function() { ... };


document.registerElement('my-element', {prototype: myProto});
```

This is fantastic, because it means that your components can both self-initialize and self-destroy.

Let's say you have a page where a user action can open up a new widget. This particular widget adds some keyboard listeners to the page to check for shortcuts. Today, you'd probably call a JS function to initialize the widget. And when the user closes it, you'd need to call a destroy function that would remove the event listeners. Because you don't want those listeners to stick around &ndash; using up memory and continuing to take action on events.

With Custom Elements &ndash; you (or your framework) don't have to worry about those details. Just insert the element into the DOM to initialize it, and when you remove it from the DOM, it can clean up after itself. Awesome!

You can also extend native elements, like this:
```
// <input is="my-input">
document.registerElement('my-input, {
    prototype: myProto,
    type: 'input'
});
```

[Custom Elements Demo](https://chrisbateman.github.io/guide-to-web-components/demos/custom-elements.htm)


### Shadow DOM

Shadow DOM encapsulates elements. It allows you to hide a number of elements inside of an element - much like browsers do with their native UI elements (e.g. the controls in a `<video>`). This prevents other code on the page from accidentally messing with your element - and vice-versa.

```javascript
var shadowRoot = element.createShadowRoot();
shadowRoot.appendChild(whatever);
```

You can add CSS inside a shadow root, and it won't select elements outside of the shadow root (Note that you can't put `<link>` tags in a shadow root. To reference an external stylesheet, use `@import` in a `<style>` tag). To target the element holding the shadow root, just use the `:host` selector.

And conversely, CSS selectors on the page won't select elements inside a shadow root (and that goes for querySelector too). But those elements will still inherit inheritable properties (like font-family).

If the page needs to style something that's in a shadow root, it's still possible, and the intention of your CSS will be very obvious (which is a good thing):

```css
my-element::shadow p {
    /* selects <p> tags in shadow roots of <my-element>'s */
}
body /deep/ p {
    /* selects all <p> tags - in shadow roots or not */
}
```

There's one other thing you can do with Shadow DOM: leave an element's contents outside of the shadow root &ndash; so they're still accessible to the page &ndash; but visually reflow them as if they *were* in the shadow root. Just add a `<content>` element, and it will reflow any children of the root element:

```markup
<my-element>
    #shadow-root#
        <content></content>
        <p>one</p>
    #/shadow-root#
    <p>two</p>
    <p>three</p>
</my-element>
```

In that example &ndash; you'll see the lines orderd as "two three one" but only the "one" is actually encapsulated in the shadow root.

Note that if there wasn't a `<content>` element, the "two" and "three" paragraphs would not be visible (a shadow root hides the other children).

[Shadow DOM Demo](https://chrisbateman.github.io/guide-to-web-components/demos/shadow-dom.htm)


### HTML Imports

Imports give you a single place to put the styles, scripts, and templates required for a component, so pages only need to include one thing.

```markup
<link rel="import" href="dialog.htm">
```

CSS in an import will apply to the page, and scripts will execute in the usual global context.

Other regular HTML elements in the import will **not** be visible on the page or accessible to things like querySelector. Though you can access anything in the import if you need to, like this: `linkElement.import.querySelector('#template');`

It's very important to note that Imports will block the rendering of your page (same as plain JS and CSS resources do) &ndash; unless you add the `async` attribute (you can listen for the `load` event). Helpfully, scripts in an async import will still execute in order.

[HTML Imports Demo](https://chrisbateman.github.io/guide-to-web-components/demos/html-imports.htm)


### HTML Templates

For storing HTML templates, you may have used strings in JS, or perhaps a `<script>` tag with a non-standard `type`. But now there's a dedicated element for it:

```markup
<template id="MyTemplate">
  <div>Some stuff</div>
</template>
```

Using it is pretty straightforward:

```javascript
var clone = document.importNode(templateNode.content, true); // 2nd parameter for "deep" clone
// now you can append the clone wherever you like
```

That's it &ndash; nothing too fancy.

And when it comes to Web Components &ndash; Templates are pretty useless without Imports (where else would you put them?).

[HTML Templates Demo](https://chrisbateman.github.io/guide-to-web-components/demos/html-templates.htm)


## All Together Now

Now that we've looked at the pieces in isolation, let's see how they look together. This is a super-basic example of how you might build a Web Component without any frameworks or polyfills.

[Web Components Demo](https://chrisbateman.github.io/guide-to-web-components/demos/all-together-now.htm)


Fun stuff, but don't get too excited just yet. In [Part 2](02-practical-use.md) we'll talk about using Web Components in real life.
