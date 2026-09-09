# FontJit

A little helper for just-in-time font loading! Load fonts when they _enter the viewport_, or when they _almost enter the viewport_, or _immediately_.

It's tiny, just 793 bytes minified and Brotli zipped!

## Quick Start

### 1. Set up FontJit in JavaScript

Load FontJit, and call it. By default it'll automatically find all FontJit elements on the page, but you can also pass a CSS selector string, a DOM element, or a Nodelist to target specific elements. By default fonts will be loaded when they're _in_ the viewport on pageload, and when they _enter_ the viewport when the user scrolls.

```html
<script type="module">
	import { fontJit } from './fontjit.js'
	fontJit() // Or target specific elements: fontJit('.boing')
</script>
```

### 2. Prepare your HTML elements

Add data attributes with the font URL and font name to the element.

```html
<div class="boing" data-fontjit-url="boing.woff2" data-fontjit-name="Boing">
	This is the Boing font!
</div>
```

ℹ️ Make sure you sanitize the font name to avoid icky browser bugs. Remove spaces, quotes, plus signs etc. Read more about this issue [in this Mastodon post](https://typo.social/@pixelambacht/110615435477645570).

### 3. Apply the font

The font will now be loaded when this element enters the viewport. Remember, `fontJit` only takes care of _loading_ the font—you'll have to actually _apply_ it yourself.

```html
<style>
	.boing {
		font-family: Boing;
	}
</style>
```

## Loading Options

By default, `fontJit` lazy-loads fonts when they enter the viewport. You can also have them load when they _almost enter_ the viewport by passing an [IntersectionObserver](https://developer.mozilla.org/en-US/docs/Web/API/IntersectionObserver) config, or load them immediately:

```javascript
// Load font for elements with class "boing" when they're in the viewport
fontJit('.boing')

// Load font for elements with class "boing" when they're almost in the viewport
fontJit('.boing', { rootMargin: '400px 0px' })

// Load font for elements with class "boing" immediately, regardless
// of whether they are (almost) in the viewport
fontJit('.boing', { immediate: true })
```

## Loading States

While loading, the `data-fontjit-status` attribute will be added and updated. This lets you keep track of the four loading states:

```
unloaded = not yet started
loading = font is loading
loaded = font successfully loaded
error = loading failed
```

You can use these in CSS, for example to hide the text until the font has been loaded, or show an error when the font fails to load.

```css
/* Apply the font and hide it until it has loaded */
.boing {
	font-family: Boing;
	opacity: 0;
}

/* Font has loaded, show the element */
.boing[data-fontjit-status='loaded'] {
	opacity: 1;
}

/* Font couldn't load, show error state */
.boing[data-fontjit-status='error'] {
	opacity: 1;
	outline: 10px solid red;
}
```

## Using JavaScript

You can also interact with FontJit in JavaScript by using promises:

```javascript
import { fontJit, LoadingState } from './fontjit.js'

// Using async/await
await fontJit('.boing')
console.log('All fonts loaded!')

// Using .then()
fontJit('.boing').then(() => {
	console.log('All fonts loaded!')
}).catch((error) => {
	console.error('Some fonts failed to load:', error)
})

// Load multiple fonts and wait for all
await Promise.all([
	fontJit('.boing'),
	fontJit('.prose'),
	fontJit('.code-example')
])

// Alternartively, check loading state using constants
const element = document.querySelector('.boing')
if (element.getAttribute('data-fontjit-status') === LoadingState.LOADED) {
	// Do something with the loaded font!
}
```

The promise works with both immediate and lazy loading modes. For lazy loading, the promise resolves as fonts enter the viewport and complete loading.

## Font Descriptors

You can pass optional [FontFace descriptors](https://developer.mozilla.org/en-US/docs/Web/API/FontFace/FontFace#descriptors) using the `data-fontjit-descriptors` attribute with a JSON string:

```html
<div
	class="boing"
	data-fontjit-url="boing.woff2"
	data-fontjit-name="Boing"
	data-fontjit-descriptors='{"weight":"100 900","stretch":"75% 125%"}'
>
	Hello World
</div>
```

(Note the single quotes used for `data-fontjit-descriptors`, as the JSON string must use double quotes!)

## Performance Tips

You can load fonts in the initial viewport immediately, and preload nearby fonts after page has fully loaded:

```javascript
// Load fonts in viewport...
fontJit('.boing')

// ...and after page loads, preload fonts just below the fold
window.addEventListener('load', () => {
	fontJit('.boing', { rootMargin: '400px 0px' })
})
```

## Gotchas

If you update an element after its font has been loaded, for example by updating its `data-fontjit-url`, you'll need to manually reset the `data-fontjit-status` attribute before calling `fontJit()` again. This might be automated [soon](https://github.com/RoelN/FontJit/issues/4).

## Credits

Computered by [Roel Nieskens](https://pixelambacht.nl)!
