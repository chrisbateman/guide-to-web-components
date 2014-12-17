# A No-Nonsense Guide to Web Components<br>Part 2: Practical Use &ndash; Browser Support and Other Challenges

This is Part 2 of a 3 part series.

- [Part 1: The Specs](01-the-specs.md)
- Part 2: Practical Use (Browser Support and Other Challenges)


## Introduction 

In Part 1 we learned how to code pure Web Components. But of course, adopting new web technologies is rarely painless, and Web Components are especially complicated. In this post, we'll look at the current state of browsers, polyfills, performance, accessibility, progressive enhancement, and options for practical implementations.


## Browser Support

For the latest status, check out [Are We Componentized Yet](http://jonrimmer.github.io/are-we-componentized-yet/) and [caniuse](http://caniuse.com). But here's the overall situation as of December 2014:

All browsers (except for IE) have implmented HTML Templates. For the remaining specs:

- **Chrome** completed and shipped all specs (enabled by default) by Chrome 36. **Opera** too.

- **Firefox** has implemented Custom Elements and Shadow DOM to some degree, but they are disabled by default behind the `dom.webcomponents.enabled` flag.

 Firefox will be shipping Custom Elements and Shadow DOM when they're satisfied and confident in the specs. They will **not** be shipping HTML Imports, as they want to see what the landscape looks like after ES6 modules are shipped and utilized ([12/15/2014](https://hacks.mozilla.org/2014/12/mozilla-and-web-components/)).

<!-- "It is not being shipped because we are not confident in the current specification, we offer it as an experimental API to collect feedback from developers" ([8/15/2014](https://bugzilla.mozilla.org/show_bug.cgi?id=889230#c9))-->

- **Safari** removed Google's Web Components code (in 2/2014) leftover from before Blink forked, and has yet to begin any new implementations. While not opposed to implementing something in the future, they're clearly not satisfied with the current specs. Their position is probably similar to that of Firefox.
> "There is no way I'm implementing the version [of Custom Elements] that's incompatible with ES6 classes" ([11/1/2014](https://twitter.com/ryosukeniwa/status/528774230532292609))

- **Internet Explorer**'s [status for all specs](https://status.modern.ie/) has been "under consideration" for about 6 months (as of 11/2014). They've been pulling out some surprises for IE 12, but so far it seems safe to say that Web Components won't be there.


### What's going on here?

When Chrome shipped Web Components &ndash; without a flag &ndash; [they](http://w3cmemes.tumblr.com/post/75694930387/googles-new-approach-to-standards-strives-for) [pretty](http://w3cmemes.tumblr.com/post/75701219807/ghost-of-christmas-yet-to-come-hath-an-ultimatum) [well](http://w3cmemes.tumblr.com/post/75681559100/chrome-team-rapidly-evolving-into-superior-race) [ticked](http://w3cmemes.tumblr.com/post/75717243955/live-from-the-uss-google-quick-update-on-shadow) [off](http://w3cmemes.tumblr.com/post/75719091769/you-non-googlers-misunderstand-everything-we-say) all the other browser makers, who felt that the specs weren't done baking yet. They're pretty diplomatic about it &ndash; they routinely say things like, "mail me privately for my feelings on the matter," because their feelings probably involve expletives.

How's it all going to pan out? Will the adjustments Safari and Firefox want be possible when Chrome has already shipped? Will everyone get on the same page anytime soon? I wish I had answers to these questions. Web standards are complicated things. Tune in next year (or the next) to find out!


## Polyfills

In [Part 1](01-the-specs.md), we looked at an example of a pure, native Web Component. If you were hoping that you could write code like that, plug in a polyfill, and be good to go &ndash; you better sit down, because I've got some bad news for you:

It ain't gonna happen. Some parts of Web Components simply aren't possible/reasonable to polyfill.

But let's examine each spec individually.

[webcomponents.js](https://github.com/webcomponents/webcomponentsjs) (which was previously part of [Polymer](https://www.polymer-project.org/)) is definitely the biggest game in town when it comes to polyfills, so that's mainly what we'll be talking about.


### Custom Elements ([native support](http://caniuse.com/#feat=custom-elements))

Custom Elements is the easist to polyfill of all the specs &ndash; down to IE 9 and Android 2.2, if you use [document-register-element](https://github.com/WebReflection/document-register-element) for the polyfill (and it's only 3KB gzipped too!). The webcomponents.js polyfill works down to Android 4.1.


### HTML Templates ([native support](http://caniuse.com/#feat=template))

The webcomponents.js polyfill works down to IE 9 and Android 2 (and aren't needed in other modern browsers).

**Caveat**: Polyfilled templates aren't truly inert &ndash; resources like images will still download.


### HTML Imports ([native support](http://caniuse.com/#feat=imports))

The webcomponents.js polyfill works down to IE 9 and Android 4.4 (some things like CSS references work down to Android 4.1, but there's other bugginess), and in other modern browsers.

**Caveat #1**: Polyfilled imports load asynchronously, even if you didn't add the `async` attribute. 

**Caveat #2**: They load via XHR &ndash; which isn't great for performance (compared to native Imports).

**Caveat #3**: `document.currentScript`, which is needed in the import to access templates (or other elements) in the import, can't be polyfilled. It is, however, shimmed with `_currentScript`. So, to write code that works in both supported and polyfilled browsers, you must do this:

```javascript
document.currentScript || document._currentScript;
```

### Shadow DOM ([native support](http://caniuse.com/#feat=shadowdom))

It is not reasonable/possible to polyfill Shadow DOM, thanks to its fancy encapsulation features. You just can't fake the behavior of a shadow root.

The webcomponents.js code *attempts* to shim *some* of the encapsulation features by way of rewriting your CSS. It polyfills Shadow DOM's JavaScript API, querySelector and other DOM APIs to behave (hopefully) as they should, and it moves elements selected with the `<content>` tag to where the shadow root would be.

**Caveats:**

- CSS rules in the page will still apply to elements in the (fake) shadow root. It's like everything gets a `/deep/`.
- To use `::shadow` and `/deep/` CSS rules in the page, you must add the `shim-shadowdom` attribute to the `<style>` or `<link>`.
  - Even then, `::shadow` rules will behave like `/deep/` rules anyhow.
- To include CSS in a shadow root, you have to add some JS to the component to check whether the Shadow Dom shim is in effect, and if so, grab the css text, run it through the a shimming function, add the resulting CSS text to the document, and delete the original CSS.
  - You don't have to do all this if you're using Polymer and its wrapper/syntax. But native syntax doesn't cut it.
- `::content` rules in the shadow root will apply to everything in the shadow root &ndash; not just children of `<content>` elements.
- When using `<content>`, DOM hierchy will be different in polyfilled browsers. Normally, elements selected by `<content>` would be children of the root element, but in polyfilled browsers they will be children of the `<content>`'s parent. You can certainly work around this &ndash; but it's an additional thing to keep in mind as you write your component's JS.


### Polyfill Sizes

webcomponents.js includes a "lite" file which includes Custom Elements, Templates, and Imports at 9KB, gzipped.

Adding Shadow DOM brings the polyfill total up to 30KB, gzipped (it's 103KB minified, without gzipping, by the way. TJ VanToll has written about [why it's so large](http://developer.telerik.com/featured/web-components-arent-ready-production-yet/)).


## Performance & HTML Imports

An HTML Import for a component contains individual links to all of the component's dependencies. This really isn't consistent with today's practice of concatenating JS and CSS files, to keep the number of HTTP requests down.

Imports were designed with HTTP/2 in mind (which basically makes it fine to skip concatenation by way of multiplexing). Unfortunately, not every hosting provider, CDN, or server supports it yet. Browser support for SPDY (the predecessor to HTTP/2's multiplexing) [isn't too shabby](http://caniuse.com/#search=spdy), but there are still some issues (mostly older IE, and IE 11 on Windows 7-). If you're ready to go all HTTP/2 &ndash; you're in good shape to use HTML Imports.

If not &ndash; Polymer does have a tool called [Vulcanize](https://github.com/polymer/vulcanize) that concatenates Import files &ndash; but there are some gotchas:

- Since all the Imports' contents will get lumped together, you must ensure there are no duplicated element IDs (mainly used for templates) between Imports.
- `document.currentScript.ownerDocument` will point to the importing page's document, rather than the (original) import document.
- Anything in the import other than templates, CSS, and JS will be removed. Which is probably fine if you're just using Imports for Web Components.

**Bottom line:** The general point of Imports is to give you an *easy* way to get a component's dependencies on a page. If you already have a solution for that &ndash; you might be fine to keep with it. The biggest thing you'll miss is the ability to store HTML Templates in the Import. But at least [ES6 template strings](http://tc39wiki.calculist.org/es6/template-strings/) will make templates in JS less painful.


## Accessibility

Accessibility with Web Components really isn't any different than accessibility with any other kind of UI components. 

Yes, Web Components can make it convenient to reimplement native elements, but you certainly don't *have* to, and people already do that without "Web Components" when the native element doesn't provide the flexibility they want. We've seen custom dropdowns, range sliders, radios, checkboxes, buttons, and more. Some accessible, and some not so much.

The same principle applies, whether it's a "Web Component" or not: **You must add the appropriate accessibility features to anything you build (whether you're reimplementing a native element or not).** And that often means [more work](http://www.paciellogroup.com/blog/2014/08/what-aria-does-not-do/) than just adding an ARIA role. Here's a [handy checklist](http://www.paciellogroup.com/blog/2014/09/web-components-punch-list/), courtesy of Steve Faulkner.

And don't forget &ndash; while Custom Elements lets you create new elements:

```markup
<ul>
    <crazy-li role="listitem">First</crazy-li>
</ul>
```

It also allows you to extend native elements &ndash; saving you the trouble of reimplementing built-in accessibility features:

```markup
<ul>
    <li is="crazy-li">First</li>
</ul>
```

**Bottom line:** if you can write an accessible regular component, then you can write an accessible Web Component. Whatever you're doing &ndash; make it accessible!


## Progressive Enhancement

If you haven't noticed yet &ndash; Web Components rely on JavaScript. Substantially. Imports is the only part that really works without JS. There are differing opinions on progressive enhancement, but let's just examine a couple approaches from a high level, so that you can make the best decision for your situation:

### Component renders its own internal HTML.

This is kind of the assumed default with Web Components.

**Good:** 

- Page markup is clean, understandable, and simple
- Components' internal HTML can be easily updated on all pages

**Bad:** 

- No JS = empty component
- A synchronous-loading component (in the `<head>`) will slow down the page's initial render time
- An asynchronous-loading component will pop into existence after the initial render (Flash of Loaded Component - FOLC? Or FOCL?)

### Server includes the component's internal markup.

This still lets you take advantage of Custom Elements' lifecycle callbacks and element functions, while maintaining progressive enhancement.

**Good:**

- Faster initial render
- No FOLC
- All markup is there if JS fails

**Bad:** 

- Page markup is messy again
- Updating a component's HTML means updating every page's markup (unless you build something server-side to automate it)
- Shadow DOM will probably need to sit this one out
- The CSS needs to load in the `<head>`, of course. So if you're using an Import, it'll need to be synchronous (and maybe load its JS *asynchronously*, for performance).
  - But remember the Imports polyfill doesn't do synchronous loading.


## Conclusions

Here are my conclusions, based on the topics we've discussed:

- **Custom Elements** are helpful, and fairly easy to polyfill.
- **HTML Imports** have too many caveats right now (particularly around performance), and Firefox isn't going to do it (I kind of doubt Safari will either). I agree with them that it's too soon to try to get this solution right. In the meanwhile &ndash; our current solutions for including resources will have to do.
- Polyfilled **Shadow DOM** has *way* too many caveats, and the polyfill is big (and especially slow on mobile devices). Shadow DOM will be useful someday when broad browser support is available.

So I'm left with Custom Elements (this was TJ's [conclusion](http://developer.telerik.com/featured/web-components-ready-production/) as well). They're the only spec that's polyfillable on all the older platforms I'd like to support (IE 9, Android 4.3 and below, etc.), and I love the lifecycle callbacks and the "semantic" and clean way you use them on a page.

My suggestion to you is to try it out &ndash; build a component with Custom Elements, and see how it works in your environment. When Shadow DOM is ready, you can add it &ndash; so just keep that potential future state in mind as you build components.


### A note on frameworks/libraries

Libaries like [Polymer](https://www.polymer-project.org/) were developed to solve a number of common tasks related to Custom Elements (and the other specs). Things like easy attribute binding, smarter templating, and events.

However, they also seem to violate one of the objectives of Web Components, which is reusability. If you want to build a component that can be reused in a variety of environments, keeping your dependencies to a minimum is usually a good thing. I'm not sure I'm comfortable with forcing another largeish (Polymer is ~37KB, gzipped) dependency on everyone who might want to use my component. 

*However*, if you want to develop components to be used in environments that *you* control, I'd feel much better about a library like Polymer, and it'd probably be fairly helpful. [X-Tag](http://x-tags.org/) is another alternative which provides a neat wrapper for creating Custom Elements (and they don't even bother with Shadow DOM, which is perfect).
