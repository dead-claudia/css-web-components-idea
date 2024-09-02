# HTML API

## Instance data provision

Provide initial instance data declaratively via `<meta provide="name@data">`.

- `name` is the instance type name.
- `data` is the corresponding initial instance data as a URL (usually an `data:` URL) to a JSON blob. The actual MIME type specified is ignored.
- Surrounding spaces are trimmed (though it doesn't entirely matter with the data URL).
- This is declaratively registered by name to its parent element and removed just like `<template shadowrootmode="...">`. It's specially designed to be efficiently elidable during parsing, so hydration data can be parsed out without needing to retain two copies of it.
- This brings native hydration without needing tons of declarative shadow DOM elements. It also works better with instancing.
- SVG should consider adding a similar `<meta provide="..." />` in their metadata sections. It could simplify a lot of element repetition without having to resort to the relatively heavy `<use>` element.
- It's recommended to specify this before any element whose style refers to the data, so it can be eagerly decoded and applied.

## Instance linking

Use an `instanced` attribute to enable instancing on an element. It and its child elements are replaced with placeholder nodes to manage the instancing. No element is created or upgraded.

This has to be done in HTML land. The alternative is an extra boilerplate node with a very bug-prone story for custom elements.

SVG and MathML does the same, using their native namespace

## Custom element definition

Define a custom element using `<define>`.

- `name`: The custom element's name
- `extends`: The builtin to extend
- `attributes`: A space-separated list of attributes to watch
- `shadowroot*` attributes apply to the created shadow root
- Children are template children, not constructed.
- Inner `<script type="module">...</script>` drives the instance.

Its inner children are also implicitly instanced.

## Declarative shadow DOM extensions

- `shadowrootexpose`: A space-separated series of instance data names to expose.
