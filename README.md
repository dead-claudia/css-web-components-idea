# css-web-components-idea
An idea of how to use instancing to define and use CSS Web Components. I've also included a few comparisons with Svelte as the data model is somewhat similar.

> Note: I do *not* rely on https://github.com/w3c/csswg-drafts/issues/10064, though such an if/else in CSS would be extremely powerful when combined with this.
>
> Also, please excuse the extremely rough nature of this. I copied all of this from a bunch of bullet point abstract notes I made on my phone over the course of several days, with basically no alteration. Feel free to file issues or pull requests if you need help understanding things.

- [CSS API](./css.md)
- [HTML API](./html.md)
- [JS API](./js.md)
- [Examples](./examples/)

## Intuition

- JS<->DOM interaction is a major bottleneck in DOM updates. This is why high-performance implementations like Mithril and Inferno push to the extreme to avoid even invoking APIs like `.nextSibling`, and why frameworks like Svelte often build their initial trees straight from HTML templates.
- HTML processing is highly optimized, but it is only good at expressing data. JSX and virtual DOM worked around it by using an embedded DSL, while Svelte, Handlebars, and friends made it halfway to an ERB derivative. In either case, it's not truly declarative, and code reuse can be slightly awkward.
- CSS already provides most of the backbone needed for template control. Extreme layout flexibility is already leveraged for very complex designs, and at this point it has almost everything a shader language provides except literal texture arithmetic.
- OpenGL has this neat concept called instanced array rendering. It's specifically designed to let you render many similar items really fast with almost no code by just initializing the data in advance and placing it somewhere accessible to the GPU.

## Advantages

- No need for a heavy new template language. Just a relatively simple set of CSS extensions to smooth out the edges.
- It provides a genuinely low-code way to do simple data displaying, and loading can just be done by detecting the absence of an instance binding.
- No need for tons of new attributes.
- It's much easier to reuse repeating logic. In large, data-heavy apps, this is very valuable, but even in the small, I leveraged this to great extent in [this Hacker News conceptual clone](./examples/hacker-news.html).
- It's almost like an inverted spreadsheet in how it selects ranges.
- Microdata can be set up with far less boilerplate. It can be coupled with class layout, essentially turning CSS into an HTML -> microdata transformation layer.
- Nothing here grants CSS security access greater than it had before. However, it provides a new potential extension surface area for sites like (old) Reddit that allow CSS customization but not HTML or JS customization. Browser extensions and userscripts could also leverage this to provide helpful info.
