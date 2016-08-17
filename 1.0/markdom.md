# ![logo](resource/markdom-mark-negative-inline.png) Markdom

Markdom is lightweight specification for a rich text domain model.

The two main intents of Markdom are to define the lowest common denominator for rich text and to be platform and representation independent. Theese properties enable Markdom to be used in a wide variety of applications, such as websites, electronic publishing or native rendering on mobile platforms.

## Why Markdom

Markdom was created to be an answer to a simple, but as yet unanswered [question](https://stackoverflow.com/questions/34955835/cross-device-rich-text-formatting):

1. Allow an editor to enter rich text in a human friendly way.
2. Transmit the rich text to different platforms, using existing technologies.
3. Display the rich text on different platforms in a native way.

The latter includes at least Android apps (using [Spanned Strings](https://developer.android.com/reference/android/text/Spanned.html)), iOS apps (using [Attributed Strings](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSAttributedString_Class/)) and websites (using HTML).

Markdom tries to fill a gap left open by existing technologies. As the name Markdom indicates, it is closly related to [Markdown](https://daringfireball.net/projects/markdown/).

### Why not HTML?

HTML itself is hardly an editor friendly language, although usable [WYSISYG editors](https://www.tinymce.com/) exist.

The crucial drawback of HTML is, that is is way too powerfull for the intended purpose. There are [good reasons](https://leanpub.com/markua/read#leanpub-auto-why-is-inline-html-not-supported-in-markua) why HTML is not suitable, if the intended target platform aren't just websites.

### Why not Markdown?

Markdown is introduced with the following description:

> Markdown is a text-to-HTML conversion tool for web writers. Markdown allows you to write using an easy-to-read, easy-to-write plain text format, then convert it to structurally valid XHTML (or HTML).

The intended limitation to be nothing more than a text-to-HTML conversion tool is recognizable in a multitude of Markdown implementation. Only few implementations offer conversion targets that are not HTML or a domain model that is open to programmatic manipulation.

Since Markdown was only released as an implementation and without a formal specification, different implementations [disagree](http://johnmacfarlane.net/babelmark2/?text=%3E+*+A+quote%0A%3E+Another+quote) over the meaning of edge cases like the following snippet:

```
> * A quote
> Another quote
```

Additionally, many Markdown implmentation use custom flavors of the Markdown syntax or offer custom extensions.

As a result, it is [almost](https://uncodin.github.io/bypass/) impossible to find a set of markdown implmentations that generate Spanned Strings for Android, Attributed Strings for iOS and HTML for websites, that interprete a given Markdown text identically and genereate comparable outputs.

### Why not CommonMark?

[CommonMark](http://commonmark.org/) is a specification for Markdown and compatible CommonMark implementations exist for [different languages](https://github.com/jgm/CommonMark/wiki/List-of-CommonMark-Implementations).

While this solves some of the issues of Markdown, CommonMark still allows the integration of arbitrary HTML and available implementations are still only text-to-HTML converters or tend to offer custom extensions.

## Name

The name Markdom is a composition of *Mark*up (or *Mark*down) and DOM (domain object model).

```
# Markdom

1. Allow an editor to enter rich text in a human friendly way.
2. Transmit the rich text to different platforms, using existing technologies.
3. Display the rich text on different platforms in a native way.
```

## Domain

*This sections describes the general structure of a Markdom document, independent of any representation.*

### Overview

A *Markdom document* represents the entirety of a rich text document, including the actual rich text and the formatting instructions. A Markdom document contains of a sequence of *Markdom blocks*.

A Markdom block represents a portion of a rich text document that serves a specific purpose. Each type of Markdom block has the necessary properties that describe it's content. Some Markdom blocks, e.g. a Markdom block representing a blockm quote, contain further Markdom blocks. Other Markdom blocks, e.g. a Markdom block representing a paragraph, contain *Markdom content*.

A Markdom content represents a portion of rich text that serves a specific purpose. Each type of Markdom block has the necessary properties that describe it's content. Some Markdom contents, e.g. a Markdom block representing a link, contain further Markdom content. A Markdom content never contains a Markdom block.

Put another way, a Markdom document is represented as a tree of *Markdom nodes*. Starting with a root node that represents an entire rich text document, each markdom node represents a portion of the corresponding Markdom document.

### Nodes {#domain-node}

*This sections describes the Markdom nodes of a Markdom document.*

#### Document

A `Document` node represents an entire Markdom document.

A `Document` node is the root node of a Markdom document and contains a sequence of `Block` nodes named `blocks`.

The `blocks` sequence shouldn't be empty.

#### Block

A `Block` node represents a portion of a Markdom document.

A `Block` node is either a

* `CodeBlock` node, or an
* `DivisionBlock` node, or a
* `HeadingBlock` node, or a
* `OrderedListBlock` nodeect, or a
* `ParagraphBlock` node, or a
* `QuoteBlock` node, or an
* `UnorderedListBlock` node.

*The following sections describe the `Block` node of a Markdom document.*

##### Code Block

A `CodeBlock` node is a `Block` node that represents a portion of plain text (e.g. source code) that may be augmented with meaningful syntax highlighting. Implementations that generate output that is displayed to humans should display the text as preformatted text in a monospaced font.

A `CodeBlock` node has a mandatory string parameter named `code` and an optional string parameter named `hint`. 

Values of the `code` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters other then `LINE_FEED` (`\n`) or `CHARACTER_TABULATION` (`\t`). Values of the `code` parameter shouldn't be empty.

Values of the `hint` parameter, if present, should be the common name of a programming or markup language in lowercase (i.e `java`, `php`, `json`, `html`, ...) and are intended to be used by implementations to augment the preformatted text with meaningful syntax highliting.  Values of the `code` parameter, if present, shouldn't be empty.

##### Division Block

A `DivisionBlock` node is a `Block` node that represents a division or thematic break. Implementations that generate output that is displayed to humans should display a horizontal rule.

##### Heading Block

A `HeadingBlock` node is a `Block` node that represents a headline. Implementations that generate output that is displayed to humans should display the headline text as an appropriatly emphasized and separated paragraph.

A `HeadingBlock` node has a mandatory integer parameter named `level` and a sequence of `Content` nodes named `contents`.

Valid values of the `level` parameter are 1 to 6, where a lower value indicates a higher precedence of the headline.

The `contents` sequnece shouldn't be empty.

##### Ordered List Block

An `OrderedListBlock` node is a `Block` node that represents an ordered list (enumeration) of list items. Each list item contains further portions of the Markdom document. Implementations that generate output that is displayed to humans should indent all list items and display the index of each list item in front of it.

An `OrderedListBlock` has a mandatory integer parameter `startIndex` and a sequence of `ListItem` nodes named `items`.

Valid values of the `startIndex` parameter are non-negative integers. The value of the `startIndex` parameter is the inidex of the first item in the `items` sequnece. Following items in the `items` sequence have successive indices.

The `items` sequence shouldn't be empty.

##### Paragraph Block

A `ParagraphBlock` node is a `Block` node that represents a paragraph. Implementations that generate output that is displayed to humans should display the paragraph text as an appropriatly separated paragraph.

A `ParagraphBlock` node has a sequence of `Content` nodes named `contents`.

The `contents` sequnece shouldn't be empty.

##### Quote Block

A `QuoteBlock` node is a `Block` node that represents a quote. The quote contains further portions of the Markdom document. Implementations that generate output that is displayed to humans should emphasize (e.g. a vertical line, indentation, slanted font) the quote appropriatly.

An `QuoteBlock` has a sequence of `Block` nodes named `blocks`.

The `blocks` sequence shouldn't be empty.

##### Unordered List Block

An `UnorderedListBlock` node is a `Block` node that represents an unordered list (bullet list) of list items. Each list item contains further portions of the Markdom document. Implementations that generate output that is displayed to humans should indent all list items and display an appropriate symbol (e.g. a bullet) in front of each item.

An `UnorderedListBlock` has a sequence of `ListItem` nodes named `items`.

The `items` sequence shouldn't be empty.

#### List Item

A `ListItem` node represents a list item. The list item contains further portions of the Markdom document. Implementations that generate output that is displayed to humans should display the list item according to the corresponding `OrderedListBlock` or `UnorderedListBlock`.

A `ListItem` has a sequence of `Block` nodes named `blocks`.

The `blocks` sequence shouldn't be empty.

#### Content

A `Content` node represents a portion of a Markdom paragraph or a Markdom heading.

A `Content` node is either a

* `CodeContent` node, or an
* `EmphasisContent` node, or an
* `ImageContent` node, or an
* `LineBreakContent` node, or a
* `LinkContent` node, or a
* `TextContent` node.

*The following sections describe the `Content` nodes of a Markdom paragraph.*

##### Code Content

A `CodeContent` node is a `Content` node that represents a portion of plain text (e.g. source code). Implementations that generate output that is displayed to humans should display the text as preformatted text in a monospaced font.

A `CodeContent` node has a mandatory string parameter named `code`. 

Values of the `code` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters. Values of the `code` parameter shouldn't be empty.

##### Emphasis Content

An `EmphasisContent` node is a `Content` node that represents emphasized text. Implementations that generate output that is displayed to humans should display text appropriatly emphasized (e.g. italic or bold).

A `EmphasisContent` node has a mandatory integer parameter named `level` and a sequence of `Content` nodes named `contents`.

Valid values of the `level` parameter are 1 to 2, where a higher value indicates a higher importance of the emphasized text.

The `contents` sequnece shouldn't be empty.

##### Image Content

An `ImageContent` node is a `Content` node that represents an image, including a title text and an alternative text. Implementations that generate output that is displayed to humans should display the linked image with the title text or, if the linked image couldn't be resolved, the alternative text as if the `ImageContent` node was a `TextContent` node.

A `LinkContent` node has a mandatory integer parameter named `uri` and an optional string parameter named `title` and an optional string parameter named `alternative`.  

Valid values of the `uri` parameter are valid [URI references](https://tools.ietf.org/html/rfc3986#section-4.1). Values of the `uri` parameter should link to an image resource (e.g. a JPEG or PNG file). Actual applications that process Markdom documents may impose further constraints.

Values of the `title` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters.

Values of the `alternative` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters. Values of the `alternative` parameter shouldn't be empty.

##### Line Break Content

A `LineBreakontent` node is a `Content` node that represents a link. A line break is either a *hard line break* or a *soft line break*. Implementations that generate output that is displayed to humans should display only hard line breaks. Implementations that generate output that is processed as souce code, e.g. as Markdown text, should also display soft line breaks in a way that doesn't introduce line breaks during further processing of the generated source code.

A `LinkContent` node has a mandatory boolean parameter named `hard`.

##### Link Content

A `LinkContent` node is a `Content` node that represents text with a link. Implementations that generate output that is displayed to humans should display text appropriatly emphasized (e.g. underlined or in a different color).

A `LinkContent` node has a mandatory integer parameter named `uri` and a sequence of `Content` nodes named `contents`.

Valid values of the `uri` parameter are valid [URI references](https://tools.ietf.org/html/rfc3986#section-4.1). Actual applications that process Markdom documents may impose further constraints.

The `contents` sequnece shouldn't be empty. A `Content` node in the `contents` sequence must not contain another `LinkContent` node or, recursively, contain a `Content` node that contains a `LinkContent` node.

##### Text Content

A `TextContent` node is a `Content` node that represents a portion of text. Implementations that generate output that is displayed to humans should display the text as preformatted text in an appropriate font.

A `TextContent` node has a mandatory string parameter named `text`. 

Values of the `text` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters. Values of the `text` parameter shouldn't be empty.

### Example document

This CommonMark text describes a rich text document that serves as an example document throughout the rest of this specification:

````
# Markdom

1. [Foo](#Bar)
2. *Lorem ipsum*\
   `goto 11`
3. > Baz

```
goto 11
```
````

The rich text document consists of a heading, an ordered list with three list items and some plain text. The first list item of the ordered list contains a link. the second list item of the unordered list contains emphasized text, a hard line break and some plain text. The third list item of the ordered list contains a quoted text.

The following image shows a tree of Markdom nodes, i.e. a Markdom document, that describes the same rich text document:

![](resource/markdom-example.png)


## APIs

*This sections describes differenst APIs that can be used to work with Markdom documents*

### Overview

This specification describes two distinct APIs:

1. The *Domain Object API* describes a set of interfaces that describe how a Markdom document is represented as an object graph and methods to compose and consume such an object graph.

   The Domain Object API allows to create a representation of a Markdom document in the memory, which can then be examined, modified and further processed.
2. The *Handler API* describes a `Handler` interface that describes how a Markdom document is represented as a sequence of events.

   The Handler API allows to process a Markdom document on the fly whithout the necessity to create an object graph.
   

### Domain Object API {#api-dom}

*This section describes the Domain Object API*

#### Classes used by the Domain Object API

*This section lists the interfaces, enumerations and classes that are used by the Domain Object API.*

The Domain Object API uses the following classes:

// hint**

// arrays (var arg), iterator and iterable

// source

// sequence

// primitives (int, ), unicode string, 


#### Interfaces {#api-dom-interfaces}

The Domain Object API has the following interfaces:

* `Block` objects represent Markdom block nodes.  
  `Block` objects are a `Node` objects.
* `BlockParent` objects represent Markdom nodes that contain `Block` objects.  
  `BlockParent` objects are a `Node` objects.
* `CodeBlock` objects represent Markdom code blocks.  
  `CodeBlock` objects are `Block` objects.
* `CodeContent` objects represent Markdom code content.  
  `CodeContent` objects are `Content` objects.
* `Content` objects represent Markdom content nodes.  
  `Content` objects are `Node` objects.
* `ContentBlock` objects represent Markdom blocks that contain Markdom content.  
  `ContentBlock` objects are `Block` object and `ContentParent` objects.
* `ContentContent` objects represent Markdom contents that contain Markdom content.  
  `ContentContent` objects are `Content` object and `ContentParent` object.
* `ContentParent` objects represent Markdom nodes that contain Markdom content.  
  `ContentParent` objects are `Node` objects.
* `DivisionBlock` objects represent Markdom division blocks.  
  `DivisionBlock` are `Block` objects.
* `Document` objects represent Markdom documents.  
  `Document` objects are `BlockParent` objects.
* `EmphasisContent` objects represent Markdom emphasis contents.  
  `EmphasisContent` objects are `ContentContent` objects.
* `HeadingBlock` objects represent Markdom heading blocks.  
  `HeadingBlock` objects are `ContentBlock` objects.
* `ImageContent` objects represent Markdom image contents.  
  `ImageContent` objects are `Content` objects.
* `LineBreakContent` objects represent Markdom line break contents.  
  `LineBreakContent` objects are `Content` objects.
* `LinkContent` objects represent Markdom link contents.  
  `LinkContent` objects are `ContentContent` objects.
* `ListBlock` objects represent Markdom list blocks.  
  `ListBlock` objects are `Block` objects.
* `ListItem` objects represent Markdom list items.  
  `ListItem` objects represent are `BlockParent` objects.
* `Node` objects represent [Markdom nodes](#domain-node).
* `OrderedListBlock` objects represent an ordered Markdom list blocks.  
  `OrderedListBlock` objects are `ListBlock` objects.
* `ParagraphBlock` objects represent Markdom paragraph blocks.  
  `ParagraphBlock` objects are `ContentBlock` objects.
* `QuoteBlock` objects represent Markdom quote blocks.  
  `QuoteBlock` objects are `BlockParent` objects.
* `TextContent` objects represent Markdom text contents.  
  `TextContent` objects are `Content` objects.
* `UnorderedListBlock` object represent an unordered Markdom list blocks.  
  `UnorderedListBlock` object are `ListBlock` objects.

The methods of theese interfaces are specified below.

The following image shows the interfaces that are part of the Domain Object API as well as the specified interface inheritances and the specified compositions:

![](resource/markdom-nodes.png)


### Handler API

### Combining the Domain Object API and the Handler API

// document is dispatcher, 

// markdom document markdom handler

## Data representations

### JSON

### YAML

### XML