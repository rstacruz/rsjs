# rsjs

**Reasonable Standard\* for JavaScript Structure.**

:construction: This document is a work in progress. Also see [rscss](https://github.com/rstacruz/rscss), a document on CSS conventions that follows a similar line of thinking.

(`*`: or **S** can also stand for "suggestions".)

<br>

## Problem

For a typical non-[SPA] website, it will eventually be apparent that there needs to be conventions enforced for consistency. While [styleguides][abnb] cover coding styles, conventions on namespacing and file structures are usually absent.

[abnb]: https://github.com/airbnb/javascript
[SPA]: https://en.wikipedia.org/wiki/Single-page_application

<br>

#### The jQuery soup anti-pattern
You will typically see Rails projects with behaviors randomly attached to classes, such as the problematic example below.

```html
<script src='blogpost.js'></script>
<div class='author footnote'>
  This article was written by
  <a class='profile-link' href="/user/rstacruz">Rico Sta. Cruz</a>.
</div>
```

```js
$(function () {
  $('.author a').on('hover', function () {
    var username = $(this).attr('href');
    
    showTooltipProfile(username, { below: $(this) });
  });
});
```

<br>

#### What's wrong?

This anti-pattern leads to many issues, which rsjs attempts to address.

 * **Ambiguious sources:** It's not obvious where to look for the JS behavior. (Is the handler attached to *.author*, *.footnote*, or *.profile-link*?)

 * **Non-reusable:** It's not obvious how to reuse the behavior in another page. (Should *blogpost.js* be included in the other pages that need it? What if that file contains other behaviors you don't need for that page?)

 * **Lack of organization:** Making new behaviors gets confusing. (Do you make a new `.js` file for each page? Do you add them to the global `application.js`? How do you load them?)

<br>

#### About Rails

This styleguide assumes Rails conventions of concatenating .js files. These don't apply to loaders like Browserify or RequireJS.

<br>

## Structure

#### Think in component behaviors

Think that a piece of JavaScript code to will only affect 1 "component", that is, a section in the DOM.

There files are "behaviors": code to describe dynamic JS behavior to affect a block of static HTML. In this example, the JS component `animate-links` only affects a certain DOM subtree, and is placed on its own file.

```html
<div class='main-navbar' role='animate-links'>
  <a href='/'>Home</a>
  <ul>
    <li><!-- links go here --></li>
  </ul>
</div>
```

```js
// behaviors/animate-links.js
// shows links on hover

$(document).on('hover', '[role~="animate-links"]', function () {
  $(this).addClass('-show-links');
});
```

<br>

#### One component per file

Each file should a self-contained piece of code that only affects a *single* element type.

Keep them in your project's `behaviors/` path. Name these files according to the roles ([see below](#usetheroleattribute)) or class names they affect.

```
└── javascripts/
    └── behaviors/
        ├── collapsible-nav.js
        ├── avatar-hover.js
        ├── popup-dialog.js
        └── notification.js
```

<br>

#### Load components in all pages

Your main .js file should be a concatenation of all your `behaviors`.

You should be able to safely load all behaviors for all pages. Since your files's behaviors are localized to a certain "component", they will not have any effect unless the component it services is present the page. In Rails, this can be accomplished with `require_tree`.

```js
// js/application.js
/*= require_tree ./behaviors
```

<br>

#### Prefer event delegation

*Optional:* Instead of using `document.ready` to bind your events, consider using [jQuery event delegation][del] instead.

This allows you to bind behaviors to, say, modal windows, where the element may not be present when the document loads. There are cases wherein document.ready is necessary. If there's no other way to implement it, this should be fine.

[del]: http://learn.jquery.com/events/event-delegation/

```js
/* bad */
$(function () {
  $('[role~="collapsible-rows"]').on('hover', function () {
  });
});
```

```js
/* better */
$(document).on('hover', '[role~="collapsible-rows"]', function () {
});
```

<br>

#### Use the role attribute

*Optional:* It's prefered to mark your component with a `role` attribute.

You can use ID's and classes, but this can lead to confusion because your class name now means 2 things, and it makes it difficult to re-style if need be.

```html
<!-- bad -->
<div class='user-info'>...</div>
$('.user-info').on('hover', function() { ... });
```

```html
<!-- ok -->
<div class='user-info' role='avatar-popup'>...</div>
$('[role~="avatar-popup"]').on('hover', function() { ... });
```

<br>

#### Don't overload class names

If you don't like the `role` attribute and prefer classes, don't add styles to the classes that your JS uses. For instance, if you're styling a `.user-info` class, don't attach an event to it; instead, add another class name (eg, `.js-user-info`) to use in your JS.

This will also make it easier to restyle components as needed.

```html
<!-- bad:
  is the JavaScript behavior attached to .user-info? or to .centered?
  This can be confusing developers unfamiliar with your code.
-->
<div class='user-info centered '>...</div>
$('.user-info').on('hover', function() { ... });
```

```html
<!-- better:
  this makes it more obvious to find the source of the behavior.
-->
<div class='user-info js-avatar-popup'>...</div>
$('.js-avatar-popup').on('hover', function() { ... });
```

<br>

#### Avoid side effects

Make sure that each of your JavaScript files will not throw errors or have side effects when the element is not present on the page. This allows you to include all your behavior files in all parts of the site without fear that it might cause unintended behavior.

```js
/* bad: can make scrolling sluggish on pages without .collapsible-nav */
$('html, body').on('scroll', function () {
  var $nav = $("[role~='collapsible-nav']");
  var isScrolled = $(window).scrollTop() > $nav.height;
  $nav.toggleClass('-hidden', !isScrolled);
});
```

```js
/* better: disable behavior when not around */
$('html, body').on('scroll', function () {
  var $nav = $("[role~='collapsible-nav']");
  if (!$nav.length) return;
  
  // ...
});
```

```js
/* also better: don't bind the event when the element isn't present */
$(function () {
  var $nav = $("[role~='collapsible-nav']");
  if (!$nav.length) return;
  
  $('html, body').on('scroll', function () {
    // ...
  });
});
```

<br>

## Third party libraries

If you want to integrate 3rd-party scripts into your app, consider them as component behaviors as well. For instance, you can integrate [select2.js] by affecting only `.js-select2` classes.

[select2.js]: https://github.com/select2/select2
[wow.js]: http://mynameismatthieu.com/WOW/

```js
...
└── javascripts/
    └── behaviors/
        ├── colorpicker.js
        ├── select2.js
        └── wow.js
```

```js
// select2.js -- affects `.js-select2`
$(function () {
  $(".js-select2").select2();
})
```

```js
// wow.js -- affects `.wow`
$(function () {
  new WOW().init();
})
```

<br>

#### Separate your vendor libs

Keep your 3rd-party libraries in something like `vendor.js`.

This will have browsers cache the 3rd-party libs separately so users won't need to re-fetch them on your next deployment.

It also makes it easier to create new app packages should you need more than one (eg, one for your public pages and another for private dashboards).

```js
/* vendor.js */
//= require jquery
//= require jquery_ujs
//= require bootstrap
```

```js
/* application.js */
//= require_tree ./helpers
//= require_tree ./behaviors
```

<br>

## Namespacing

#### Don't pollute the global namespace

Place your publically-accessible classes and functions. Prefer to put them in an object, like `App`.

```js
if (!window.App) window.App = {};

App.Editor = function() {
  // ...
};
```

<br>

#### Organize your helpers

If there are functions that will be reusable across multiple behaviors, put them in a namespace. Place these files in `helpers/`.

```js
/* helpers/format_error.js */
if (!window.Helpers) window.Helpers = {};

Helpers.formatError = function (err) {
  return "" + err.project_id + " error: " + err.message;
};
```

<br>

## Turbolinks

These are guidelines to make your JS structure more friendly to Turbolinks. They may still be of benefit even without Turbolinks, however.

#### Make document.ready calls idempotent

Make sure your `$(function(){...})` handlers can be ran multiple times in a page without any side effects. This is great so you can use jQuery.turbolinks.

```js
$(function () {
  $('[role~="fizzle"]').each(function () {
    if ($(this).data('loaded')) return;
    
    $(this).fizzle().data({ loaded: true });
  });
});
```

#### Tag your event handlers

Makes things neater, and allows you to trigger them later on.

#### Clean up if needed

Listen to `page:unload` to clean up anything to prepare the DOM for the next page.

<br>

## Conclusion

This document is a result of my own trial-and-error across many projects, finding out what patterns are worth adhering to on the next project.

This document prioritizes developer sanity first. The approaches here may not have the most performant, especially given its preference for event delegations and running `document.ready` code for pages that don't need it. Regardless, the pros and cons were weighed and ultimately favored approaches that would be maintainable in the long run.

The guidelines outlined here are not a one-size-fits-all approach. For one, it's not suited for [single page applications][SPA], or any other website relying on very heavy client-side behavior. It's written for the typical conventional web app in mind: a collection of HTML pages that occasionally need a bit of JavaScript to make things work.

As with every other guideline document out there, try out and find out what works for you and what doesn't, and adapt accordingly.

#### Further reading

- [kossnocorp/role](https://github.com/kossnocorp/role#use-role-attribute-ftw)'s explanation on using the `role` attribute
