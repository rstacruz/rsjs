# rsjs

**Reasonable Standard\* for JavaScript Structure.**

:construction: This document is a work in progress. Also see [rscss](https://github.com/rstacruz/rscss), a document on CSS conventions that follows a similar line of thinking.

(`*`: or **S** can also stand for "suggestions".)

<br>

## Problem

For a typical non-[SPA] website, how would you organize your JavaScript behaviors as you scale? You will typically see Rails projects with behaviors randomly attached to classes, such as the problematic example below.

[SPA]: https://en.wikipedia.org/wiki/Single-page_application

```html
<script src='blogpost.js'></script>
<div class='author footnote'>
  This article was written by <a class='profile-link' href="/user/rstacruz">Rico Sta. Cruz</a>.
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

This anti-pattern leads to many issues:

 * **Ambiguious sources:** If you need to edit the tooltip behavior later on, it's not obvious where to look first. Is the behavior attached to `.author`, `.footnote`, or `.profile-link`?

 * **Non-reusable:** If you would like to reuse this behavior in a different page, it's not obvious how to do that either. Should `blogpost.js` be included in the other pages that need it? What if that file contains other behaviors you don't need for that page?

 * **Lack of organization:** Without any imposed standard on where to place JS behaviors, making new behaviors gets confusing. Do you make a new `.js` file for each page? Do you add them to the global `application.js`? How do you load them?

This styleguide assumes Rails conventions: that is, your final script builds are concatenated JavaScript files with no loaders like Browserify or RequireJS. You can adopt these guidelines to any other tech stack that follows this pattern.

<br>

## Structure

### Think in component behaviors

Think that a piece of JavaScript code to will only affect 1 "component", that is, a section in the DOM. There files are "behaviors": code to describe dynamic JS behavior to affect a block of static HTML.

In this example, the JS component `animate-links` only affects a certain DOM subtree, and is placed on its own file.

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

----

### One component per file

Each file should a self-contained piece of code that only affects a *single* element type. Keep them in your project's `behaviors/` path. Name these files according to the role ([see below](#usetheroleattribute)) or class names they affect.

```
└── javascripts/
    └── behaviors/
        ├── collapsible-nav.js
        ├── avatar-hover.js
        ├── popup-dialog.js
        └── notification.js
```

----

### Load components in all pages

You should be able to safely load all behaviors for all pages. Since your files's behaviors are localized to a certain "component", they will not have any effect unless the component it services is present the page. In Rails, this can be accomplished with `require_tree`.

```js
// js/application.js
/*= require_tree ./behaviors
```

----

### Prefer to use event delegation

Instead of using `document.ready` to bind your events, consider using [jQuery event delegation][del] instead. This allows you to bind behaviors to, say, modal windows, where the element may not be present when the document loads.

There are cases wherein document.ready is necessary. If there's no other way to implement it, this should be fine.

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

----

### Use the role attribute

*Optional:* It's prefered to mark your component with a `role` attribute. You can use ID's and classes, but this can lead to confusion because your class name now means 2 things, and it makes it difficult to re-style if need be.

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

----

### Don't overload class names

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

----

### Avoid side effects

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

----

## Third party libraries

If you want to integrate 3rd-party scripts into your app, they can be considered as component behaviors as well. For instance, you integrate [select2.js] by affecting only `.js-select2` classes.

For libraries that prescribe their own conventions (eg, [wow.js] affects `.wow` by default), make sure that your 3rd-party scripts also "think in components" as well. That is, it should have no side effects if their classes are not available in the page, and it should produce no conflicts with other components you may have on your site.

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

----

### Separate your vendor libs

Separate your 3rd-party libraries into something like `vendor.js`.

Rails asks you to load by minify everything into `application.js` by default, including 3rd-party libraries. This is not favorable in the long term: your application code will eventually bloat, resulting in large files that will need to be re-cached on your next deployment.

By separating your app and vendor code, you can have your 3rd-party libraries cached separately, without asking users to re-fetch them on your next deployment. It also makes it easier to create new app packages: should you need more than one `application.js` (eg, one for your public pages and another for private dashboards), they can share a common `vendor.js`.

```js
/* bad */
//= require jquery
//= require jquery_ujs
//= require bootstrap
// ...
// (your actual app code here)
```
 
```js
/* better: separate them into ™ files */

/* vendor.js */
  //= require jquery
  //= require jquery_ujs
  //= require bootstrap

/* application.js */
  //= require_tree ./behaviors
```

----

## Conclusion

This document is a result of my own trial-and-error across many projects, finding out what patterns are worth adhering to on the next project.

The guidelines outlined here are not a one-size-fits-all approach. For one, it's not suited for [single page applications][SPA], or any other website relying on very heavy client-side behavior. It's written for the typical conventional web app in mind: a collection of HTML pages that occasionally need a bit of JavaScript to make things work.

As with every other guideline document out there, try out and find out what works for you and what doesn't, and adapt accordingly.
