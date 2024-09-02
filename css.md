# CSS API

## Chain expressions

- `[n]`: Get the `n`th entry in a delimited list or a given named key. `n` may be a number, a string, chain expression resolving to a string or number (using the same host), or a binding access expression followed by such a chain expression.
- `.key`: Get a named property key. `key` must be an identifier.

## Instance definition at-rules

To define the instance, an `@instance name { ... }` element is used.

- `key: value`: Value to use as the state tracking key for list-based instances.
- Entries with duplicate key values are tracked by positional order.
- `@chain name as expr;`: Link a child chain view to a chain expression for nested instance viewing.
- All binding access expressions and instance names here are late-bound. It's not an error to not define them right away.

Instances' sources are defined per-element and are scoped to that element. As an exception, instance sources in `document.head` are also registered in `document.body` for convenience and ease of use.

## Binding access functions

This concept also includes `var(...)`s and `attr(...)`s, but no change to wither function is proposed. This grouping is purely editorial.

- `param(...)`: Like `attr(...)`, but reads from the node's shadow root's host element (or, if a shadow root, its own host element).
- `ikey(...)`: Name for current key from a given instance span context. Same syntax as `var`, but the name is an instance name.
- `ivalue(...)`: Name for current valhe from a given instance span context. Same syntax as `var`, but the name is an instance name.
- `ivar(...)`: Name for current value from a given instance name. Same syntax as `var`, but the name is an instance name. Invalid if it expands to a collection outside `get`.

## Value transform functions

- `host.key`, `host[n]`: Look up a chain expression inside the given host.
  - Open question: does this need a special function? I'm not aware of any spec that currently allows `func(...)` followed by either a left bracket or period.
- `string(value, options...)`: Converts `value` to a string, optionally using one or more optional format `options`. `value` can be a space-separated sequence, in which all values are converted then concatenated. Fails if `value` is multiple tokens. String format options:
  - `base(n)`: Convert numbers via the given base
  - `precision(n)`: Set max significant precision digits
  - `exponential`: Convert numbers to an exponential format, impacted by `precision(n)`
  - `fixed`: Convert numbers to a fixed-precision format, impacted by `precision(n)`
  - `integer(value unit? [base(n)]?)`: Converts `value` to a number, optionally from a given base. Fails if `value` isn't a valid integer.
  - `float(value unit?)`: Like `integer(...)` but converts to a float and doesn't allow changing base.
  - `uppercase(...)`, `lowercase(...)`, and `titlecase(...)` work like `string` but don't
  - `ident(...)`: Same arguments as `string(...)`, but returns an identifier. Identifiers (including true/false/null) are passed through, and strings are converted to identifiers. If the parameter is anything else or if the string isn't a valid identifier name, it's considered invalid and the referencing property/selector/rule dropped.
  - `format(kind value, language-tag, {options...})`: Do formatting
    - `kind` is the formatting kind
      - date
      - time
      - date-time
      - relative-time
      - iso-date (no language tag or options)
    - `language-tag` is the language tag as a string identifier. Omit to use the system default.
  - `{options...}` is a braced block of Intl option values. Omit for default values.

## Other functions

- `animation-time()`: The current monotonic timestamp in seconds (result has that unit)
  - Useful for clocks and for avoiding animation keyframes in simple linear cases.
  - Why? This value necessarily is generated for every `requestAnimationFrame()` pass, and it's really an analog signal of sorts. It doesn't require external state to synchronize.
- `wall-time()`: Like `animation-time()`, but reads from the system wall clock instead.
  - Not suitable for animations, but suitable for time-based conditional logic.
  - Why? It could be used for stuff like pure-CSS calendars and clocks. Notably, people could use this to do stuff like "starts in N minutes" or "finished N minutes/hours ago" in pure CSS and have it live-update in real time with almost no CPU.

## Other properties

- `instance: ...`: shorthand for `instance-*` properties
- `instance-source: ...`: specify instance source names
  - CSS-wide constants can be used as per usual.
  - If an instance source's value is neither an object, array, or number, the property is invalid.
  - Number instances enumerate integers from 0 to the given key's value.
  - Array and object sources enumerate their contents.
- `instance-filter: cond`: conditional expression identical to selector expressions, used to select what instances to include. Instances that don't match are not synthesized into elements. This is inheritable.
  - CSS-wide constants can also be used as per usual.
- `instance-window: 1em`: window margin or `auto` for an intelligent estimation
- `instance-sort: forward|reverse|ascending|descending`: sort the list in the given order
  - Default is "forward"
  - Impacts the accessibility tree
- `instance-sort-key: value`: Set the sort key for collection instance values. Default is the value's key. Sorting must be stable.
- `instance-order: n`: change the instance item order
  - Impacts the accessibility tree
- `attr(name unit?): value` causes the specified element attribute to be set with the matched element's namespace.
  - Prefix with `append` to append to whatever the existing value is, rather than replace it. Needed for class modification
  - This may cause a JS task to be scheduled after animation frame callbacks run, but before paint.
  - This requires a subsequent style calc pass to resolve if that attribute is matched by other selectors.
  - Selector rules are applied at most once to a given element, to prevent recalc cycles.
  - Animation of this is discrete by default.
  - Invalid in pseudo-elements other than instances.
- `text: value`: Like `content`, but sets the text and alt text together. It reflects in the accessibility tree, but not in the DOM proper, to avoid mucking with `:has(...)` selectors unpredictably. Invalid in pseudo-elements other than instances.

Use custom properties and property registrations to continuously animate attributes.

## Attribute selector extensions

- `[ivalue(name)...]`: Selects by instance value. Chain expressions are accepted before the `=`.
- `[ikey(name)...]`: Selects by instance key. This is just a normal attribute match.
- RHS can be a sequence of generic chain expressions, read like in numerical comparison selectors.

Works like normal attributes and with the same operators, but compares by instance key. Non-numeric values are first converted to strings via `string(ivalue(name))` and `string(ikey(name))` respectively.

## Selector extension for numerical comparisons: `[[left OP right]]`

- `left` and `right` are `calc` expression arguments. Identifiers with leading hyphens are treated as negated attribute values for consistency and simplicity.
- Binding access functions are accepted and, except `attr(...)`, read in the parent context. `attr(...)`s are read from the child. And any data transformation expression operating on either of the two is accepted.
- `OP` is either "==", "!=", ">", ">=", "<", or "<=". Chaining two inequalities is also allowed, and "and", "or", "not", and parentheses are allowed for combining them.
- This obviously establishes a dependency on every referenced attribute. And obviously, non-numerical attributes (attributes whose values don't match the number syntax) are invalid, causing the whole rule to not match. CSS already supports attribute selectors, so this is just a small(ish) +1.

> Why not container queries? I can't do comparisons like `custom property --foo >= child attribute bar` with them.

## Other notes

Named keys are either strings, numbers, or identifiers. Identifiers are treated as unquoted strings, both for simplicity and easier DOM API design.
