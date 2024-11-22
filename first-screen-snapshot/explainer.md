# Explainer: First Screen Snapshot

## Motivation

When loading a page, the user typically sees a blank screen before anything is displayed (aka first paint). This doc proposes a mechanism that caches the rendering of a developer-chosen section of the page, and displays it on subsequent loads of the same page as the first screen, so that the user no longer seens a blank screen at all.

The snapshot can be used to create a static splash screen (which is currently only available to PWA and hybrid apps with a WebView).

[![Demo Video](https://youtube.com/watch?v=nN0uxNbCSpQ)](https://youtube.com/watch?v=nN0uxNbCSpQ)

This is a very rough draft to gather some thoughts and see if this is something worth pursuing. Any comments are appreciated!

## The Proposal

We introduce a new JS function, tentatively named `window.snapshotFirstScreen(element)`, to take a snapshot of the current rendering of `element`. The snapshot will be displayed next time at the same location when the same page is loaded.

Note that for any page, there can be at most one snapshot. If the function is called for multiple times during the lifetime of the page, only the first call is effective.

We also introduce `window.dropFirstScreenSnapshot()` (named tentatively) that allows the author to explicitly drop any existing snapshot.

The UA should also drop the existing snapshot when necessary, especially when it finds out that preserving the snapshot will cause [privacy issues](#security-and-privacy-considerations). Another typical case is when the viewport is resized, so that the existing snapshot will no longer work.

*Discussion.* How do we reduce the amount of magic here and still make everything work?

We may consider declarative API later, if any progress can be made on this imperative version.

### When to display and hide the snapshot

The snapshot is displayed as soon as navigation starts and a blank screen can be displayed (which probably needs a more refined definition), and removed right after the first paint. It's displayed on top of the page content. The snapshot will be hidden when the page calls `window.snapshotFirstScreen()` for the first time, or after a UA-defined timeout.

*Discussion.* The snapshot is visible but not interactable, which may cause some negative user experience. Or does it matter?

*Discussion.* Any possibility to display it before the first rendering opportunity? Which means it's not even part of DOM/CSS, it's just an overlay added by the renderer. Otherwise, if we make the snapshot a pseudo-element (like in view transisitions) then we still need to wait for it to be fully ready, which beats the purpose of the proposal.

### Snapshot Storage

The user agent manages a snapshot storage keyed by profile and page url. The storage must be persistent and encrypted, and should not be exposed via any API.

### Interaction with View Transitions

The snapshot will not be displayed if cross-doc VT is active.

### Accessibility Considerations

The snapshot is not exposed to AT, since it's presentational.

### Security and Privacy Considerations

*Discussion.* What if the snapshot contains something sensitive? E.g. user name?

*Discussion.* What if e.g. the snapshot is not properly invalidated (for any reason, e.g., page doesn't explicitly drop snapshot when needed), and the page is run on a public computer, so that a user somehow sees the previous user's information?

Looks like certain restrictions on the browser/OS level is required. For example:
- The feature is disabled on guest/incognito mode
- The feature is diabled if the OS is not logged in and protected with PIN/fingerprint/...

And we also need some comprehensive mechanism so that the UA can automatically drop the snapshot when necessary.

## Existing Solutions

Speculation Rules API
- Pro: Shortens or even eliminates the blank screen, and delivers the actualy page very quickly
- Cons: Doesn't work when opening a new page; Prefetching/prerendering consumes resources and still takes time.

Server-Side Rendering
- Pros: Reduces the blank screen time, and improves search engine ranking
- Cons: Harder to develop and maintain, as the frontend and backend are more tightly coupled. Also takes more resources on the server side.

Show a skeleton / loading screen
- Pros: Eliminates the blank screen at all
- Cons: Doesn't shorten the time to display anything meaningful

Resource preloading
- Pros: Simple and low overhead to reduce the blank screen time
- Cons: Doesn't always work; doesn't work if the slowness is caused by JS

Implementing the page as a static page + dynamic data
- Shortens blank screen time; less JS execution
- Very complicated to implement

### Solutions that apply to WebView only

There are also some solutions that do not apply to browsers but apply to hybrid apps that host a WebView, which is the use case our team focuses on.

Pre-rendering with a WebView Pool
- Pros: Eliminates blank screen, and reduces WebView startup time by a lot
- Cons: Needs to maintain many WebView instances, which consume a lot of resources; And it still needs some time to warm up

Caching the HTML of the previous page load
- Pros: Shortens the blank screen time
- Cons: Causes a sudden jump in page display
