# css

## position

position properties:

- static: default, placed relative to normal document flow.
- relative: offset from the static position using `top`, `right`, `bottom`, and `left`. Surrounding elements behave as if the relative element is in its static (non-offset) position.
- absolute: like relative, except surrounding elements don't behave as if element is in regular static document flow, and instead fill in the gap. It is positioned relative to the nearest non-static ancestor (if one doesn't exist it is relative to `:root`).
  - tip: set `position: relative` to the parent element that the abolutely positioned element should be relative to, but don't set top/right/bottom/left on the `position: relative` element so that it behaves exactly like `position: static`.
- fixed: places item relative to the viewport, ignores underlying content. move with `top`, `right`, `bottom`, and `left`.
- sticky: elements behave like normal static elements, but will "stick" relative to the containing element by the offsets defined by `top`, `right`, `bottom`, and `left`. If offset is not explicitly defined, element will not stick.

helpful property: `inset`, which is a shorthand for `top`, `right`, `bottom`, and `left`. It has the same multi-value syntax as the `margin` shorthand.

## flexbox

when using `display: flex` on a container, the following properties are available on the **container**:

- flex-direction
  - row (default)
  - row-reverse
  - column
  - column-reverse
- flex-wrap
  - nowrap (default)
  - wrap
  - wrap-reverse
- flex-flow (space delimited combo of flex-direction and flex-wrap)
  - e.g. `flex-flow: column wrap` (order does not matter)
- justify-content
  - normal (default)
  - flex-start
  - flex-end
  - center
  - space-between (equal space between items, outer items touch sides)
  - space-around (equal space between items, but the space on the outer edges is **half** the space between items)
  - space-evenly (all gaps--including the outer edges--are exactly the same size)
- align-items
  - normal (default)
  - flex-start
  - flex-end
  - center
  - baseline (display at baseline of container)
  - stretch (stretch to fit container)
- align-content (only used when `flex-wrap` is `wrap` or `wrap-reverse`)
  - normal (default)
  - flex-start
  - flex-end
  - center
  - space-between (equal space between items, but closer to sides)
  - space-around (equal space around all items, including the sides)
  - stretch (lines are stretched to fit the container)
- gap

the following properties are available on the flex **items** within the container:

- order
  - 0 (default)
  - any postive or negative integer value
- align-self (same properties as align-items)
  - flex-start (default)
  - flex-end
  - center
  - baseline (display at baseline of container)
  - stretch (stretch to fit container)
- flex-grow
  - 0 (default)
  - 1 (items will grow to fill out empty space in the container)
  - 2 (will scale 2x to items that have flex-grow: 1)
  - etc.
- flex-shrink
  - 1 (default. note that `flex-shrink: 1` is why content appears to shrink when `display: flex` is added to a container)
  - 0 (prevent shrinking. useful for images, icons, etc.)
- flex-basis (overrides the **main axis** length, **takes precedence over flex-grow/shink**)
  - auto (default)
  - any standard unit (px, rem, etc.)
  - 0 (forces all items to be the same size on main axis, useful if content causes items sizes to be different. works well in tandem with `flex-grow: 1`)
- flex (shorthand of flex-grow, flex-shrink, and flex-basis)
  - e.g. `flex: 0 1 auto` (default)
  - useful: `flex: 1 1 0;` (fills container, allows shrinking, all items are the same size regardless of content)
  - `flex: 1` (equivalent to `flex: 1 1 0`)

tips:

- when `flex-direction: row`, justify is for x-axis, and align for y-axis.
- when `flex-direction: column`, justify is for y-axis, and align for x-axis.
- `align-items` vs `align-content` is a bit tricky: `align-content` can only be used for when there are multiple lines. `align-content` changes the entire content, whereas `align-items` changes the item alignment _relative to the line a given item is on_.
- `flex-basis` is relative to the main axis. its default property is `auto` which referes to the `width` property for `flex-direction: row` and the `height` property for `flex-direction: column`. Generally, using the `width` and `height` properties directly will give the desired behavior and keeps the item sizes the same if the `flex-direction` changes.
- `flex-basis` will overwrite the `width` or `height` properties, but `max-width` and `max-height` will overwrite `flex-basis`.

## grid

when using `display: grid`, set a dynamic number of columns based off the available viewport width:

```css
.grid-container {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(8rem, 1fr));
}
```

## counters

you can count any given type (element, class, etc.), and display that counter with the `counter(<counter-name>)` property:

```css
/* name the counter whatever you like */
.count-me {
  counter-increment: count-me-counter;
}

/* this also adds a space to the end of the number */
.count-me::before {
  content: counter(count-me-counter) " ";
}
```

## filters

filter properties (w/ example values):

- none; (default)
- blur(5px);
- brightness(0.4);
- contrast(200%);
- drop-shadow(16px 16px 20px blue);
- grayscale(50%);
- hue-rotate(90deg);
- invert(75%);
- opacity(25%);
- saturate(30%);
- sepia(60%);
- url("filters.svg#filter-id");

note that multiple filters can be used at once, e.g. `filter: blur(5px) grayscale(100%);`

## animations

set a `transition` property on an element you want to animate, for example to animate the `filter`:

```css
/* default */
.card-img {
  filter: grayscale(100%);
  transition: filter 500ms ease;
}

/* transition on hover */
.card:hover > .card-img {
  filter: grayscale(0%);
}
```

the `transition` property is a shorthand for the following properties, in the following order:

- transition-property (determines which css properties will be animated)
  - all (default)
  - otherwise select individual properties, e.g. `transition-property: background-color margin`. Non-selected properties will instantly switch to their new value without animating.
- transition-duration
  - 0s (default)
  - also accepts ms, e.g. `500ms`
- transition-timing-function
  - ease (default. s-curve variation, similar to `ease-out`)
  - linear
  - ease-in (exponential)
  - ease-out (logarithmic)
  - ease-in-out (s-curve)
  - cubic-bezier(0.25, 0.1, 0.25, 1.0) (given values are default for `ease`)
  - steps(n, \<jump-term>) (allows for breaking animation into a given number of "frames". see docs for more details)
- transition-delay (delay until animation begins)
  - 0s (default)
  - also accepts ms, e.g. `500ms`
- transition-behavior (used for very niche circumstances such as animating the `display` property)
  - normal (default)
  - allow-discrete

## tips

- useful default:

```css
/* debug */
/* * {
  outline: solid 1px hsl(0 100% 50% / 50%);
} */

* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

:root {
  --bg-color: #323437;
  --sub-color: #646669;
  --sub-color-alt: #2c2e31;
  --text-color: #d1d0c5;
  --main-color: #e2b714;
  --roundness: 0.5rem;
}

body {
  background-color: var(--bg-color);
  color: var(--text-color);
  font-family: monospace;
  min-height: 100dvh;
  min-width: 32rem;

  display: grid;
  place-items: center;
}
```

- `min-width` takes precedence over `max-width` (same with height).
- use "inline" for horizontal and "block" for vertical, e.g. `margin-inline` and `margin-block`.
- `isolation: isolate;` sets a new "stacking context". For example, if you have a div with `isolation: isolate;`, and an item within the div with `z-index: -1` (or -999, or whatever), that items z-index will never go below the div itself.
- when using multi-value property such as `padding`, `margin`, or `inset`, use the `auto` property to take the default value, e.g. `margin: auto 1rem 2rem 3rem` will take whatever the default for `margin-top` already was.
- calc: `width: calc(var(--some-var) + 1rem);`
- easily add a gradient border to a button or similar element:

```css
.my-button {
  border-image: linear-gradient(to right, #0066ff, #ff32d6) 1;
}
```

- use the `<details>` tag to create collapsable elements:

```html
<details>
  <summary>Click to expand</summary>
  <p>lorem ipsum dolor sit amet consectetur adipiscicing elit.</p>
</details>
```

- use `clip-path` to cut out portions of an element into specific shapes, or cut specific shapes out of an element. online tools such as https://cssportal.com/css-clip-path-generator/ can help with this.

## useful websites

https://cssportal.com

- a lot of useful css tools.
- cool fonts to check out (see the "Animated Text Generator" with "Rubik Maze" font, or "Google Fonts CSS").

https://gradient.style

- for all your gradient needs
