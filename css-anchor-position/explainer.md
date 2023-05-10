# CSS Anchor Positioning Explainer

[CSS Anchor positioning](https://drafts.csswg.org/css-anchor-position-1/) is a CSS feature that allows an element to size and position itself relative to one or more "anchor elements" elsewhere on the page. A typical use case is to "pin" a tooltip to something that triggers it.

This is a short explainer covering some basic use cases. See also:
- [Chrome Developers article](https://developer.chrome.com/blog/tether-elements-to-each-other-with-css-anchor-positioning) for a complete introduction
- [Spec](https://drafts.csswg.org/css-anchor-position-1/) for the full technical details.

## The Basics

First, use the [`anchor-name`](https://drafts.csswg.org/css-anchor-position-1/#propdef-anchor-name) property to define a named anchor.

```css
.anchor {
  anchor-name: --my-anchor;
}
```

Then, to position another element above this element:

```css
.target {
  position: fixed;
  left: anchor(--my-anchor left);  /* Align left edge with anchor's left edge */
  bottom: anchor(--my-anchor top); /* Align bottom edge with anchor's top edge */
}
```

<iframe width="100%" height="300" src="//jsfiddle.net/w05frn47/embedded/" allowfullscreen="allowfullscreen" allowpaymentrequest frameborder="0"></iframe>
