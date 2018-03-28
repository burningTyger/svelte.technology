---
title: Template syntax
---

Rather than reinventing the wheel, Svelte templates are built on foundations that have stood the test of time: HTML, CSS and JavaScript. There's very little extra stuff to learn.


### Tags

Tags allow you to bind data to your template. Whenever your data changes (for example after `component.set(...)`), the DOM updates automatically. You can use any JavaScript expression in templates, and it will also automatically update:

```html
<p>{{a}} + {{b}} = {{a + b}}</p>
```

```hidden-data
{
	"a": 1,
	"b": 2
}
```

You can also use tags in attributes:

```html
<h1 style='color: {{color}};'>{{color}}</h1>
<p hidden='{{hideParagraph}}'>You can hide this paragraph.</p>
```

```hidden-data
{
	"color": "steelblue",
	"hideParagraph": false
}
```
[Boolean attributes](https://www.w3.org/TR/html5/infrastructure.html#sec-boolean-attributes) like `hidden` will be omitted if the tag expression evaluates to false.

> While tags are delimited using `{{` and `}}`, Svelte does not use [Mustache](https://mustache.github.io/) syntax. Tags are just JavaScript expressions.


### Triples

Ordinary tags render expressions as plain text. If you need your expression interpreted as HTML, wrap it in triple braces, `{{{` and `}}}`.

```html
<p>This HTML: {{html}}</p>
<p>Renders as: {{{html}}}</p>
```

```hidden-data
{
	"html": "Some <b>bold</b> text."
}
```

As with tags, you can use any JavaScript expression in triples, and it will automatically update the document when your data changes.

> Triples will **not** sanitize the HTML before rendering it! If you are displaying user input, you are responsible for first sanitizing it. Not doing so opens you up to all sorts of different attacks.


### If blocks

Control whether or not part of your template is rendered by wrapping it in an if block.

```html-no-repl
{{#if user.loggedIn}}
	<a href='/logout'>log out</a>
{{/if}}

{{#if !user.loggedIn}}
	<a href='/login'>log in</a>
{{/if}}
```

You can combine the two blocks above with `{{else}}`:

```html-no-repl
{{#if user.loggedIn}}
	<a href='/logout'>log out</a>
{{else}}
	<a href='/login'>log in</a>
{{/if}}
```

You can also use `{{elseif ...}}`:

```html
{{#if x > 10}}
	<p>{{x}} is greater than 10</p>
{{elseif 5 > x}}
	<p>{{x}} is less than 5</p>
{{else}}
	<p>{{x}} is between 5 and 10</p>
{{/if}}
```

```hidden-data
{
	"x": 7
}
```

### Each blocks

Iterate over lists of data:

```html
<h1>Cats of YouTube</h1>

<ul>
	{{#each cats as cat}}
		<li><a target='_blank' href='{{cat.video}}'>{{cat.name}}</a></li>
	{{/each}}
</ul>
```

```hidden-data
{
	"cats": [
		{
			"name": "Keyboard Cat",
			"video": "https://www.youtube.com/watch?v=J---aiyznGQ"
		},
		{
			"name": "Maru",
			"video": "https://www.youtube.com/watch?v=z_AbfPXTKms"
		},
		{
			"name": "Henri The Existential Cat",
			"video": "https://www.youtube.com/watch?v=OUtn3pvWmpg"
		}
	]
}
```

You can access the index of the current element with *expression* as *name*, *index*:

```html
<div class='grid'>
	{{#each rows as row, y}}
		<div class='row'>
			{{#each columns as column, x}}
				<code class='cell'>
					{{x + 1}},{{y + 1}}:
					<strong>{{row[column]}}</strong>
				</code>
			{{/each}}
		</div>
	{{/each}}
</div>
```

```hidden-data
{
	"columns": [ "foo", "bar", "baz" ],
	"rows": [
		{ "foo": "a", "bar": "b", "baz": "c" },
		{ "foo": "d", "bar": "e", "baz": "f" },
		{ "foo": "g", "bar": "h", "baz": "i" }
  ]
}
```

> By default, if the list `a, b, c` becomes `a, c`, Svelte will *remove* the third block and *change* the second from `b` to `c`, rather than removing `b`. If that's not what you want, use a [keyed each block](guide#keyed-each-blocks).

Also, if you wish, you can perform one level of array destructuring on the elements of the array directly in the each block:

```html
<h1>It's the cats of YouTube again</h1>

<ul>
	{{#each cats as [name, video]}}
		<li><a target='_blank' href='{{video}}'>{{name}}</a></li>
	{{/each}}
</ul>
```

```hidden-data
{
	"cats": [
		[
			"Keyboard Cat",
			"https://www.youtube.com/watch?v=J---aiyznGQ"
		],
		[
			"Maru",
			"https://www.youtube.com/watch?v=z_AbfPXTKms"
		],
		[
			"Henri The Existential Cat",
			"https://www.youtube.com/watch?v=OUtn3pvWmpg"
		]
	]
}
```

```html
<h1>Cats and Dogs</h1>

{{#each Object.entries(animals) as [animal, names]}}
    <br/> {{animal}}: {{names.join(" and ")}}
{{/each}}
```

```hidden-data
{
  "animals": {
	"Cats": [ "Buzz", "Stella" ],
  	"Dogs": [ "Hector", "Victoria" ]
	}
}
```

### Await blocks

You can represent the three states of a [Promise](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) — pending, fulfilled and rejected — with an `await` block:

```html
{{#await promise}}
	<p>wait for it...</p>
{{then answer}}
	<p>the answer is {{answer}}!</p>
{{catch error}}
	<p>well that's odd</p>
{{/await}}

<script>
	export default {
		data() {
			return {
				promise: new Promise(fulfil => {
					setTimeout(() => fulfil(42), 3000);
				})
			};
		}
	};
</script>
```

If the expression in `{{#await expression}}` *isn't* a promise, Svelte skips ahead to the `then` section.


### Directives

The last place where Svelte template syntax differs from regular HTML: *directives* allow you to add special instructions for adding [event handlers](guide#event-handlers), [two-way bindings](guide#two-way-binding), [refs](guide#refs) and so on. We'll cover each of those in later stages of this guide – for now, all you need to know is that directives can be identified by the `:` character:

```html
<p>Count: {{count}}</p>
<button on:click='set({ count: count + 1 })'>+1</button>
```

```hidden-data
{
	"count": 0
}
```

> Technically, the `:` character is used to denote namespaced attributes in HTML. These will *not* be treated as directives, if encountered.
