## Events

#### Example of each()

Here's a more concrete example of the jQuery.each pattern.

```js
$(function() {
  $('[role~="expanding-menu"]').each(function() {
    // Some cached lookups.
    var $menu = $(this);
    var $box  = $menu.find('[role~="content"]');
    var state = {};
    
    // Bind events via delegation.
    $menu
      .on('click', '[role~="expand"]', expand)
      .on('click', '[role~="close"]', close);
    
    // Init: expand it on page load.
    if ($menu.data('expanded')) expand();
    
    function expand() {
      $box.show();
      loadFromAjax();
    }
    
    function loadFromAjax() {
      if (state.loaded) return;
      // TODO: load rows from ajax
      state.loaded = true;
    }    
  });
});
```

#### Event delegation

*Optional:* Instead of using `document.ready` to bind your events, consider using [jQuery event delegation][del] instead.

This allows you to bind behaviors to, say, modal windows, where the element may not be present when the document loads. There are cases wherein document.ready is necessary. If there's no other way to implement it, this should be fine.

[del]: http://learn.jquery.com/events/event-delegation/

```js
$(function () {
  $('[role~="sortable-table"]').on('hover', function () {
  });
});
```

```js
/* better */
$(document).on('hover', '[role~="sortable-table"]', function () {
});
```

## Turbolinks

These are guidelines to make your JS structure more friendly to Turbolinks. They may still be of benefit even without Turbolinks, however.

<br>

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

<br>

#### Tag your event handlers

Makes things neater, and allows you to trigger them later on. This also makes it easy to unbind them.

```js
$('...').on('click.myevent', function () {
});

/* later: */
$('...').off('.myevent');
```

<br>

#### Clean up if needed

Listen to `page:change` (?) to clean up anything to prepare the DOM for the next page.

```js
... // can't think of an example, lol.
```

<br>
