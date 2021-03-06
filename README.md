# html-to-react
[![Build Status](https://travis-ci.org/aknuds1/html-to-react.svg?branch=master)](https://travis-ci.org/aknuds1/html-to-react)
[![npm version](https://badge.fury.io/js/html-to-react.svg)](http://badge.fury.io/js/html-to-react)
[![Dependency Status](https://david-dm.org/aknuds1/html-to-react.svg)](https://david-dm.org/aknuds1/html-to-react)
[![Coverage Status](https://coveralls.io/repos/aknuds1/html-to-react/badge.svg?branch=master)](https://coveralls.io/r/aknuds1/html-to-react?branch=master)
[![npm](https://img.shields.io/npm/dm/html-to-react.svg?maxAge=2592000)](https://www.npmjs.com/package/html-to-react)

A lightweight library that converts raw HTML to a React DOM structure.

## Why?
I had a scenario where an HTML template was generated by a different team, yet I wanted to leverage
React for the parts I did have control over. The template basically contains something like:

```
<div class="row">
    <div class="col-sm-6">
        <div data-report-id="report-1">
          <!-- A React component for report-1 -->
        </div>
    </div>
    <div class="col-sm-6">
        <div data-report-id="report-2">
          <!-- A React component for report-2 -->
        </div>
    </div>
</div>
```

I had to replace each `<div>` that contains a `data-report-id` attribute with an actual report,
which was nothing more than a React component.

Simply replacing the `<div>` elements with a React component would end up with multiple top-level
React components that have no common parent.

The **html-to-react** module solves this problem by parsing each DOM element and converting it to a
React tree with one single parent.

## Installation

`$ npm install --save html-to-react`

## Examples

### Simple

The following example parses each node and its attributes and returns a tree of React components.

```javascript
var React = require('react');
var HtmlToReactParser = require('html-to-react').Parser;

var htmlInput = '<div><h1>Title</h1><p>A paragraph</p></div>';
var htmlToReactParser = new HtmlToReactParser();
var reactComponent = htmlToReactParser.parse(htmlInput);
var reactHtml = React.renderToStaticMarkup(reactComponent);

assert.equal(reactHtml, htmlInput); // true
```

### With Custom Processing Instructions

If certain DOM nodes require specific processing, for example if you want to capitalize each
`<h1>` tag, the following example demonstrates this:

```javascript
var React = require('react');
var HtmlToReactParser = require('html-to-react').Parser;

var htmlInput = '<div><h1>Title</h1><p>Paragraph</p><h1>Another title</h1></div>';
var htmlExpected = '<div><h1>TITLE</h1><p>Paragraph</p><h1>ANOTHER TITLE</h1></div>';

var isValidNode = function() {
    return true;
};

// Order matters. Instructions are processed in the order they're defined
var processNodeDefinitions = new HtmlToReact.ProcessNodeDefinitions(React);
var processingInstructions = [
    {
        // Custom <h1> processing
        shouldProcessNode: function(node) {
            return node.parent && node.parent.name && node.parent.name === 'h1';
        },
        processNode: function(node, children) {
            return node.data.toUpperCase();
        }
    }, {
        // Anything else
        shouldProcessNode: function(node) {
            return true;
        },
        processNode: processNodeDefinitions.processDefaultNode
    }];
var htmlToReactParser = new HtmlToReactParser();
var reactComponent = htmlToReactParser.parseWithInstructions(htmlInput, isValidNode,
  processingInstructions);
var reactHtml = React.renderToStaticMarkup(reactComponent);
assert.equal(reactHtml, htmlExpected);
```

### Replace the Children of an Element

There may be a situation where you want to replace the children of an element with a React
component. This is beneficial if you want to:
- a) Preserve the containing element
- b) Not rely on any child node to insert your React component

#### Example

Below is a simple template that could get loaded via ajax into your application

##### Before
```
<div class="row">
    <div class="col-sm-6">
        <div data-container="wysiwyg">
            <h1>Sample Heading</h1>
            <p>Sample Text</p>
        </div>
    </div>
</div>
```

##### After

You may want to extract the inner html from the `data-container` attribute, store it and then pass
it as a prop to your injected `RichTextEditor`.

```
<div class="row">
    <div class="col-sm-6">
        <div data-container="wysiwyg">
            <RichTextEditor html={"<h1>Sample heading</h1><p>Sample Text</p>"} />
        </div>
    </div>
</div>
```

#### Setup

In your instructions object, you must specify `replaceChildren: true`.

```javascript
var React = require('react');
var HtmlToReactParser = require('html-to-react').Parser;

var htmlToReactParser = new HtmlToReactParser();
var htmlInput = '<div><div data-test="foo"><p>Text</p><p>Text</p></div></div>';
var htmlExpected = '<div><div data-test="foo"><h1>Heading</h1></div></div>';

var isValidNode = function() {
    return true;
};

// Order matters. Instructions are processed in
// the order they're defined
var processingInstructions = [
    {
       // This is REQUIRED, it tells the parser
       // that we want to insert our React
       // component as a child
       replaceChildren: true,
       shouldProcessNode: function (node) {
            return node.attribs && node.attribs['data-test'] === 'foo';
        },
        processNode: function (node, children, index) {
            return React.createElement('h1', {key: index,}, 'Heading');
        }
    },
    {
        // Anything else
        shouldProcessNode: function (node) {
            return true;
        },
        processNode: processNodeDefinitions.processDefaultNode,
    },
];

var reactComponent = parser.parseWithInstructions(
  htmlInput, isValidNode, processingInstructions);
var reactHtml = ReactDOMServer.renderToStaticMarkup(
  reactComponent);
assert.equal(reactHtml, htmlExpected);
```

## Tests & Coverage

Test locally: `$ npm test`

Test with coverage and report coverage to Coveralls: `$ npm run test-coverage`

Test with coverage and open HTML report: `$ npm run test-html-coverage`
