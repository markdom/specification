# ![logo](resource/markdom-mark-negative-inline.png) Markdom

Markdom is lightweight specification for a rich text domain model.

The two main intents of Markdom are to define the lowest common denominator for rich text and to be platform and representation independent. Theese properties enable Markdom to be used in a wide variety of applications, such as websites, electronic publishing or native rendering on mobile platforms.

## Overview




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

## Introduction 

```
# Markdom

1. Allow an editor to enter rich text in a human friendly way.
2. Transmit the rich text to different platforms, using existing technologies.
3. Display the rich text on different platforms in a native way.
```


## Domain {#domain}

*This sections describes the general structure of a Markdom document, independent of any representation.*

### Overview

A Markdom document represents the entirety of a rich text document, including the actual rich text and the formatting instructions. A Markdom document contains of a sequence of Markdom block.

A Markdom block represents a portion of a rich text document that serves a specific purpose. Each type of Markdom block has the necessary properties that describe it's content. Some Markdom blocks, e.g. a Markdom block representing a blockm quote, contain further Markdom blocks. Other Markdom blocks, e.g. a Markdom block representing a paragraph, contain Markdom content.

A Markdom content represents a portion of rich text that serves a specific purpose. Each type of Markdom block has the necessary properties that describe it's content. Some Markdom contents, e.g. a Markdom block representing a link, contain further Markdom content. A Markdom content never contains a Markdom block.

Put another way, a Markdom document is represented as a tree of Markdom nodes. Starting with a root node that represents an entire rich text document, each markdom node represents a portion of the corresponding Markdom document.

### Nodes {#domain-node}

A *Markdom block* is either a

* *Markdom document*, or a
* *Markdom block*, or a
* *Markdom list item*, or a
* *Markdom content*.

#### Document {#domain-document}

A *Markdom document* is a *Markdom node* that represents the entirety of a rich text document. A *Markdom document* is the root node of a tree of *Markdom nodes*.

A *Markdom document* contains a sequence of *Markdom blocks* named `blocks`.

The `blocks` sequence shouldn't be empty.

#### Block {#domain-block}

A *Markdom block* is a *Markdom node* that represents a portion of a rich text document.

A *Markdom block* is either a

* *Markdom code block*, or a
* *Markdom division block*, or a
* *Markdom heading block*, or an
* *ordered Markdom list block*, or a
* *Markdom paragraph block*, or a
* *Markdom quote block*, or an
* *ordered Markdom list bliock*.

##### Code Block {#domain-codeblock}

A *Markdom code block* is a *Markdom block* that represents a portion of plain text (e.g. source code) that may be augmented with meaningful syntax highlighting. Implementations that generate output that is displayed to humans should display the text as preformatted text in a monospaced font.

A *Markdom code block* has a mandatory string parameter named `code` and an optional string parameter named `hint`. 

Values of the `code` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters other then `LINE_FEED` (`\n`) or `CHARACTER_TABULATION` (`\t`). Values of the `code` parameter shouldn't be empty.

Values of the `hint` parameter, if present, should be the common name of a programming or markup language in lowercase (i.e `java`, `php`, `json`, `html`, ...) and are intended to be used by implementations to augment the preformatted text with meaningful syntax highliting.  Values of the `code` parameter, if present, shouldn't be empty.

##### Division Block {#domain-divisionblock}

A *Markdom division block* is a *Markdom block* that represents a division or thematic break. Implementations that generate output that is displayed to humans should display a horizontal rule.

##### Heading Block {#domain-headingblock}

A *Markdom heading block* is a *Markdom block* that represents a headline. Implementations that generate output that is displayed to humans should display the headline text as an appropriatly emphasized and separated paragraph.

A *Markdom heading block* has a mandatory integer parameter named `level` and a sequence of *Markdom contents* named `contents`.

Valid values of the `level` parameter are 1 to 6, where a lower value indicates a higher precedence of the headline.

The `contents` sequnece shouldn't be empty.

##### Ordered List Block {#domain-orderedlistblock}

An *ordered Markdom list block* is a *Markdom block* that represents an ordered list (enumeration) of list items. Each list item contains further portions of the Markdom document. Implementations that generate output that is displayed to humans should indent all list items and display the index of each list item in front of it.

An *ordered Markdom list block* has a mandatory integer parameter `startIndex` and a sequence of *Markdom list items* named `items`.

Valid values of the `startIndex` parameter are non-negative integers. The value of the `startIndex` parameter is the inidex of the first item in the `items` sequnece. Following items in the `items` sequence have successive indices.

The `items` sequence shouldn't be empty.

##### Paragraph Block {#domain-paragraphblock}

A *Markdom pagargraph block* is a *markdom block* that represents a paragraph. Implementations that generate output that is displayed to humans should display the paragraph text as an appropriatly separated paragraph.

A *Markdom pagargraph block* has a sequence of *Markdom contents* named `contents`.

The `contents` sequnece shouldn't be empty.

##### Quote Block {#domain-quoteblock}

A *Markdom quote block* is a *Markdom block* that represents a quote. The quote contains further portions of the Markdom document. Implementations that generate output that is displayed to humans should emphasize (e.g. a vertical line, indentation, slanted font) the quote appropriatly.

An `QuoteBlock` has a sequence of *Markdom blocks* named `blocks`.

The `blocks` sequence shouldn't be empty.

##### Unordered List Block {#domain-unorderedlistblock}

An *unordered Markdom list block* is a *Markdom block* that represents an unordered list (bullet list) of list items. Each list item contains further portions of the Markdom document. Implementations that generate output that is displayed to humans should indent all list items and display an appropriate symbol (e.g. a bullet) in front of each item.

An `UnorderedListBlock` has a sequence of *Markdom list items* named `items`.

The `items` sequence shouldn't be empty.

#### List Item {#domain-listitem}

A *Markdom list item* represents a list item. The list item contains further portions of the Markdom document. Implementations that generate output that is displayed to humans should display the list item according to the corresponding `OrderedListBlock` or `UnorderedListBlock`.

A `ListItem` has a sequence of *Markdom blocks* named `blocks`.

The `blocks` sequence shouldn't be empty.

#### Content {#domain-content}

A *Markdom content* represents a portion of a Markdom paragraph or a Markdom heading.

A *Markdom content* is either a

* *Markdom code content*, or an
* *Markdom emphasis content*, or an
* *Markdom image content*, or an
* *Markdom line break content*, or a
* *Markdom link content*, or a
* *Markdom text content*.

*The following sections describe the *Markdom contents* of a Markdom paragraph.*

##### Code Content {#domain-codecontent}

A *Markdom code content* is a *Markdom content* that represents a portion of plain text (e.g. source code). Implementations that generate output that is displayed to humans should display the text as preformatted text in a monospaced font.

A *Markdom code content* has a mandatory string parameter named `code`. 

Values of the `code` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters. Values of the `code` parameter shouldn't be empty.

##### Emphasis Content {#domain-emphasiscontent}

An *Markdom emphasis content* is a *Markdom content* that represents emphasized text. Implementations that generate output that is displayed to humans should display text appropriatly emphasized (e.g. italic or bold).

A *Markdom emphasis content* has a mandatory integer parameter named `level` and a sequence of *Markdom contents* named `contents`.

Valid values of the `level` parameter are 1 to 2, where a higher value indicates a higher importance of the emphasized text.

The `contents` sequnece shouldn't be empty.

##### Image Content {#domain-imagecontent}

An *Markdom image content* is a *Markdom content* that represents an image, including a title text and an alternative text. Implementations that generate output that is displayed to humans should display the linked image with the title text or, if the linked image couldn't be resolved, the alternative text as if the *Markdom image content* was a *Markdom text content*.

A *Markdom link content* has a mandatory integer parameter named `uri` and an optional string parameter named `title` and an optional string parameter named `alternative`.  

Valid values of the `uri` parameter are valid [URI references](https://tools.ietf.org/html/rfc3986#section-4.1). Values of the `uri` parameter should link to an image resource (e.g. a JPEG or PNG file). Actual applications that process Markdom documents may impose further constraints.

Values of the `title` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters.

Values of the `alternative` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters. Values of the `alternative` parameter shouldn't be empty.

##### Line Break Content {#domain-linebreakcontent}

A *Markdom line break content* is a *Markdom content* that represents a link. A line break is either a *hard line break* or a *soft line break*. Implementations that generate output that is displayed to humans should display only hard line breaks. Implementations that generate output that is processed as souce code, e.g. as Markdown text, should also display soft line breaks in a way that doesn't introduce line breaks during further processing of the generated source code.

A *Markdom line break content* has a mandatory boolean parameter named `hard`.

##### Link Content {#domain-linkcontent}

A *Markdom link content* is a *Markdom content* that represents text with a link. Implementations that generate output that is displayed to humans should display text appropriatly emphasized (e.g. underlined or in a different color).

A *Markdom link content* has a mandatory integer parameter named `uri` and a sequence of *Markdom contents* named `contents`.

Valid values of the `uri` parameter are valid [URI references](https://tools.ietf.org/html/rfc3986#section-4.1). Actual applications that process Markdom documents may impose further constraints.

The `contents` sequnece shouldn't be empty. A *Markdom content* in the `contents` sequence must not contain another *Markdom link content* or, recursively, contain a *Markdom content* that contains a *Markdom link content*.

##### Text Content {#domain-textcontent}

A *Markdom text content* is a *Markdom content* that represents a portion of text. Implementations that generate output that is displayed to humans should display the text as preformatted text in an appropriate font.

A *Markdom text content* has a mandatory string parameter named `text`. 

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

The following image shows a tree of *Markdom nodes*, i.e. a Markdom document, that describes the same rich text document:

![](resource/markdom-example.png)





## APIs {#api}

*This sections describes differenst APIs that can be used to work with Markdom documents*

### Overview

This specification describes two distinct APIs:

1. The *Domain Object API* describes a set of interfaces that describe how a Markdom document is represented as an object graph and methods to compose and consume such an object graph.

   The Domain Object API allows to create a representation of a Markdom document in the memory, which can then be examined, modified and further processed.
2. The *Handler API* describes a `Handler` interface that describes how a Markdom document is represented as a sequence of events.

   The Handler API allows to process a Markdom document on the fly whithout the necessity to create an object graph.
   
### Implementation guideline


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
  `Block` objects are a [`Node`](#api-dom-node) objects.
* `BlockParent` objects represent Markdom nodes that contain `Block` objects.  
  `BlockParent` objects are a [`Node`](#api-dom-node) objects.
* `CodeBlock` objects represent Markdom code blocks.  
  `CodeBlock` objects are `Block` objects.
* `CodeContent` objects represent Markdom code content.  
  `CodeContent` objects are `Content` objects.
* `Content` objects represent Markdom content nodes.  
  `Content` objects are [`Node`](#api-dom-node) objects.
* `ContentBlock` objects represent Markdom blocks that contain Markdom content.  
  `ContentBlock` objects are `Block` object and `ContentParent` objects.
* `ContentContent` objects represent Markdom contents that contain Markdom content.  
  `ContentContent` objects are `Content` object and `ContentParent` object.
* `ContentParent` objects represent Markdom nodes that contain Markdom content.  
  `ContentParent` objects are [`Node`](#api-dom-node) objects.
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
* [`Node`](#api-dom-node) objects represent [Markdom nodes](#domain-node).
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

The methods of theese interfaces are specified [below](#api-dom-block).

The following image shows the interfaces that are part of the Domain Object API as well as the specified interface inheritances and the specified compositions:

![](resource/markdom-nodes.png)

##### `Block` {#api-dom-block}

###### Constructors {api-dom-block-constructor}

An implementation of `Block` should have a constructor with signature `Block()`.

###### `getBlockType` {api-dom-block-getblocktype}

A `Block` object must have a method with signature `BlockType getBlockType()`.  

This method must return the `BlockType` value that correspondes to the type of the `Block` object.

##### `BlockParent` {#api-dom-blockparent}

An implementation of `BlockParent` must have a final and initially empty `Sequence` of `Block` objects that is associated with the `BlockParent` object.

Any structural modification (insert, remove, clear, replace) to the associated `Sequence` of `Block` objects must reflect the fact, that a `Block` objcets that is added to the associated `Sequence` object is attached to the `BlockParent` object until is is removed from the associated `Sequence` of `Block` objects.

If a `Block` object is attached to the `BlockParent` object, the `BlockParent` object is returned by `Block#getParent()`. 

Attaching a `Block` object to the `BlockParent` object (i.e. adding the `Block` object to the associated `Sequence` of `Block` objects) must fail
* if the `Block` object is not present, or  
* if the `Block` object is already attached to a `BlockParent` object, or 
* if attaching the `Block` object to the `BlockParent` object would create a cycle in the tree of Markdom nodes that the `BlockParent` object is part of.

###### Constructors

An implementation of `BlockParent` should have a constructor with signature `BlockParent()`.

For convenience, an implementation of `BlockParent` should have a constructor with signature `BlockParent(Block... blocks)` that takes that takes an array of `Block` objects named `blocks` and delegates to `addBlocks(Block... blocks)`.

###### `getBlocks` 

A `Block` object must have a method with signature `Sequence getBlocks()`.

This method must return the associated `Sequence` of `Block` objects.

###### `addBlock`

For convenience, a `BlockParent` object should have a method with signature `addBlock(Block block)` that takes a `Block` object named `block`.

This method must add `block` at the end of the associated `Sequence` of `Block` objects. This attaches `block` to the `BlockParent` object.

This method must fail if add `block` to the  associated `Sequence` of `Block` objects failed.

###### `addBlocks`

For convenience, a `BlockParent` object should have a method with signature `addBlocks(Block... blocks)` that takes an array of `Block` objects named `blocks`.

This method must add all `Block` objects from `blocks` in the given order at the end of the `Sequence` of `Block` objects of the `BlockParent` object, as if `addBlock(Block block)` has been called repeatedly for all `Block` objects from `blocks`.  This attaches all `Block` objects from `blocks` to the `BlockParent` object.
  
This method must fail if `blocks` is not present.
This method must fail if adding any `Block` object from `blocks` to the  associated `Sequence` of `Block` objects failed.
  
Because this method is a short hand for repeated calls to `addBlock(Block block)`, it must add all prior `Block` objects from `blocks` to the  associated `Sequence` of `Block` objects, if it fails beacause of a violating `Block` object from `blocks`.

##### `CodeBlock` {#api-dom-codeblock}

###### Constructors

An implementation of `CodeBlock` should have a constructor with signature `CodeBlock()` that set the code of the `CodeBlock` object to the empty string ant the hint of the `Code` object to be not present.

For convenience, an implementation of `CodeBlock` should have a constructor with signature `CodeBlock(String code)` that takes a `String` named `code` and delegates to `setCode(String code)`.

For convenience, an implementation of `CodeBlock` should have a constructor with signature `CodeBlock(String code, String hint)` that takes a `String` named `code` and a `String` named `hint` and delegates to `setCode(String code)` and `setHint(String hint)`.

###### `getCode`

A `CodeBlock` object must have a method with signature `String getCode()`.

The method must return the code of the `CodeBlock` object.

###### `setCode`

A `CodeBlock` object must have a method with signature `setCode(String code)` that takes a `String` named `code`.

This method must set the code of the `CodeBlock` object. 
  
This method must fail if `code` is not present.

###### `getHint`

A `CodeBlock` object must have a method with signature `String? getHint()`.

The method must return the optional hint of the `CodeBlock` object.

###### `setHint`
  
A `CodeBlock` object must have a method with signature `setHint(String? hint)` that takes an optional `String` named `hint`.

This method must set the hint of the `CodeBlock` object. 
 	
##### `CodeContent` {#api-dom-codecontent}

###### Constructors

An implementation of `CodeContent` should have a constructor with signature `CodeContent()` that set the code of the `CodeContent` object to the empty string.

For convenience, an implementation of `CodeContent` should have a constructor with signature `CodeContent(String code)` that takes a `String` named `code` and delegates to `setCode(String code)`.

###### `setCode`

A `CodeContent` object must have a method with signature `setCode(String code)` that takes a `String` named `code`.

This method must set the code of the `CodeContent` object. 
  
This method must fail if `code` is not present.

##### `Content` {#api-dom-content}

###### Constructors 

An implementation of `Content` should have a constructor with signature `Content()`.

###### `getContentType`

A `Content` object must have a method with signature `ContentType getContentType()`.  

This method must return the `ContentType` value that correspondes to the type of the `Content` object.

###### `getParent`

A `Content` object must have a method with signature `ContentParent getParent()`.

This method specializes the corresponding method from the `Node` interface.  
This method must return the `ContentParent` object the `Content` object is currently added to.

This method must fail if the `Content` object is currently not added to a `ContentParent` object.

###### `getBlock`

A `Content` object must have a method with signature `Block getBlock()`.

This method must return the nearest `Block` object object the `Content` in the tree of Markdom nodes  the `Content` object is part of, which is the `Block` object of the `ContentParent` this `Content` object is currently added to.

This method must fail if the `Content` object is currently not added to a `ContentParent` object.
   
##### `ContentBlock` {#api-dom-contentblock}

###### Constructors

An implementation of `ContentBlock` should have a constructor with signature `ContentBlock()`.

For convenience, an implementation of `ContentBlock` should have a constructor with signature `ContentBlock(Content... contents)` that takes that takes an array of `Content` objects named `contents` and delegates to `ContentParent#addContents(Content... content)`.

##### `ContentContent` {#api-dom-contentcontent}

###### Constructors

An implementation of `ContentParent` should have a constructor with signature `ContentContent()`.

For convenience, an implementation of `ContentContent` should have a constructor with signature `ContentContent(Content... contents)` that takes that takes an array of `Content` objects named `contents` and delegates to `ContentParent#addContents(Content... content)`.
	
##### `ContentParent` {#api-dom-contentparent}

###### Constructors

An implementation of `ContentParent` should have a constructor with signature `ContentParent()`.

For convenience, an implementation of `ContentParent` should have a constructor with signature `ContentParent(Content... contents)` that takes that takes an array of `Content` objects named `contents` and delegates to `addContents(Content... content)`.

###### `addContent`

For convenience, a `ContentParent` object should have a method with signature `addContent(Content content)` that takes a `Content` object named `content`.

This method must add `content` as the last Markdom content of the `ContentParent` object.

This method must fail if `content` is not present.  
This method must fail if `content` is already added to a `ContentParent` object.  
This method must fail if adding `content` to the `ContentParent` object would create a cycle in the tree of Markdom nodes that the `ContentParent` object is part of.

###### `addContents`

For convenience, a `ContentParent` object should have a method with signature `addContents(Content... contents)` that takes an array of `Content` objects named `contents`.

This method must add all `Content` objects from `contents` to the `ContentParent` object.  
This method is a short hand for repeated calls to `addContent(Content content)` and should call `addContent(Content content)` for all `Content` objects from `contents` to the `ContentParent` object.
  
This method must fail if `contents` is not present.  
This method must fail if any of the `Content` objects from `contents` is not present.  
This method must fail if any of the `Content` objects from `contents` is already added to a `ContentParent` object.  
This method must fail if adding any of the `Content` objects from `contents` to the `ContentParent` object would create a cycle in the tree of Markdom nodes that the `ContentParent` object is part of.
  
Because this method is a short hand for repeated calls to `addContent(Content content)`, it must add all prior `Content` objects from `contents` to the `ContentParent` object, if it fails beacause of a violating `Content` object from `contents`.
  
##### `Document` {#api-dom-document}

###### Constructors

For convenience, an implementation of `Document` should have a constructor with signature `Document(Block... blocks)` that takes that takes an array of `Block` objects and delegates to `BlockParent#addBlocks(Block... blocks)`.

##### `EmphasisContent` {#api-dom-emphasiscontent}

An implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent()` that sets the level of the `EmphasisContent` object to `LEVEL_1`.

For convenience, an implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent(EmphasisLevel level)` that takes an `EmphasisLevel` named `level` and delegates to `setLevel(EmphasisLevel level)`.

For convenience, an implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent(EmphasisLevel level, Content... contents)` that takes an `EmphasisLevel` named `level` and an array of `Content` objects named `contents` and delegates to `setLevel(EmphasisLevel level)` and `ContentParent#addContents(Content... contents)`.

###### `setLevel`

An `EmphasisContent` object must have a method with signature `setLevel(EmphasisLevel level)` that takes an `EmphasisLevel` named `level`.

This method must set the level of the `EmphasisContent` object. 
  
This method must fail if `level` is not present.
   
##### `HeadingBlock` {#api-dom-hedingblock}

###### Constructors

An implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock()` that sets the level of the `HeadingBlock` object to `LEVEL_1`.

For convenience, an implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock(HeadingLevel level)` that takes a `HeadingLevel` named `level` and delegates to `setLevel(HeadingLevel level)`.

For convenience, an implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock(HeadingLevel level, Content... contents)` that takes a `HeadingLevel` named `level` and an array of `Content` objects named `contents` and delegates to `setLevel(HeadingLevel level)` and `ContentParent#addContents(Contents... contents)`.

###### `setLevel`

A `HeadingBlock` object must have a method with signature `setLevel(HeadingLevel level)` that takes a `HeadingLevel` named `level`.

This method must set the level of the `HeadingBlock` object. 
  
This method must fail if `level` is not present.

   
##### `ImageContent` {#api-dom-imagecontent}

###### Constructors 

An implementation of `LinkContent` should have a constructor with signature `LinkContent()` that sets the uri of the `ImageContent` object to the empty string and the title of the `ImageContent` object to be not present and the alternative text of the `ImageContent` object to be not present.

For convenience, an implementation of `ImageContent` should have a constructor with signature `ImageContent(String uri)` that takes a `String` named `uri` and delegates to `setUri(String uri)`.

For convenience, an implementation of `ImageContent` should have a constructor with signature `ImageContent(String uri, String? title, String? alternative)` that takes a `String` named `uri` and an optional `String` named `title` and an optional `String` named `alternative `and delegates to `setUri(String uri)` and `setTitle(String? title)` and `setAlternative(String? alternative)`.

###### `setUri`

An `ImageContent` object must have a method with signature `setUri(String uri)` that takes a `String` named `level`.

This method must set the uri of the `ImageContent` object. 
  
This method must fail if `uri` is not present.

###### `setTitle`
  
An `ImageContent` object must have a method with signature `setTitle(String? title)` that takes an optional `String` named `title`.

This method must set the title of the `ImageContent` object. 

###### `setAlternative`

An `ImageContent` object must have a method with signature `setAlternative(String? alternative)` that takes an optional `String` named `alternative`.

This method must set the alternative text of the `ImageContent` object.

##### `LineBreakContent` {#api-dom-linebreakcontent}

###### Constructors

An implementation of `LineBreakContent` should have a constructor with signature `LineBreakContent()` that sets the `LinkContent` object to represent a soft line break.

For convenience, an implementation of `LineBreakContent` should have a constructor with signature `LinkContent(Boolean hard)` that takes a `Boolean` named `hard` and delegates to `setHard(Boolean hard)`.

###### `setHard`

A `LineBreakContent` object must have a method with signature `setHard(Boolean hard)` that takes a `Boolean` named `hard`.

This method must set whether the `LinkContent` object represents a hard line break or a soft line break.. 
  
This method must fail if `hard` is not present.
   
##### `LinkContent` {#api-dom-linkcontent}

###### Constructors

An implementation of `LinkContent` should have a constructor with signature `LinkContent()` that sets the uri of the `LinkContent` object to the empty string.

For convenience, an implementation of `LinkContent` should have a constructor with signature `LinkContent(String uri)` that takes a `String` named `uri` and delegates to `setUri(String uri)`.

For convenience, an implementation of `LinkContent` should have a constructor with signature `LinkContent(String uri, Content... contents)` that takes a `String` named `uri` and an array of `Content` objects named `contents` and delegates to `setUri(String uri)` and `ContentParent#addContents(Content... contents)`.

###### `setUri`

A `LinkContent` object must have a method with signature `setUri(String uri)` that takes a `String` named `level`.

This method must set the uri of the `LinkContent` object. 
  
This method must fail if `uri` is not present.
   
##### `ListBlock` {#api-dom-listblock}

###### Constructors 

An implementation of `ListBlock` should have a constructor with signature `ListBlock()`.

For convenience, an implementation of `ListBlock` should have a constructor with signature `ListBlock(ListItem... items)` that takes that takes an array of `ListItem` objects named `items` and delegates to `addItems(ListItem... items)`.

###### `addItem`

For convenience, a `ListItem` object should have a method with signature `addItem(ListItem item)` that takes a `ListItem` object named `item`.

This method must add `item` as the last Markdom item of the `ListItem` object.

This method must fail if `item` is not present.  
This method must fail if `item` is already added to a `ListItem` object.  
This method must fail if adding `item` to the `ListItem` object would create a cycle in the tree of Markdom nodes that the `ListItem` object is part of.

###### `addItems`

For convenience, a `ListItem` object should have a method with signature `addItems(ListItem... items)` that takes an array of `ListItem` objects named `items`.

This method must add all `ListItem` objects from `items` to the `ListItem` object.  
This method is a short hand for repeated calls to `ListItem(ListItem item)` and should call `ListItem(ListItem item)` for all `ListItem` objects from `items` to the `ListItem` object.
  
This method must fail if `items` is not present.  
This method must fail if any of the `ListItem` objects from `items` is not present`.  
This method must fail if any of the `ListItem` objects from `items` is already added to a `ListItem` object.  
This method must fail if adding any of the `ListItem` objects from `items` to the `ListItem` object would create a cycle in the tree of Markdom nodes that the `ListItem` object is part of.
  
Because this method is a short hand for repeated calls to `ListItem(ListItem item)`, it must add all prior `ListItem` objects from `items` to the `ListItem` object, if it fails beacause of a violating `ListItem` object from `items`.

##### `ListItem` {#api-dom-listitem}

###### Constructors

An implementation of `ListItem` should have a constructor with signature `ListItem()`.

For convenience, an implementation of `ListItem` should have a constructor with signature `ListItem(Block... blocks)` that takes an array of `Block` objects named `blocks` and delegates to `BlockParent#addBlocks(Block... blocks)`.

##### `Node` {#api-dom-node}

###### `getNodeType` {#api-dom-node-getnodetype}

A [`Node`](#api-dom-node) object must have a method with signature `NodeType getNodeType()`.  

This method must return the [`NodeType`](#api-dom-nodetype) value that correspondes to the type of the [`Node`](#api-dom-node) object.

###### `hasParent` {#api-dom-node-hasparent}

A [`Node`](#api-dom-node) object must have a method with signature `Boolean hasParent()`.

This method must return whether the `Node` has a parent [`Node`](#api-dom-node) object.  

Specifically, this method must return `true`
* if the [`Node`](#api-dom-node) object is a `Block` object and currently attached to a `BlockParent` object, or
* if the [`Node`](#api-dom-node) object is a `ListItem` object and currently attached to a `ListBlock` object, or
* if the [`Node`](#api-dom-node) object is a `Content` object and currently attached to a `ContentParent` object.

Specifically, this method must return `false`
* if the [`Node`](#api-dom-node) object is a `Document`, or
* if the [`Node`](#api-dom-node) object is a `Block` object and currently not attached to a `BlockParent` object, or
* if the [`Node`](#api-dom-node) object is a `ListItem` object and currently not attached to a `ListBlock` object, or
* if the [`Node`](#api-dom-node) object is a `Content` object and currently not attached to a `ContentParent` object.


###### `getParent` {#api-dom-node-getparent}

A [`Node`](#api-dom-node) object must have a method with signature `Node getParent()`.

This method must return the parent [`Node`](#api-dom-node) object of the [`Node`](#api-dom-node) object. 

Specifically, this method must return 
* the `BlockParent` object it is currently attached to, if the [`Node`](#api-dom-node) object is a `Block` object, or
* the `ListBlock` object it is currently attached to, if the [`Node`](#api-dom-node) object is a `ListItem` object, or
* the `ContentParent` object it is currently attached, if the [`Node`](#api-dom-node) object is a `Content` object.

This method must fail if the `Node` doesn't [have](#api-dom-node-hasparent) a [parent](#api-dom-node-getparent) [`Node`](#api-dom-node) object.

###### `getDocument` {#api-dom-node-getdocument}

A [`Node`](#api-dom-node) object must have a method with signature `Document getDocument()`.

This method must return the `Document` object that is the root of the tree of [Markdom nodes](#domain-node) the [`Node`](#api-dom-node) object is part of.

Specifically, this method must return 
* the `Node` object, if the [`Node`](#api-dom-node) object is a `Document` object, or
* the `Document` object of the parent `Node` object, if the `Node` object has a parent `Node` object.
  
This method must fail if the `Node` doesn't [have](#api-dom-node-hasparent) a [parent](#api-dom-node-getparent) [`Node`](#api-dom-node) object.

###### `getIndex` {#api-dom-node-getindex}

A `Node` object must have a method with signature `Integer getIndex()`.

This method must return the index of the `Node` object in the `Source` of child `Node` object of the parent `Node`.

Specifically, this method must return 
* the index in the attached `Sequence` of `Block` objects of the `BlockParent` object it is currently attached to, if the [`Node`](#api-dom-node) object is a `Block` object, or
* the index in the attached `Sequence` of `ListItem` objects of the `ListBlock` object it is currently attached to, if the [`Node`](#api-dom-node) object is a `ListItem` object, or
* the index in the attached `Sequence` of `Content` objects of the `ContentParent` object it is currently attached to, if the [`Node`](#api-dom-node) object is a `Content` object, or

This method must fail if the `Node` doesn't [have](#api-dom-node-hasparent) a [parent](#api-dom-node-getparent) [`Node`](#api-dom-node) object.

###### `hasChildren`

A `Node` object must have a method with signature `Integer getIndex()`.

This method must return whether the `Node` object has child `Node` objects.

Specifically, this method must return `true`
* if the [`Node`](#api-dom-node) object is a `BlockParent` object and currently has a `Block` object attached to it, or
* if the [`Node`](#api-dom-node) object is a `ListBlock` object and currently has a `ListItem` object attached to it, or
* if the [`Node`](#api-dom-node) object is a `ContentParent` object and currently has a `Content` object attached to it.

Specifically, this method must return `false`
* if the [`Node`](#api-dom-node) object is a `BlockParent` object and currently has no `Block` object attached to it, or
* if the [`Node`](#api-dom-node) object is a `ListBlock` object and currently has no `ListItem` object attached to it, or
* if the [`Node`](#api-dom-node) object is a `ContentParent` object and currently has no `Content` object attached to it, or
* if the [`Node`](#api-dom-node) object is a `CodeBlock` object or a `CodeContent` object or a `DivisionBlock` object or an `ImageContent` object or a `LineBreakContent` object or a `TextContent` object.

###### `getChildren`

A `Node` object must have a method with signature `Source getChildren()`.

This method must return a `Source` object that yields the child `Node` objects of the `Node` object.

Specifically, this method must return `true`
* the attached `Sequence` of `Block` objects, if the [`Node`](#api-dom-node) object is a `BlockParent` object, or
* the attached `Sequence` of `ListItem` objects, if the [`Node`](#api-dom-node) object is a `ListBlock` object, or
* the attached `Sequence` of `Content` objects, if the [`Node`](#api-dom-node) object is a `ContentParent` object, or
* an emty `Source` object, if the [`Node`](#api-dom-node) object is a `CodeBlock` object or a `CodeContent` object or a `DivisionBlock` object or an `ImageContent` object or a `LineBreakContent` object or a `TextContent` object.


##### `OrderedListBlock` {#api-dom-orderedlistblock}

###### Constructors

An implementation of `OrderedListBlock` should have a constructor with signature `OrderedListBlock()` that sets the start index of the `OrderedListBlock` object to `1`.

For convenience, an implementation of `OrderedListBlock` should have a constructor with signature `OrderedListBlock(Integer startIndex)` that takes an `Integer` named `startIndex` and delegates to `setStartIndex(Integer startIndex)`.

For convenience, an implementation of `OrderedListBlock` should have a constructor with signature `OrderedListBlock(Integer startIndex, ListItem... items)` that takes an `Integer` named `startIndex` and an array of `ListItem` objects named `items` and delegates to `setStartINdex(Integer startIndex)` and `ContentParent#addContents(Content... contents)`.

###### setStartIndex

An `OrderedListBlock` object must have a method with signature `setStartIndex(Integer startIndex)` that takes an `INteger` named `startIndex`.

This method must set the start index of the `OrderedListBlock` object. 
  
This method must fail if `startIndex` is not present.  
This method must fail if `startIndex` is negative.
  
##### `ParagraphBlock` {#api-dom-paragraphblock}

###### Constructors

An implementation of `ParagraphBlock` should have a constructor with signature `ParagraphBlock()`.

For convenience, an implementation of `ParagraphBlock` should have a constructor with signature `ParagraphBlock(Content... contents)` that takes an array of `Content` objects named `contents` and delegates to `ContentBlock#addContents(Contents... contents)`.

##### `QuoteBlock` {#api-dom-quoteblock}

###### Constructors

An implementation of `QuoteBlock` should have a constructor with signature `QuoteBlock()`.

For convenience, an implementation of `QuoteBlock` should have a constructor with signature `QuoteBlock(Block... blocks)` that takes an array of `Block` objects named `blocks` and delegates to `BlockParent#addBlocks(Block... blocks)`.
    
##### `TextContent` {#api-dom-textcontent}

###### Constructors

An implementation of `TextContent` should have a constructor with signature `TextContent()` that sets the text of the `TextContent` object to the empty string.
   
For convenience, an implementation of `TextContent` should have a constructor with signature `TextContent(String text)` that takes a `String` named `text` and delegates to `setText(String text)`.

###### `setText`

A `TextContent` object must have a method with signature `setText(String text)` that takes a `String` named `text`.

This method must set the text of the `TextContent` object. 
  
This method must fail if `text` is not present.
   
##### `UnorderedListBlock` {#api-dom-unorderedlistblock}

###### Constructors

An implementation of `UnorderedListBlock` should have a constructor with signature `UnorderedListBlock()`.
   
For convenience, an implementation of `UnorderedListBlock` should have a constructor with signature `UnorderedListBlock(ListItem... items)` that takes an array of `ListItem` objects named `items` and delegates to  `ListBlock#addItems(ListItem... items)`.


#### Enumerations {#api-dom-enumerations}

The Domain Object API has the following enumerations:

##### `BlockType` {#api-dom-blocktype}

The `BlockType` enum represents the node type of a [`Block`](#api-dom-block) object and has the following constants:

* `CODE`,
* `DIVISION`,
* `HEADING`,
* `ORDERED_LIST`,
* `PARAGRAPH`,
* `QUOTE`,
* `UNORDERED_LIST`.
  
##### `ContentType` {#api-dom-contenttype}  

The `ContentType` enum represents the node type of a [`Content`](#api-dom-content) object and has the following constants:

* `CODE`,
* `EMPHASIS`,
* `IMAGE`,
* `LINE_BERAK`,
* `LINK`,
* `TEXT`.

##### `EmphasisLevel` {#api-dom-emphasislevel}

The `EmphasisLevel` enum represents the level of an [`EmphasisContent`](#api-dom-emphasiscontent) object and has the following constants:

* `LEVEL_1`,
* `LEVEL_2`.

##### `HeadingLevel` {#api-dom-headinglevel}

The `HeadingLevel` enum represents the level of an [`HeadingBlock`](#api-dom-headingblock) object and has the following constants:

* `LEVEL_1`,
* `LEVEL_2`,
* `LEVEL_3`,
* `LEVEL_4`,
* `LEVEL_5`,
* `LEVEL_6`.
  
##### `NodeType` {#api-dom-nodetype}

The `NodeType` enum represents the node type of a [`Node`](#api-dom-node) object and has the following constants:

* `DOCUMENT`,
* `BLOCK`,
* `LIST_ITEM`,
* `CONTENT`.


### Handler API

### Combining the Domain Object API and the Handler API

// document is dispatcher, 

// markdom document markdom handler

## Data representations

### JSON

### YAML

### XML