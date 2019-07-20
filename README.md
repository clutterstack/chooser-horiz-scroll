# chooser-horiz-scroll

A Vue.js component for selecting a value from an array, displayed horizontally and scrollable if the rendered row of items is larger than the containing `<div>`.

This repo wraps the component in a test Vue app to make it easy to demonstrate. Live demo with further details lives at [chrisnicoll.net](http://chrisnicoll.net/web-stuff/vue-misc/vue-horizontal-chooser-component/).

## Props

Takes an array of values (`targets`) as a prop. Emits one of two custom events, depending on whether a value received a full click or just a mousedown. Both events carry the value as a payload. This allows, for example, either selecting the value or "dragging" it to another component and doing something with it there.

In the demo, these events are treated as a selection and a tentative selection.

There is another prop (`mouseOutOfBounds`), that is watched by the component and is treated the same as a mouseup on its `document`. It is used by the demo app to indicate that the mouse has left its root element, which allows more predictable behaviour when embedded using `<object>` or `<iframe>` in another page (such as the demo post linked above).

## Animation

There is some "physics" to allow the chooser to be "thrown" and to bounce off the end of its travel. Animation parameters are set in the component's `data` option.
