# Deprecated Blocks

When updating static blocks markup and attributes, block authors need to consider existing posts using the old versions of their block. In order to provide a good upgrade path, you can choose one of the following strategies:

 - Do not deprecate the block and create a new one (a different name)
 - Provide a "deprecated" version of the block allowing users opening these blocks in Gutenberg to edit them using the updated block.

A block can have several deprecated versions. Gutenberg will try them one after another until finding the one that matches the saved markup.

To declare a deprecated version, you need to copy the old `attributes`, `support` and `save` properties from the old definition to the `deprecated` property of the updated block.

### Example:

{% codetabs %}
{% ES5 %}
```js
var el = wp.element.createElement,
	registerBlockType = wp.blocks.registerBlockType,
	attributes = {
		text: {
			type: 'string',
			default: 'some random value',
		}
	};

registerBlockType( 'gutenberg/block-with-deprecated-version', {

	// ... other block properties go here

	attributes: attributes,

	save: function( props ) {
		return el( 'div', {}, props.attributes.text );
	},

	deprecated: [
		{
			attributes: attributes,

			save: function( props ) {
				return el( 'p', {}, props.attributes.text );
			},
		}
	]
} );
```
{% ESNext %}
```js
const { registerBlockType } = wp.blocks;
const attributes = {
	text: {
		type: 'string',
		default: 'some random value',
	}
};

registerBlockType( 'gutenberg/block-with-deprecated-version', {

	// ... other block properties go here

	attributes,

	save( props ) {
		return <div>{ props.attributes.text }</div>;
	},

	deprecated: [
		{
			attributes,

			save( props ) {
				return <p>{ props.attributes.text }</p>;
			},
		}
	]
} );
```
{% end %}

In the example above we updated the markup of the block to use a `div` instead of `p`.


## Changing the attributes set

Sometimes, you need to update the attributes set to rename or modify old attributes.

### Example:

{% codetabs %}
{% ES5 %}
```js
var el = wp.element.createElement,
	registerBlockType = wp.blocks.registerBlockType;

registerBlockType( 'gutenberg/block-with-deprecated-version', {

	// ... other block properties go here

	attributes: {
		content: {
			type: 'string',
			default: 'some random value',
		}
	},

	save: function( props ) {
		return el( 'div', {}, props.attributes.text );
	},

	deprecated: [
		{
			attributes: {
				text: {
					type: 'string',
					default: 'some random value',
				}
			},

			migrate: function( attributes ) {
				return {
					content: attributes.text
				};
			},

			save: function( props ) {
				return el( 'p', {}, props.attributes.text );
			},
		}
	]
} );
```
{% ESNext %}
```js
const { registerBlockType } = wp.blocks;

registerBlockType( 'gutenberg/block-with-deprecated-version', {

	// ... other block properties go here

	attributes: {
		content: {
			type: 'string',
			default: 'some random value',
		}
	},

	save( props ) {
		return <div>{ props.attributes.text }</div>;
	},

	deprecated: [
		{
			attributes: {
				text: {
					type: 'string',
					default: 'some random value',
				}
			},

			migrate( { text } ) {
				return {
					content: text
				};
			},

			save( props ) {
				return <p>{ props.attributes.text }</p>;
			},
		}
	]
} );
```
{% end %}

In the example above we updated the markup of the block to use a `div` instead of `p` and rename the `text` attribute to `content`.


## Changing the innerBlocks

Situations may exist where when migrating the block we may need to add or remove innerBlocks.
E.g: a block wants to migrate a title attribute to a paragraph innerBlock.

### Example:

{% codetabs %}
{% ES5 %}
```js
var el = wp.element.createElement,
	registerBlockType = wp.blocks.registerBlockType;

registerBlockType( 'gutenberg/block-with-deprecated-version', {

	// ... block properties go here

	deprecated: [
		{
			attributes: {
				title: {
					type: 'array',
					source: 'children',
					selector: 'p',
				},
			},

			migrate: function( attributes, innerBlocks ) {
				return [
					omit( attributes, 'title' ),
					[
						createBlock( 'core/paragraph', {
							content: attributes.title,
							fontSize: 'large',
						} ),
					].concat( innerBlocks ),
				];
			},

			save: function( props ) {
				return el( 'p', {}, props.attributes.title );
			},
		}
	]
} );
```
{% ESNext %}
```js
const { registerBlockType } = wp.blocks;

registerBlockType( 'gutenberg/block-with-deprecated-version', {

	// ... block properties go here

	save( props ) {
		return <p>{ props.attributes.title }</div>;
	},

	deprecated: [
		{
			attributes: {
				title: {
					type: 'array',
					source: 'children',
					selector: 'p',
				},
			},

			migrate( attributes, innerBlocks  ) {
				return [
					omit( attributes, 'title' ),
					[
						createBlock( 'core/paragraph', {
							content: attributes.title,
							fontSize: 'large',
						} ),
						...innerBlocks,
					],
				];
			},

			save( props ) {
				return <p>{ props.attributes.title }</p>;
			},
		}
	]
} );
```
{% end %}

In the example above we updated the block to use an inner paragraph block with a title instead of a title attribute.
