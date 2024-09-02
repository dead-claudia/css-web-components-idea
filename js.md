# JS API

## Registration

- `element.instanceSources`: Return the `InstanceSource` for the given element.
  - `InstanceSource` is a `maplike<DOMString, any>`. Not much else is offerred, except that values are observably `JSON.stringify`'d on set and return new `JSON.parse` results from that on every get.
  - `.get(...)` accepts an additional optional reviver parameter.
  - `.set(...)` accepts an additional optional replacer parameter. No space parameter is accepted, as the serialized representation is never returned.
- `CSS.registerInstance("name", {...})`: Same as `@instance name { ... }`, but as a JS API
  - Chains use `chain: {name: ".path", ...}`
- `document.createElement(...)` accepts an `instanced` boolean option alongside other options, and returns an `InstancedNode` placeholder that accepts all the same attributes and styles, but doesn't actually construct a full-featured maanged element and can only accept text nodes and instanced elements as children. This is also what the `instanced` attribute generates.
  - `HTMLDefineElement` instances also only allow text and instanced nodes as children.

## Element instance maps

`element.instances()`: Returns an iterable over an element's instance children.

Instance elements can have inner attributes, children, and styles changed as usual. If attributes, children, and styles in the host element are changed, those changes are reflected across all instances.

Instance elements cannot be directly moved or removed, and elements cannot be inserted between them. They are explicitly kept outside the normal tree, so they can have their rendering and updating better optimized.

When an element with instances is cloned deeply, its instances are *not* included. This is because the instance list could vary based on where the cloned node is inserted later, and there's no easy and efficient way to diff on insertion. Also, the instance source isn't accessible in the cloned node.

## Custom element API

https://github.com/WICG/webcomponents/issues/1064 for the core of it. Not much else to say.
