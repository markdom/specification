![](resource/markdom-mark-positive.png)

---

# Markdom

Markdom is lightweight specification for a rich text domain model.

The two main intents of Markdom are to define the lowest common denominator for rich text and to be platform and representation independent. These properties enable Markdom to be used in a wide variety of applications, such as websites, electronic publishing or native rendering on mobile platforms.

The name Markdom is a composition of [**mark**up](https://en.wikipedia.org/wiki/Markup_language) and [**do**main **m**odel](https://en.wikipedia.org/wiki/Domain_model).

## Overview

This specification covers multiple aspects related to Markdom documents:

* The [domain section](#domain) covers the general structure and the supported formatting instructions for Markdom documents.
* The [API section](#api) covers programming interfaces to programmatically create, modify and process Markdom documents.
* The [Data representation section](#data) covers the representation of Markdom documents in common data exchange formats. 
* The [Text representation section](#text) covers the representation of Markdom documents in common markup languages. 

## Why Markdom?

Markdom was created to be an answer to a simple, but as yet unanswered [question](https://stackoverflow.com/questions/34955835/cross-device-rich-text-formatting):

1. Allow an editor to enter rich text in a human friendly way.
2. Transmit the rich text to different platforms, using existing technologies.
3. Display the rich text on different platforms in a native way.

The latter includes at least Android apps (using [Spanned Strings](https://developer.android.com/reference/android/text/Spanned.html)), iOS apps (using [Attributed Strings](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSAttributedString_Class/)) and websites (using HTML).

Markdom tries to fill a gap left open by existing technologies. As the name Markdom indicates, it is closely related to [Markdown](https://daringfireball.net/projects/markdown/).

### Why not HTML?

HTML itself is hardly an editor friendly language, although usable [WYSISYG editors](https://www.tinymce.com/) exist.

The crucial drawback of HTML is, that is is way too powerful for the intended purpose. There are [good reasons](https://leanpub.com/markua/read#leanpub-auto-why-is-inline-html-not-supported-in-markua) why HTML is not suitable, if the intended target platform aren't just websites.

### Why not Markdown?

Markdown is introduced with the following description:

> Markdown is a text-to-HTML conversion tool for web writers. Markdown allows you to write using an easy-to-read, easy-to-write plain text format, then convert it to structurally valid XHTML (or HTML).

The intended limitation to be nothing more than a text-to-HTML conversion tool is recognizable in a multitude of Markdown implementation. Only few implementations offer conversion targets that are not HTML or a domain model that is open to programmatic manipulation.

Since Markdown was only released as an implementation and without a formal specification, different implementations [disagree](http://johnmacfarlane.net/babelmark2/?text=%3E+*+A+quote%0A%3E+Another+quote) over the meaning of edge cases like the following snippet:

```
> * A quote
> Another quote
```

Additionally, many Markdown implementations use custom flavors of the Markdown syntax or offer custom extensions.

As a result, it is [almost](https://uncodin.github.io/bypass/) impossible to find a set of markdown implementations that generate Spanned Strings for Android, Attributed Strings for iOS and HTML for websites, that interpret a given Markdown text identically and generate equal outputs.

### Why not CommonMark?

[CommonMark](http://commonmark.org/) is a specification for Markdown and compatible CommonMark implementations exist for [different languages](https://github.com/jgm/CommonMark/wiki/List-of-CommonMark-Implementations).

While this solves some of the issues of Markdown, CommonMark still allows the integration of arbitrary HTML and available implementations are still only text-to-HTML converters or tend to offer custom extensions.

### Why not Zoidberg?

Zoidberg is awesome, but not a rich text related technology.

### So, why Markdom?

Markdom tries to overcome the mentioned shortcomings of HTML and lightweight markup languages like CommonMark by introducing a standardized representation that can be processed in an unambiguous manner.

![](resource/markdom-overview.png)

A typical application of Markdom may involve the following use cases.

When an editor uses a backend to create content:

* The server allows an editor to enter CommonMark text.
* The server parses the markup with any suitable parser (i.e. with an actual parser, not just a regex-based converter) that is available for the language the server is written in, and creates a Markdom document.
* The server generates a HTML representation of the Markdom document and delivers it to the editor.
* The server generates a XML representation of the Markdom document and stores is for later use.

When the editor returns to review the content:

* The server loads the previously stored XML representation and creates a Markdom document.
* The server creates a CommonMark representation of the Markdom document and delivers it to the editor.

When a visitor uses the frontend to view the content:

* The server loads the previously stored XML representation and creates a Markdom document.
* The server creates a HTML representation of the Markdom document and delivers it to the visitor.

When a visitor uses a smartphone app to view the content:

* The server loads the previously stored XML representation and creates a Markdom document.
* The server creates a JSON representation of the Markdom document and delivers it to the app.
* The app receives the JSON representation and creates a Markdom document.
* The app creates a native representation of the Markdom document and displays it.
  
Other use cases might include to interpret other sources of rich text (e.g. an uploaded HTML or DOC file) to create a Markdom document or generate other representations (e.g. a PDF file) of a Markdom document.
  
The important benefit of Markdom is, that the responsibility to interpret the somewhat cumbersome markup language lies completely by the server. Once the server has interpreted the original input, it is completely unambiguous and easy to process for all participants.

## Domain {#domain}

A Markdom document represents the entirety of a rich text document, including the actual rich text and the formatting instructions. A Markdom document contains of a sequence of Markdom block.

A Markdom block represents a portion of a rich text document that serves a specific purpose. Each type of Markdom block has the necessary properties that describe it's content. Some Markdom blocks, e.g. a Markdom block representing a block quote, contain further Markdom blocks. Other Markdom blocks, e.g. a Markdom block representing a paragraph, contain Markdom content.

A Markdom content represents a portion of rich text that serves a specific purpose. Each type of Markdom block has the necessary properties that describe it's content. Some Markdom contents, e.g. a Markdom block representing a link, contain further Markdom content. A Markdom content never contains a Markdom block.

Put another way, a Markdom document is represented as a tree of Markdom nodes. Starting with a root node that represents an entire rich text document, each Markdom node represents a portion of the corresponding Markdom document.

### Nodes {#domain-node}

A *Markdom node* is either a

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
* *ordered Markdom list block*.

##### Code Block {#domain-codeblock}

A *Markdom code block* is a *Markdom block* that represents a portion of plain text (e.g. source code) that may be augmented with meaningful syntax highlighting. Implementations that generate output that is displayed to humans should display the text as preformatted text in a monospaced font.

A *Markdom code block* has a mandatory string parameter named `code` and an optional string parameter named `hint`. 

Values of the `code` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters other then `LINE_FEED` (`\n`) or `CHARACTER_TABULATION` (`\t`). Values of the `code` parameter shouldn't be empty.

Values of the `hint` parameter, if present, should be the common name of a programming or markup language in lowercase (i.e `java`, `php`, `json`, `html`, ...) and are intended to be used by implementations to augment the preformatted text with meaningful syntax highlighting. Values of the `code` parameter, if present, shouldn't be empty.

##### Division Block {#domain-divisionblock}

A *Markdom division block* is a *Markdom block* that represents a division or thematic break. Implementations that generate output that is displayed to humans should display a horizontal rule.

##### Heading Block {#domain-headingblock}

A *Markdom heading block* is a *Markdom block* that represents a headline. Implementations that generate output that is displayed to humans should display the headline text as an appropriately emphasized and separated paragraph.

A *Markdom heading block* has a mandatory integer parameter named `level` and a sequence of *Markdom contents* named `contents`.

Valid values of the `level` parameter are 1 to 6, where a lower value indicates a higher precedence of the headline.

The `contents` sequence shouldn't be empty.

##### Ordered List Block {#domain-orderedlistblock}

An *ordered Markdom list block* is a *Markdom block* that represents an ordered list (enumeration) of list items. Each list item contains further portions of the Markdom document. Implementations that generate output that is displayed to humans should indent all list items and display the index of each list item in front of it.

An *ordered Markdom list block* has a mandatory integer parameter `startIndex` and a sequence of *Markdom list items* named `items`.

Valid values of the `startIndex` parameter are non-negative integers. The value of the `startIndex` parameter is the index of the first item in the `items` sequence. Following items in the `items` sequence have successive indices.

The `items` sequence shouldn't be empty.

##### Paragraph Block {#domain-paragraphblock}

A *Markdom paragraph block* is a *markdom block* that represents a paragraph. Implementations that generate output that is displayed to humans should display the paragraph text as an appropriately separated paragraph.

A *Markdom paragraph block* has a sequence of *Markdom contents* named `contents`.

The `contents` sequence shouldn't be empty.

##### Quote Block {#domain-quoteblock}

A *Markdom quote block* is a *Markdom block* that represents a quote. The quote contains further portions of the Markdom document. Implementations that generate output that is displayed to humans should emphasize (e.g. a vertical line, indentation, slanted font) the quote appropriately.

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

An *Markdom emphasis content* is a *Markdom content* that represents emphasized text. Implementations that generate output that is displayed to humans should display text appropriately emphasized (e.g. italic or bold).

A *Markdom emphasis content* has a mandatory integer parameter named `level` and a sequence of *Markdom contents* named `contents`.

Valid values of the `level` parameter are 1 to 2, where a higher value indicates a higher importance of the emphasized text.

The `contents` sequence shouldn't be empty.

##### Image Content {#domain-imagecontent}

An *Markdom image content* is a *Markdom content* that represents an image, including a title text and an alternative text. Implementations that generate output that is displayed to humans should display the linked image with the title text or, if the linked image couldn't be resolved, the alternative text as if the *Markdom image content* was a *Markdom text content*.

A *Markdom link content* has a mandatory integer parameter named `uri` and an optional string parameter named `title` and an optional string parameter named `alternative`.  

Valid values of the `uri` parameter are valid [URI references](https://tools.ietf.org/html/rfc3986#section-4.1). Values of the `uri` parameter should link to an image resource (e.g. a JPEG or PNG file). Actual applications that process Markdom documents may impose further constraints.

Values of the `title` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters.

Values of the `alternative` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters. Values of the `alternative` parameter shouldn't be empty.

##### Line Break Content {#domain-linebreakcontent}

A *Markdom line break content* is a *Markdom content* that represents a link. A line break is either a *hard line break* or a *soft line break*. Implementations that generate output that is displayed to humans should display only hard line breaks. Implementations that generate output that is processed as source code, e.g. as Markdown text, should also display soft line breaks in a way that doesn't introduce line breaks during further processing of the generated source code.

A *Markdom line break content* has a mandatory boolean parameter named `hard`.

##### Link Content {#domain-linkcontent}

A *Markdom link content* is a *Markdom content* that represents text with a link. Implementations that generate output that is displayed to humans should display text appropriately emphasized (e.g. underlined or in a different color).

A *Markdom link content* has a mandatory integer parameter named `uri` and a sequence of *Markdom contents* named `contents`.

Valid values of the `uri` parameter are valid [URI references](https://tools.ietf.org/html/rfc3986#section-4.1). Actual applications that process Markdom documents may impose further constraints.

The `contents` sequence shouldn't be empty. A *Markdom content* in the `contents` sequence must not contain another *Markdom link content* or, recursively, contain a *Markdom content* that contains a *Markdom link content*.

##### Text Content {#domain-textcontent}

A *Markdom text content* is a *Markdom content* that represents a portion of text. Implementations that generate output that is displayed to humans should display the text as text in an appropriate font.

A *Markdom text content* has a mandatory string parameter named `text`. 

Values of the `text` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters. Values of the `text` parameter shouldn't be empty.

## Example document {#example}

This CommonMark text describes a rich text document that serves as an example document throughout the rest of this specification:

    # Markdom
    
    1. [Foo](#Bar)
    2. Lorem ipsum\
       `dolor sit amet`
    3. > *Baz*
    
    ```
    goto 11
    ```


The rich text document consists of a heading, an ordered list with three list items and some plain text. The first list item of the ordered list contains a link. the second list item of the unordered list contains emphasized text, a hard line break and some plain text. The third list item of the ordered list contains a quoted text.

The following image shows a tree of *Markdom nodes*, i.e. a Markdom document, that describes the same rich text document:

![](resource/markdom-example.png)

The same Markdom document can be represented as 

* an [object graph](#api-dom-example), 
* a [sequence of events](#api-handler-example), 
* a [JSON document](#data-json-example), 
* a [YAML document](#data-yaml-example), 
* a [XML document](#data-xml-example) or
* a [HTML document](#text-html-example).

## APIs {#api}

This specification describes two distinct APIs:

* The [Domain Model API](#api-dom) contains a set of interfaces that describe how a [*Markdom document*](#domain-document) is represented as an object graph and methods to compose and consume such an object graph.

   The Domain Model API allows to create a representation of a [*Markdom document*](#domain-document) in the memory, which can then be examined, modified and further processed.
   
* The [Handler API](#api-dom) contains a [`Handler`](#api-handler-handler) interface that describes how a Markdom document is represented as a sequence of events.

  The Handler API allows to process a [*Markdom document*](#domain-document) on the fly without the necessity to create an object graph. Events that describe a [*Markdom document*](#domain-document) might be dispatched by an object graph from the Domain Model API or by a specific event dispatcher implementation that process a [data representation](#data) or [text representation](#text) of a [*Markdom document*](#domain-document).
   
This specifications primarily covers the interfaces and enumerations for both APIs. A concrete implementation of this specification for a given programming language should consist of corresponding interface definitions as well as an concrete implementation of the Domain Model API interfaces and some commonly useful concrete implementations of the Handler API. 

Depending on the programming language, it might be sensible to divide the interfaces, enumerations and concrete implementations into multiple packages. 

* A *Common* package that contains the common enumerations and a base exception for all Markdom related operations.
* A *Handler* package that contains the interfaces and commonly useful concrete implementations of the Handler API.
* A *Model* package that depends contains the interfaces and enumerations of the Domain Model API. This allows for different concrete implementations of the Domain Model API.
* A *Model* reference implementation package that contains a concrete implementation of the Domain Model API.
* Several *Handler* implementation packages for different tasks some of which may depend on the *Model* package. 

![](resource/markdom-packages.png)

It is commonly recommended to implement an algorithms that processes a [*Markdom document*](#domain-document) as a [`Handler`](#api-handler-handler) rather than a method that directly processes a [Domain Model API](#api-dom) object graph. This allows to use the algorithm implementation in a multitude of scenarios (e.g. converting the [XML representation](#data-xml) of a [*Markdom document*](#domain-document) into a corresponding [HTML representation](#text-html) as a stream without creating an object graph for XML, Markdom or HTML). The Domain Model API should generally only be used if it is necessary to temporarily store a [*Markdom document*](#domain-document) in the memory or to programmatically modify a [*Markdom document*](#domain-document) before it is further processed with a [`Handler`](#api-handler-handler).
   
### Common {#api-common}

#### Dependencies {#api-common-dependencies}

*This section lists the interfaces, enumerations and classes that are used by the Domain Model API.*

The Domain Model API uses the following classes:

// hint**

// arrays (var arg), iterator and iterable

// source

// sequence

// primitives (int, ), unicode string,

#### Enumerations {#api-common-enumerations}

Markdom APIs have the following enumerations:

##### `BlockType` {#api-common-blocktype}

The `BlockType` enum represents the node type of a [*Markdom block*](#domain-block) and has the following constants:

* `CODE`,
* `DIVISION`,
* `HEADING`,
* `ORDERED_LIST`,
* `PARAGRAPH`,
* `QUOTE`,
* `UNORDERED_LIST`.
  
##### `ContentType` {#api-common-contenttype}  

The `ContentType` enum represents the node type of a [*Markdom content*](#domain-content) and has the following constants:

* `CODE`,
* `EMPHASIS`,
* `IMAGE`,
* `LINE_BERAK`,
* `LINK`,
* `TEXT`.

##### `EmphasisLevel` {#api-common-emphasislevel}

The `EmphasisLevel` enum represents the level of a [*Markdom emphasis content*](#domain-emphasiscontent) object and has the following constants:

* `LEVEL_1`,
* `LEVEL_2`.

##### `HeadingLevel` {#api-common-headinglevel}

The `HeadingLevel` enum represents the level of a [*Markdom heading block*](#domain-headingblock) object and has the following constants:

* `LEVEL_1`,
* `LEVEL_2`,
* `LEVEL_3`,
* `LEVEL_4`,
* `LEVEL_5`,
* `LEVEL_6`.
  
### Domain Model API {#api-dom}

The Domain Model API represents a Markdom document as an object graph.

#### Interfaces {#api-dom-interfaces}

The following image shows the interfaces that are part of the Domain Model API:

![](resource/markdom-nodes.png)

*Interfaces with bold names are eminently suitable to be implemented as final classes, interfaces with italic names are eminently suitable to be implemented as abstract classes and interfaces with normal names are eminently suitable to be implemented as traits or mixins.*

##### `Block` {#api-dom-block}

A [`Block`](#api-dom-block) object is a [`Node`](#api-dom-node) object that represents a [*Markdom block*](#domain-block).

###### Constructors {#api-dom-block-constructor}

An implementation of `Block` should have a constructor with signature `Block()`.

###### `getBlockType` {#api-dom-block-getblocktype}

A [`Block`](#api-dom-block) object must have a method with signature `BlockType getBlockType()`.  

This method must return the `BlockType` value that corresponds to the type of the represented [*Markdom block*](#domain-block).

##### `BlockParent` {#api-dom-blockparent}

A [`BlockParent`](#api-dom-blockparent) object is a [`Node`](#api-dom-node) object that represents a [*Markdom node*](#domain-node) that contains [*Markdom blocks*](#domain-block).

An implementation of `BlockParent` must have a final and initially empty companion `Sequence` of [`Block`](#api-dom-block) objects that is associated with the [`BlockParent`](#api-dom-blockparent) object.

Any structural modification (insert, remove, clear, replace) to the associated `Sequence` of [`Block`](#api-dom-block) objects must reflect the fact, that a [`Block`](#api-dom-block) object that is added to the associated `Sequence` object is attached to the [`BlockParent`](#api-dom-blockparent) object until is is removed from the associated `Sequence` of [`Block`](#api-dom-block) objects.

Attaching a [`Block`](#api-dom-block) object to the [`BlockParent`](#api-dom-blockparent) object  must fail
* if the [`Block`](#api-dom-block) object is not present, or  
* if the [`Block`](#api-dom-block) object is already attached to a [`BlockParent`](#api-dom-blockparent) object, or 
* if attaching the [`Block`](#api-dom-block) object to the [`BlockParent`](#api-dom-blockparent) object would create a [cycle](#api-dom-detecting-cycles) in the tree of Markdom nodes that the [`BlockParent`](#api-dom-blockparent) object is part of.

###### Constructors {#api-dom-blockparent-constructor}

An implementation of `BlockParent` should have a constructor with signature `BlockParent()`.

For convenience, an implementation of `BlockParent` should have a constructor with signature `BlockParent(Block... blocks)` that delegates to `addBlocks(Block... blocks)`.

###### `getBlockParentType` {#api-dom-listblock-getblockparenttype}

A [`BlockParent`](#api-dom-blockparent) object must have a method with signature `BlockParentType getBlockParentType()`.  

This method must return the [`BlockParentType`](#api-dom-blockparenttype) value that corresponds to the type of the [`BlockParent`](#api-dom-blockparent) object.

###### `getBlocks` {#api-dom-blockparent-getblocks}

A [`BlockParent`](#api-dom-blockparent) object must have a method with signature `Sequence getBlocks()`.

This method must return the associated `Sequence` of [`Block`](#api-dom-block) objects.

###### `addBlock` {#api-dom-blockparent-addblock}

For convenience, a [`BlockParent`](#api-dom-blockparent) object should have a method with signature `addBlock(Block block)`.

This method must add `block` at the end of the associated `Sequence` of [`Block`](#api-dom-block) objects. This attaches `block` to the [`BlockParent`](#api-dom-blockparent) object.

This method must fail if add `block` to the  associated `Sequence` of [`Block`](#api-dom-block) objects failed.

###### `addBlocks` {#api-dom-blockparent-addblocks}

For convenience, a [`BlockParent`](#api-dom-blockparent) object should have a method with signature `addBlocks(Block... blocks)`.

This method must add all [`Block`](#api-dom-block) objects from `blocks` in the given order at the end of the `Sequence` of [`Block`](#api-dom-block) objects of the [`BlockParent`](#api-dom-blockparent) object, as if `addBlock(Block block)` has been called repeatedly for all [`Block`](#api-dom-block) objects from `blocks`.  This attaches all [`Block`](#api-dom-block) objects from `blocks` to the [`BlockParent`](#api-dom-blockparent) object.
  
This method must fail if `blocks` is not present.
This method must fail if adding any [`Block`](#api-dom-block) object from `blocks` to the  associated `Sequence` of [`Block`](#api-dom-block) objects failed.
  
Because this method is a short hand for repeated calls to `addBlock(Block block)`, it must add all prior [`Block`](#api-dom-block) objects from `blocks` to the  associated `Sequence` of [`Block`](#api-dom-block) objects, if it fails because of a violating [`Block`](#api-dom-block) object from `blocks`.

##### `CodeBlock` {#api-dom-codeblock}

A [`CodeBlock`](#api-dom-codeblock) objects is a [`Block`](#api-dom-block) object that represents a [*Markdom code block*](#domain-codeblock).

###### Constructors {#api-dom-codeblock-constructor}

An implementation of `CodeBlock` should have a constructor with signature `CodeBlock()` that set the code of the [`CodeBlock`](#api-dom-codeblock) object to the empty string ant the hint of the `Code` object to be not present.

For convenience, an implementation of `CodeBlock` should have a constructor with signature `CodeBlock(String code)` that delegates to `setCode(String code)`.

For convenience, an implementation of `CodeBlock` should have a constructor with signature `CodeBlock(String code, String hint)` that delegates to `setCode(String code)` and `setHint(String hint)`.

###### `getCode` {#api-dom-codeblock-getcode}

A [`CodeBlock`](#api-dom-codeblock) object must have a method with signature `String getCode()`.

The method must return the `code` parameter of the represented [*Markdom code block*](#domain-codeblock).

###### `setCode` {#api-dom-codeblock-setcode}

A [`CodeBlock`](#api-dom-codeblock) object must have a method with signature `setCode(String code)`.

This method must set the `code` parameter of the represented [*Markdom code block*](#domain-codeblock). 
  
This method must fail if `code` is not present.

###### `getHint` {#api-dom-codeblock-gethint}

A [`CodeBlock`](#api-dom-codeblock) object must have a method with signature `String? getHint()`.

The method must return the optional `hint` parameter of the represented [*Markdom code block*](#domain-codeblock).

###### `setHint` {#api-dom-codeblock-sethint}
  
A [`CodeBlock`](#api-dom-codeblock) object must have a method with signature `setHint(String? hint)`.

This method must set the `hint` parameter of the represented [*Markdom code block*](#domain-codeblock). 
 	
##### `CodeContent` {#api-dom-codecontent}

A [`CodeContent`](#api-dom-codecontent) object is a [`Content`](#api-dom-content) object that represents a [*Markdom code content*](#domain-codecontent).

###### Constructors {#api-dom-codecontent-constructors}

An implementation of `CodeContent` should have a constructor with signature `CodeContent()` that set the code of the [`CodeContent`](#api-dom-codecontent) object to the empty string.

For convenience, an implementation of `CodeContent` should have a constructor with signature `CodeContent(String code)` that delegates to `setCode(String code)`.

###### `getCode` {#api-dom-codecontent-getcode}

A [`CodeContent`](#api-dom-codecontent) object must have a method with signature `String getCode()`.

The method must return the `code` parameter of the represented [*Markdom code content*](#domain-codecontent).

###### `setCode` {#api-dom-codecontent-setcode}

A [`CodeContent`](#api-dom-codecontent) object must have a method with signature `setCode(String code)`.

This method must set the `code` parameter of the represented [*Markdom code content*](#domain-codecontent). 
  
This method must fail if `code` is not present.

##### `Content` {#api-dom-content}

A Content` object is a [`Node`](#api-dom-node) object that represents a [*Markdom content*](#domain-content).

###### Constructors {#api-dom-content-constructor} 

An implementation of `Content` should have a constructor with signature `Content()`.

###### `getContentType` {#api-dom-content-getcontenttype}

A [`Content`](#api-dom-content) object must have a method with signature `ContentType getContentType()`.  

This method must return the `ContentType` value that corresponds to the type of the represented [*Markdom content*](#domain-content).

###### `getBlock` {#api-dom-content-getblock}

A [`Content`](#api-dom-content) object must have a method with signature `Block getBlock()`.

This method must return the nearest [`Block`](#api-dom-block) object object the `Content` in the tree of Markdom nodes  the [`Content`](#api-dom-content) object is part of, which is the [`Block`](#api-dom-block) object of the `ContentParent` this [`Content`](#api-dom-content) object is currently attached to.

This method must fail if the [`Content`](#api-dom-content) object doesn't [have](#api-dom-node-hasparent) a [parent](#api-dom-node-getparent) [`ContentParent`](#api-dom-node) object.

##### `ContentParent` {#api-dom-contentparent}

A [`ContentParent`](#api-dom-contentparent) object is a [`Node`](#api-dom-node) object that represents a [*Markdom node*](#domain-node) that contains [*Markdom contents*](#domain-content).

An implementation of `ContentParent` must have a final and initially empty companion `Sequence` of [`Content`](#api-dom-content) objects that is associated with the [`ContentParent`](#api-dom-contentparent) object.

Any structural modification (insert, remove, clear, replace) to the associated `Sequence` of [`Content`](#api-dom-content) objects must reflect the fact, that a [`Content`](#api-dom-content) object that is added to the associated `Sequence` object is attached to the [`ContentParent`](#api-dom-contentparent) object until is is removed from the associated `Sequence` of [`Content`](#api-dom-content) objects.

Attaching a [`Content`](#api-dom-content) object to the [`ContentParent`](#api-dom-contentparent) object  must fail
* if the [`Content`](#api-dom-content) object is not present, or  
* if the [`Content`](#api-dom-content) object is already attached to a [`ContentParent`](#api-dom-contentparent) object, or 
* if attaching the [`Content`](#api-dom-content) object to the [`ContentParent`](#api-dom-contentparent) object would create a [cycle](#api-dom-detecting-cycles) in the tree of Markdom nodes that the [`ContentParent`](#api-dom-contentparent) object is part of.

###### Constructors {#api-dom-contentparent-constructor}

An implementation of `ContentParent` should have a constructor with signature `ContentParent()`.

For convenience, an implementation of `ContentParent` should have a constructor with signature `ContentParent(Content... contents)` that delegates to `addContents(Content... contents)`.

###### `getContentParentType` {#api-dom-listblock-getcontentparenttype}

A [`ContentParent`](#api-dom-contentparent) object must have a method with signature `ContentParentType getContentParentType()`.  

This method must return the [`ContentParentType`](#api-dom-contentparenttype) value that corresponds to the type of the [`ContentParent`](#api-dom-contentparent) object.

###### `getContents` {#api-dom-contentparent-getcontents}

A [`ContentParent`](#api-dom-contentparent) object must have a method with signature `Sequence getContents()`.

This method must return the associated `Sequence` of [`Content`](#api-dom-content) objects.

###### `addContent` {#api-dom-contentparent-addcontent}

For convenience, a [`ContentParent`](#api-dom-contentparent) object should have a method with signature `addContent(Content content)`.

This method must add `content` at the end of the associated `Sequence` of [`Content`](#api-dom-content) objects. This attaches `content` to the [`ContentParent`](#api-dom-contentparent) object.

This method must fail if add `content` to the  associated `Sequence` of [`Content`](#api-dom-content) objects failed.

###### `addContents` {#api-dom-contentparent-addcontents}

For convenience, a [`ContentParent`](#api-dom-contentparent) object should have a method with signature `addContents(Content... contents)`.

This method must add all [`Content`](#api-dom-content) objects from `contents` in the given order at the end of the `Sequence` of [`Content`](#api-dom-content) objects of the [`ContentParent`](#api-dom-contentparent) object, as if `addContent(Content content)` has been called repeatedly for all [`Content`](#api-dom-content) objects from `contents`.  This attaches all [`Content`](#api-dom-content) objects from `contents` to the [`ContentParent`](#api-dom-contentparent) object.
  
This method must fail if `contents` is not present.
This method must fail if adding any [`Content`](#api-dom-content) object from `contents` to the  associated `Sequence` of [`Content`](#api-dom-content) objects failed.
  
Because this method is a short hand for repeated calls to `addContent(Content content)`, it must add all prior [`Content`](#api-dom-content) objects from `contents` to the  associated `Sequence` of [`Content`](#api-dom-content) objects, if it fails because of a violating [`Content`](#api-dom-content) object from `contents`.
   
##### `ContentParentBlock` {#api-dom-contentparentblock}

A [`ContentParentBlock`](#api-dom-contentparentblock) object is a [`Block`](#api-dom-block) object and a [`ContentParent`](#api-dom-contentparent) object that represents a [*Markdom block*](#domain-block) that contains [*Markdom content*](#domain-content).

###### Constructors {#api-dom-contentparentblock-constructor}

An implementation of `ContentParentBlock` should have a constructor with signature `ContentParentBlock()`.

For convenience, an implementation of `ContentParentBlock` should have a constructor with signature `ContentParentBlock(Content... contents)` that delegates to `ContentParent#addContents(Content... content)`.

##### `ContentParentContent` {#api-dom-contentparentcontent}

A [`ContentParentContent`](#api-dom-contentparentcontent) object is a [`Content`](#api-dom-content) object and a [`ContentParent`](#api-dom-contentparent) object that represents a [*Markdom content*](#domain-content) that contains [*Markdom content*](#domain-content).

###### Constructors {#api-dom-contentparentcontent-constructor}

An implementation of `ContentParent` should have a constructor with signature `ContentParentContent()`.

For convenience, an implementation of `ContentParentContent` should have a constructor with signature `ContentParentContent(Content... contents)` that delegates to `ContentParent#addContents(Content... content)`.
  
##### `DivisionBlock` {#api-dom-divisionblock}

A [`DivisionBlock`](#api-dom-divisionblock) object is a [`Block`](#api-dom-block) object that represents a [*Markdom division block*](#domain-divisionblock)

###### Constructors {#api-dom-divisionblock-constructors}

An implementation of `DivisionBlock` should have a constructor with signature `DivisionBlock()` 
  
##### `Document` {#api-dom-document}

A [`Document`](#api-dom-document) object is a [`BlockParent`](#api-dom-blockparent) object and a [`Dispatcher`](#api-handler-dispatcher) object that represents a [*Markdom document*](#domain-document).

###### Constructors {#api-dom-document-constructor}

For convenience, an implementation of `Document` should have a constructor with signature `Document(Block... blocks)` that delegates to `BlockParent#addBlocks(Block... blocks)`.

##### `EmphasisContent` {#api-dom-emphasiscontent}

An [`EmphasisContent`](#api-dom-emphasiscontent) object is a [`ContentParentContent`](#api-dom-contentparentcontent) object that represents a [*Markdom emphasis content*](#domain-emphasiscontent).  

###### Constructors {#api-dom-emphasiscontent-constructor}

An implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent()` that sets the level of the [`EmphasisContent`](#api-dom-emphasiscontent) object to `LEVEL_1`.

For convenience, an implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent(EmphasisLevel level)` that delegates to `setLevel(EmphasisLevel level)`.

For convenience, an implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent(EmphasisLevel level, Content... contents)` that delegates to `setLevel(EmphasisLevel level)` and `ContentParent#addContents(Content... contents)`.

###### `getLevel` {#api-dom-emphasiscontent-getlevel}

An [`EmphasisContent`](#api-dom-emphasiscontent) object must have a method with signature `EmphasisLevel getLevel()`.

This method must return the `level` parameter of the represented [*Markdom emphasis content*](#domain-emphasiscontent).

###### `setLevel` {#api-dom-emphasiscontent-setlevel}

An [`EmphasisContent`](#api-dom-emphasiscontent) object must have a method with signature `setLevel(EmphasisLevel level)`.

This method must set the `level` parameter of the represented [*Markdom emphasis content*](#domain-emphasiscontent). 
  
This method must fail if `level` is not present.
   
##### `HeadingBlock` {#api-dom-headingblock}

A [`HeadingBlock`](#api-dom-headingblock) object is a [`ContentParentBlock`](#api-dom-contentparentblock) object that represents a [*Markdom heading blocks*](#domain-headingblock).  

###### Constructors {#api-dom-headingblock-constructor}

An implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock()` that sets the level of the [`HeadingBlock`](#api-dom-headingblock) object to `LEVEL_1`.

For convenience, an implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock(HeadingLevel level)` that delegates to `setLevel(HeadingLevel level)`.

For convenience, an implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock(HeadingLevel level, Content... contents)` that delegates to `setLevel(HeadingLevel level)` and `ContentParent#addContents(Contents... contents)`.

###### `getLevel` {#api-dom-headingblock-getlevel}

A [`HeadingBlock`](#api-dom-headingblock) object must have a method with signature `HeadingLevel getLevel()`.

This method must return the `level` parameter of the represented [*Markdom heading blocks*](#domain-headingblock).

###### `setLevel` {#api-dom-headingblock-setlevel}

A [`HeadingBlock`](#api-dom-headingblock) object must have a method with signature `setLevel(HeadingLevel level)`.

This method must set the `level` parameter of the represented [*Markdom heading blocks*](#domain-headingblock). 
  
This method must fail if `level` is not present.
   
##### `ImageContent` {#api-dom-imagecontent}

An [`ImageContent`](#api-dom-imagecontent) object is a [`Content`](#api-dom-content) object that represents a [*Markdom image content*](#domain-imagecontent).  

###### Constructors {#api-dom-imagecontent-constructor}

An implementation of `LinkContent` should have a constructor with signature `LinkContent()` that sets the uri of the [`ImageContent`](#api-dom-imagecontent) object to the empty string and the title of the [`ImageContent`](#api-dom-imagecontent) object to be not present and the alternative text of the [`ImageContent`](#api-dom-imagecontent) object to be not present.

For convenience, an implementation of `ImageContent` should have a constructor with signature `ImageContent(String uri)` that delegates to `setUri(String uri)`.

For convenience, an implementation of `ImageContent` should have a constructor with signature `ImageContent(String uri, String? title, String? alternative)` that delegates to `setUri(String uri)` and `setTitle(String? title)` and `setAlternative(String? alternative)`.

###### `getUri` {#api-dom-imagecontent-geturi}

A [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `String getUri()`.

This method must return the `uri` parameter of the represented [*Markdom image content*](#domain-imagecontent).

###### `setUri` {#api-dom-imagecontent-seturi}

An [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `setUri(String uri)`.

This method must set the `uri` parameter of the represented [*Markdom image content*](#domain-imagecontent).
  
This method must fail if `uri` is not present.

This method must fail if `uri` is not a valid URI reference.

###### `getTitle` {#api-dom-imagecontent-gettitle}

A [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `String getTitle()`.

This method must return the `title` parameter of the represented [*Markdom image content*](#domain-imagecontent).

###### `setTitle`{#api-dom-imagecontent-settitle}
  
An [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `setTitle(String? title)`.

This method must set the `title` parameter of the represented [*Markdom image content*](#domain-imagecontent).

###### `getAlternative` {#api-dom-imagecontent-getalternative}

A [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `String getAlternative()`.

This method must return the `alternative` parameter of the represented [*Markdom image content*](#domain-imagecontent).

###### `setAlternative` {#api-dom-imagecontent-setalternative}

An [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `setAlternative(String? alternative)`.

This method must set the `alternative` parameter of the represented [*Markdom image content*](#domain-imagecontent).

##### `LineBreakContent` {#api-dom-linebreakcontent}

A [`LineBreakContent`](#api-dom-linebreakcontent) object is a [`Content`](#api-dom-content) object that represents a [*Markdom line break content*](#domain-linebreakcontent).

###### Constructors {#api-dom-linebreakcontent-constructor}

An implementation of `LineBreakContent` should have a constructor with signature `LineBreakContent()` that sets the [`LineBreakContent`](#api-dom-linebreakcontent) object to represent a soft line break.

For convenience, an implementation of `LineBreakContent` should have a constructor with signature `LinkContent(Boolean hard)` that delegates to `setHard(Boolean hard)`.

###### `isHard` {#api-dom-linebreakcontent-ishard}

A [`LineBreakContent`](#api-dom-linebreakcontent) object must have a method with signature `Boolean isHard()`.

This method must return the `hard` parameter of the represented [*Markdom line break content*](#domain-linebreakcontent).

###### `setHard` {#api-dom-linebreakcontent-sethard}

A [`LineBreakContent`](#api-dom-linebreakcontent) object must have a method with signature `setHard(Boolean hard)`.

This method must set the `hard` parameter of the represented [*Markdom line break content*](#domain-linebreakcontent).
  
This method must fail if `hard` is not present.
   
##### `LinkContent` {#api-dom-linkcontent}

A [`LinkContent`](#api-dom-linkcontent) object is a [`ContentParentContent`](#api-dom-contentparentcontent) object that represents a [*Markdom link content*](#domain-linkcontent).

###### Constructors {#api-dom-linkcontent-constructor}

An implementation of `LinkContent` should have a constructor with signature `LinkContent()` that sets the uri of the [`LinkContent`](#api-dom-linkcontent) object to the empty string.

For convenience, an implementation of `LinkContent` should have a constructor with signature `LinkContent(String uri)` that delegates to `setUri(String uri)`.

For convenience, an implementation of `LinkContent` should have a constructor with signature `LinkContent(String uri, Content... contents)` that delegates to `setUri(String uri)` and `ContentParent#addContents(Content... contents)`.

###### `getUri` {#api-dom-imagecontent-geturi}

A [`LinkContent`](#api-dom-linkcontent) object must have a method with signature `String getUri()`.

This method must return the `uri` parameter of the represented [*Markdom link content*](#domain-linkcontent).

###### `setUri` {#api-dom-linkcontent-seturi}

A [`LinkContent`](#api-dom-linkcontent) object must have a method with signature `setUri(String uri)`.

This method must set the `uri` parameter of the represented [*Markdom link content*](#domain-linkcontent).
  
This method must fail if `uri` is not present.

This method must fail if `uri` is not a valid URI reference.
   
##### `ListBlock` {#api-dom-listblock}

A [`ListBlock`](#api-dom-listblock) object is a [`Block`](#api-dom-block) object that represents a [*Markdom list block*](#domain-listblock)  

An implementation of `ListBlock` must have a final and initially empty companion `Sequence` of [`ListItem`](#api-dom-listitem) objects that is associated with the [`ListBlock`](#api-dom-listblock) object.

Any structural modification (insert, remove, clear, replace) to the associated `Sequence` of [`ListItem`](#api-dom-listitem) objects must reflect the fact, that a [`ListItem`](#api-dom-listitem) object that is added to the associated `Sequence` object is attached to the [`ListBlock`](#api-dom-listblock) object until is is removed from the associated `Sequence` of [`ListItem`](#api-dom-listitem) objects.

Attaching a [`ListItem`](#api-dom-listitem) object to the [`ListBlock`](#api-dom-listblock) object  must fail
* if the [`ListItem`](#api-dom-listitem) object is not present, or  
* if the [`ListItem`](#api-dom-listitem) object is already attached to a [`ListBlock`](#api-dom-listblock) object, or 
* if attaching the [`ListItem`](#api-dom-listitem) object to the [`ListBlock`](#api-dom-listblock) object would create a [cycle](#api-dom-detecting-cycles) in the tree of Markdom nodes that the [`ListBlock`](#api-dom-listblock) object is part of.

###### Constructors {#api-dom-listblock-constructor}

An implementation of `ListBlock` should have a constructor with signature `ListBlock()`.

For convenience, an implementation of `ListBlock` should have a constructor with signature `ListBlock(ListItem... items)` that delegates to `addItems(ListItem... items)`.

###### `getListBlockType` {#api-dom-listblock-getlistblocktype}

A [`ListBlock`](#api-dom-listblock) object must have a method with signature `ListBlockType getListBlockType()`.  

This method must return the [`ListBlockType`](#api-dom-listblocktype) value that corresponds to the type of the [`ListBlock`](#api-dom-listblock) object.

###### `getListItems` {#api-dom-listblock-getitems}

A [`ListItem`](#api-dom-listitem) object must have a method with signature `Sequence getListItems()`.

This method must return the associated `Sequence` of [`ListItem`](#api-dom-listitem) objects.

###### `addItem` {#api-dom-listblock-additem}

For convenience, a [`ListBlock`](#api-dom-listblock) object should have a method with signature `addItem(ListItem item)`.

This method must add `item` at the end of the associated `Sequence` of [`ListItem`](#api-dom-listitem) objects. This attaches `item` to the [`ListBlock`](#api-dom-listblock) object.

This method must fail if add `item` to the  associated `Sequence` of [`ListItem`](#api-dom-listitem) objects failed.

###### `addItems` {#api-dom-listblock-additems}

For convenience, a [`ListBlock`](#api-dom-listblock) object should have a method with signature `addItems(ListItem... items)`.

This method must add all [`ListItem`](#api-dom-listitem) objects from `items` in the given order at the end of the `Sequence` of [`ListItem`](#api-dom-listitem) objects of the [`ListBlock`](#api-dom-listblock) object, as if `addItem(ListItem item)` has been called repeatedly for all [`ListItem`](#api-dom-listitem) objects from `items`.  This attaches all [`ListItem`](#api-dom-listitem) objects from `items` to the [`ListBlock`](#api-dom-listblock) object.
  
This method must fail if `items` is not present.
This method must fail if adding any [`ListItem`](#api-dom-listitem) object from `items` to the  associated `Sequence` of [`ListItem`](#api-dom-listitem) objects failed.
  
Because this method is a short hand for repeated calls to `addItem(ListItem item)`, it must add all prior [`ListItem`](#api-dom-listitem) objects from `items` to the  associated `Sequence` of [`ListItem`](#api-dom-listitem) objects, if it fails because of a violating [`ListItem`](#api-dom-listitem) object from `items`.

##### `ListItem` {#api-dom-listitem}

A [`ListItem`](#api-dom-listitem) object is a [`BlockParent`](#api-dom-blockparent) object that represents a [*Markdom list item*](#domain-listitem).

###### Constructors {#api-dom-listitem-constructor}

An implementation of `ListItem` should have a constructor with signature `ListItem()`.

For convenience, an implementation of `ListItem` should have a constructor with signature `ListItem(Block... blocks)` that delegates to `BlockParent#addBlocks(Block... blocks)`.

##### `Node` {#api-dom-node}

A [`Node`](#api-dom-node) object represents a [*Markdom node*](#domain-node).

###### `getNodeType` {#api-dom-node-getnodetype}

A [`Node`](#api-dom-node) object must have a method with signature `NodeType getNodeType()`.  

This method must return the [`NodeType`](#api-dom-nodetype) value that corresponds to the type of the [`Node`](#api-dom-node) object.

###### `hasParent` {#api-dom-node-hasparent}

A [`Node`](#api-dom-node) object must have a method with signature `Boolean hasParent()`.

This method must return whether the `Node` has a parent [`Node`](#api-dom-node) object.  

Specifically, this method must return `true`
* if the [`Node`](#api-dom-node) object is a [`Block`](#api-dom-block) object and currently attached to a [`BlockParent`](#api-dom-blockparent) object, or
* if the [`Node`](#api-dom-node) object is a [`ListItem`](#api-dom-listitem) object and currently attached to a [`ListBlock`](#api-dom-listblock) object, or
* if the [`Node`](#api-dom-node) object is a [`Content`](#api-dom-content) object and currently attached to a [`ContentParent`](#api-dom-contentparent) object.

Specifically, this method must return `false`
* if the [`Node`](#api-dom-node) object is a `Document`, or
* if the [`Node`](#api-dom-node) object is a [`Block`](#api-dom-block) object and currently not attached to a [`BlockParent`](#api-dom-blockparent) object, or
* if the [`Node`](#api-dom-node) object is a [`ListItem`](#api-dom-listitem) object and currently not attached to a [`ListBlock`](#api-dom-listblock) object, or
* if the [`Node`](#api-dom-node) object is a [`Content`](#api-dom-content) object and currently not attached to a [`ContentParent`](#api-dom-contentparent) object.

###### `getParent` {#api-dom-node-getparent}

A [`Node`](#api-dom-node) object must have a method with signature `Node getParent()`.

This method must return the parent [`Node`](#api-dom-node) object of the [`Node`](#api-dom-node) object. 

Specifically, this method must return 
* the [`BlockParent`](#api-dom-blockparent) object it is currently attached to, if the [`Node`](#api-dom-node) object is a [`Block`](#api-dom-block) object, or
* the [`ListBlock`](#api-dom-listblock) object it is currently attached to, if the [`Node`](#api-dom-node) object is a [`ListItem`](#api-dom-listitem) object, or
* the [`ContentParent`](#api-dom-contentparent) object it is currently attached, if the [`Node`](#api-dom-node) object is a [`Content`](#api-dom-content) object.

This method must fail if the [`Node`](#api-dom-node) object doesn't [have](#api-dom-node-hasparent) a [parent](#api-dom-node-getparent) [`Node`](#api-dom-node) object.

###### `getDocument` {#api-dom-node-getdocument}

A [`Node`](#api-dom-node) object must have a method with signature `Document getDocument()`.

This method must return the [`Document`](#api-dom-document) object that is the root of the tree of [Markdom nodes](#domain-node) the [`Node`](#api-dom-node) object is part of.

Specifically, this method must return 
* the [`Node`](#api-dom-node) object, if the [`Node`](#api-dom-node) object is a [`Document`](#api-dom-document) object, or
* the [`Document`](#api-dom-document) object of the parent [`Node`](#api-dom-node) object, if the [`Node`](#api-dom-node) object has a parent [`Node`](#api-dom-node) object.
  
This method must fail if the [`Node`](#api-dom-node) object doesn't [have](#api-dom-node-hasparent) a [parent](#api-dom-node-getparent) [`Node`](#api-dom-node) object.

###### `getIndex` {#api-dom-node-getindex}

A [`Node`](#api-dom-node) object must have a method with signature `Integer getIndex()`.

This method must return the index of the [`Node`](#api-dom-node) object in the `Source` of child [`Node`](#api-dom-node) object of the parent `Node`.

Specifically, this method must return 
* the index in the attached `Sequence` of [`Block`](#api-dom-block) objects of the [`BlockParent`](#api-dom-blockparent) object it is currently attached to, if the [`Node`](#api-dom-node) object is a [`Block`](#api-dom-block) object, or
* the index in the attached `Sequence` of [`ListItem`](#api-dom-listitem) objects of the [`ListBlock`](#api-dom-listblock) object it is currently attached to, if the [`Node`](#api-dom-node) object is a [`ListItem`](#api-dom-listitem) object, or
* the index in the attached `Sequence` of [`Content`](#api-dom-content) objects of the [`ContentParent`](#api-dom-contentparent) object it is currently attached to, if the [`Node`](#api-dom-node) object is a [`Content`](#api-dom-content) object, or

This method must fail if the [`Node`](#api-dom-node) object doesn't [have](#api-dom-node-hasparent) a [parent](#api-dom-node-getparent) [`Node`](#api-dom-node) object.

###### `hasChildren`

A [`Node`](#api-dom-node) object must have a method with signature `Integer getIndex()`.

This method must return whether the [`Node`](#api-dom-node) object has child [`Node`](#api-dom-node) objects.

Specifically, this method must return `true`
* if the [`Node`](#api-dom-node) object is a [`BlockParent`](#api-dom-blockparent) object and currently has a [`Block`](#api-dom-block) object attached to it, or
* if the [`Node`](#api-dom-node) object is a [`ListBlock`](#api-dom-listblock) object and currently has a [`ListItem`](#api-dom-listitem) object attached to it, or
* if the [`Node`](#api-dom-node) object is a [`ContentParent`](#api-dom-contentparent) object and currently has a [`Content`](#api-dom-content) object attached to it.

Specifically, this method must return `false`
* if the [`Node`](#api-dom-node) object is a [`BlockParent`](#api-dom-blockparent) object and currently has no [`Block`](#api-dom-block) object attached to it, or
* if the [`Node`](#api-dom-node) object is a [`ListBlock`](#api-dom-listblock) object and currently has no [`ListItem`](#api-dom-listitem) object attached to it, or
* if the [`Node`](#api-dom-node) object is a [`ContentParent`](#api-dom-contentparent) object and currently has no [`Content`](#api-dom-content) object attached to it, or
* if the [`Node`](#api-dom-node) object is a [`CodeBlock`](#api-dom-codeblock) object or a [`CodeContent`](#api-dom-codecontent) object or a [`DivisionBlock`](#api-dom-divisionblock) object or an [`ImageContent`](#api-dom-imagecontent) object or a [`LineBreakContent`](#api-dom-linebreakcontent) object or a [`TextContent`](#api-dom-textcontent) object.

###### `getChildren`

A [`Node`](#api-dom-node) object must have a method with signature `Source getChildren()`.

This method must return a `Source` object that yields the child [`Node`](#api-dom-node) objects of the [`Node`](#api-dom-node) object.

Specifically, this method must return `true`
* the attached `Sequence` of [`Block`](#api-dom-block) objects, if the [`Node`](#api-dom-node) object is a [`BlockParent`](#api-dom-blockparent) object, or
* the attached `Sequence` of [`ListItem`](#api-dom-listitem) objects, if the [`Node`](#api-dom-node) object is a [`ListBlock`](#api-dom-listblock) object, or
* the attached `Sequence` of [`Content`](#api-dom-content) objects, if the [`Node`](#api-dom-node) object is a [`ContentParent`](#api-dom-contentparent) object, or
* an emty `Source` object, if the [`Node`](#api-dom-node) object is a [`CodeBlock`](#api-dom-codeblock) object or a [`CodeContent`](#api-dom-codecontent) object or a [`DivisionBlock`](#api-dom-divisionblock) object or an [`ImageContent`](#api-dom-imagecontent) object or a [`LineBreakContent`](#api-dom-linebreakcontent) object or a [`TextContent`](#api-dom-textcontent) object.

##### `OrderedListBlock` {#api-dom-orderedlistblock}

An [`OrderedListBlock`](#api-dom-orderedlistblock) object is a [`ListBlock`](#api-dom-listblock) object that represents an [*ordered Markdom list block*](#domain-orderedlistblock).

###### Constructors {#api-dom-orderedlistblock-constructor}

An implementation of `OrderedListBlock` should have a constructor with signature `OrderedListBlock()` that sets the start index of the [`OrderedListBlock`](#api-dom-orderedlistblock) object to `1`.

For convenience, an implementation of `OrderedListBlock` should have a constructor with signature `OrderedListBlock(Integer startIndex)` that delegates to `setStartIndex(Integer startIndex)`.

For convenience, an implementation of `OrderedListBlock` should have a constructor with signature `OrderedListBlock(Integer startIndex, ListItem... items)` that delegates to `setStartIndex(Integer startIndex)` and `ContentParent#addContents(Content... contents)`.

###### getStartIndex {#api-dom-orderedlistblock-getstartindex}

An [`OrderedListBlock`](#api-dom-orderedlistblock) object must have a method with signature `Integer getStratIndex()`.

This method must return the `startIndex` parameter of the represented [*ordered Markdom list block*](#domain-orderedlistblock).

###### setStartIndex {#api-dom-orderedlistblock-setstartindex}

An [`OrderedListBlock`](#api-dom-orderedlistblock) object must have a method with signature `setStartIndex(Integer startIndex)`.

This method must set the `startIndex` parameter of the represented [*ordered Markdom list block*](#domain-orderedlistblock).
  
This method must fail if `startIndex` is not present.  
This method must fail if `startIndex` is negative.
  
##### `ParagraphBlock` {#api-dom-paragraphblock}

A [`ParagraphBlock`](#api-dom-paragraphblock) object is a [`ContentParentBlock`](#api-dom-contentparentblock) object that represents a [*Markdom paragraph block*](#domain-paragraphblock).

###### Constructors {#api-dom-paragraphblock-constructor}

An implementation of `ParagraphBlock` should have a constructor with signature `ParagraphBlock()`.

For convenience, an implementation of `ParagraphBlock` should have a constructor with signature `ParagraphBlock(Content... contents)` that delegates to `ContentParentBlock#addContents(Contents... contents)`.

##### `QuoteBlock` {#api-dom-quoteblock}

A [`QuoteBlock`](#api-dom-quoteblock) object is a [`Block`](#api-dom-block) object and a [`BlockParent`](#api-dom-blockparent) object that represents a [*Markdom quote block*](#domain-quoteblock).

###### Constructors {#api-dom-quoteblock-constructor}

An implementation of `QuoteBlock` should have a constructor with signature `QuoteBlock()`.

For convenience, an implementation of `QuoteBlock` should have a constructor with signature `QuoteBlock(Block... blocks)` that delegates to `BlockParent#addBlocks(Block... blocks)`.
    
##### `TextContent` {#api-dom-textcontent}

A [`TextContent`](#api-dom-textcontent) object is [`ContentParentContent`](#api-dom-contentparentcontent) object that represents a [*Markdom text content*](#domain-textcontent).

###### Constructors {#api-dom-textcontent-constructor}

An implementation of `TextContent` should have a constructor with signature `TextContent()` that sets the text of the [`TextContent`](#api-dom-textcontent) object to the empty string.
   
For convenience, an implementation of `TextContent` should have a constructor with signature `TextContent(String text)` that delegates to `setText(String text)`.

###### `getText` {#api-dom-textcontent-gettext}

A [`TextContent`](#api-dom-textcontent) object must have a method with signature `String getText()`.

This method must return the `text` parameter of the represented [*Markdom text content*](#domain-textcontent).

###### `setText` {#api-dom-textcontent-settext}

A [`TextContent`](#api-dom-textcontent) object must have a method with signature `setText(String text)`.

This method must set the `text` parameter of the represented [*Markdom text content*](#domain-textcontent).
  
This method must fail if `text` is not present.
   
##### `UnorderedListBlock` {#api-dom-unorderedlistblock}

An [`UnorderedListBlock`](#api-dom-unorderedlistblock) object is a [`ListBlock`](#api-dom-listblock) object that represents an [*unordered Markdom list block*](#domain-unorderedlistblock).

###### Constructors {#api-dom-unorderedlistblock-constructor}

An implementation of `UnorderedListBlock` should have a constructor with signature `UnorderedListBlock()`.
   
For convenience, an implementation of `UnorderedListBlock` should have a constructor with signature `UnorderedListBlock(ListItem... items)` that delegates to  `ListBlock#addItems(ListItem... items)`.

#### Enumerations {#api-enumerations}

The Domain Model API has the following enumerations:

##### `BlockParentType` {#api-dom-blockparenttype}  

The `BlockParentType` enum represents the type of a [`BlockParent`](#api-dom-blockparent) and has the following constants:

* `DOCUMENT`,
* `QUOTE_BLOCK`, 
* `LIST_ITEM`.

##### `ContentParentType` {#api-dom-contentparenttype}  

The `ContentParentType` enum represents the type of a [`ContentParent`](#api-dom-blockparent) and has the following constants:

* `HEADING_BLOCK`,
* `PARAGRAPH_BLOCK`, 
* `EMPHASIS_CONTENT`,
* `LINK_CONTENT`.

##### `NodeType` {#api-dom-nodetype}

The `NodeType` enum represents the type of a [`Node`](#api-dom-node) object and has the following constants:

* `DOCUMENT`,
* `BLOCK`,
* `LIST_ITEM`,
* `CONTENT`.

##### `ListBlockType` {#api-dom-listblocktype}  

The `ListBlockType` enum represents the type of a [`ListBlock`](#api-dom-listblock) and has the following constants:

* `ORDERED_LIST_BLOCK`,
* `UNORDERED_LIST_BLOCK`.

#### Example Document {#api-dom-example}  

The following Domain Model API object graph represents the [example document](#example):

![](resource/markdom-objectgraph.png)

Every `BlockParent`, `ListBlock` or `ContentParent` object has a reference to its companion `Sequence` object. A companion `Sequence` object holds references to the [children](#api-dom-node-getchildren) of the corresponding `BlockParent`, `ListBlock` or `ContentParent` object. Each child has a reference to its [parent](#api-dom-node-getparent).

#### Additional information

##### Up navigation

The following image shows the possible methods to navigate from an `Node` object upwards from the leaf `TextContent` object with `text` value `Baz` of the [example document](#example).

![](resource/markdom-navigation-up.png)

Every `Node` object in a Domain Model API object graph that is not a `Document` object has a reference to its [parent](#api-dom-node-getparent), which is

* a `BlockParent` object, if the `Node` object is a `Block` object,
* a `ListBlock` object, if the `Node` object is a `ListItem` object, or
* a `ContentParent` object, if the `Node` object is a `Content` object.

Depending on the kind of the parent, either `getBlockParentType`, `getListBlockType` or `getContentParentType` can be used to find out the actual type of the parent.

For example: Consider a method that gets leaf `Node` object as a parameter without any knowledge about it. Calling `getNodeType` reveals that is it a `Content` object. Calling `getContentType()` reveals that it is a `TextContent` object. Calling `getParent()` returns the parent as a `ContentParent` object. Calling `getContentParentType()` on the parent reveals that the parent is en `EmphasisContent` object.

Each `Node` object in a Domain Model API object graph can, through its predecessors, retrieve the root `Document` object. Each `Content` object in a Domain Model API object graph can, through its predecessors, retrieve its `Block` object.

##### Down navigation

The following image shows the possible methods to navigate from an `Node` object downwards to the leaf `TextContent` object with `text` value `Baz` of the [eample document](#example).

![](resource/markdom-navigation-down.png)

Every `Node` object in a Domain Model API object graph that is parent object has a reference to its companion `Sequence` object which has references to the [children](#api-dom-node-getchildren) of the `Node` object.

For example: Consider a method that gets the root `Node` as a parameter without any knowledge about it. Calling `getNodeType()` reveals that is it a `Document` object. Calling `getBlocks` returns the companion `Sequence` object. Calling `size` reveals that the companion `Sequence` object contains three `Block` objects. Calling `get(1)` returns the second child. Calling `getBlockType()` on the second child reveals that it is a `OrderedListBlock` object.

##### Detecting cycles {#api-dom-detecting-cycles}

If a `Block`, `ListItem` or `Content` object is about to be attached to a `BlockParent`, `ListBlock` or `ContentParent` object, it is necessary to check, whether this would create a cycle.

If the designated parent object is part of a *Markdom document* (i.e. a tree of `Node` objects which has a root `Document` object), it is not possible to create a cycle, because repeatedly retrieving the [parent](#api-dom-node-getparent) will eventually yield the root `Document` object which by definition doesn't [have](#api-dom-node-hasparent) a parent object and thus ending the path after a finite number of repetitions.

If the designated parent object is not part of a *Markdom document* (and therefore part of a tree fragment where one `Node` object that isn't a `Document` object doesn't have a parent), it is possible to create a cycle. The object that is about to be attached doesnt't currently have a parent or otherwise it wouldn't be eligible to be attached to the a parent. The only object in the tree fragment the designated parent is a part of that doesn't have a parent is the root object of that tree fragment. It is therefore necessary and sufficient to check whether the object that is about to be attached is not the root of the tree fragment that the designated parent is a part of, to ensure that no cycle is created.

This can be accomplished by repeatedly retrieving the parent of the designated parent until a `Node` object that doesn't have a parent is found and check whether the last `Node` object in the chain of parent is the object that is about to be added.

For example: Assuming a `QuoteBlock` object is about to be attached to a `BlockParent` object. That `BlockParent` object might be another `QuoteBlock` object which is attached to a `ListItem` object which is attached to an `UnorderedListBlock` object which is attached to a `QuoteBlock` object which isn't attached to a `BlockParent` object.  The last `QuoteBlock` in the chain of parents might be the same `QuoteBlock` object that is about to be attached. Attaching the `QuoteBlock` object to its designated parent would therefore create a cycle. The following image illustrates this example.

![](resource/markdom-cycles.png)

### Handler API {#api-handler}

The Handler API represents a [*Markdom document*](#domain-document) as a sequence of events.

#### Events {#api-handler-event}

The Handler API defines several events and the order in which the must occur to successfully describe a [*Markdom document*](#domain-document).

To help different handler implementations to process the described [*Markdom document*](#domain-document) easily, some of the events carry redundant information:
 * Every event concerning a [*Markdom node*](#domain-node) of a polymorph kind (i.e. [*Markdom blocks*](#domain-block) and [*Markdom contents*](#domain-content)) is reported twice. Once in a general form and once in a specific form with the specific parameters. This allows to execute general or specific behavior when necessary.
 * Every event that is part of a pair (i.e. opening and closing events) and has parameters, carries the same values for the parameters as its counterpart. This usually eliminates the need to remember previously reported values.

##### Document {#api-handler-event-document}

A [*Markdom document*](#domain-document) is represented by an `onDocumentBegin` event and an `onDocumentEnd` event that frame the events that describe the sequence of [*Markdom blocks*](#domain-block) the described [*Markdom document*](#domain-document) consists of. 

![](resource/markdom-events-document.png)

##### Blocks {#api-handler-event-blocks}

A sequence of [*Markdom blocks*](#domain-block) is represented by an `onBlocksBegin` event and an `onBlocksEnd` event that frame the events that describe the [*Markdom blocks*](#domain-block). Consecutive [*Markdom blocks*](#domain-block) are separated by an `onNextBlock` event.

![](resource/markdom-events-blocks.png)

##### Block {#api-handler-event-block}

A [*Markdom block*](#domain-block) is represented by an `onBlockBegin` event and an `onBlockEnd` event that frame the events that describe the [*Markdom block*](#domain-block). Both events carry the [`BlockType`](#api-common-blocktype)  value that corresponds to the type of the described [*Markdom block*](#domain-block) as a parameter.

* If the [*Markdom block*](#domain-block) is a [*Markdom code block*](#domain-codeblock), it is represented as an `onCodeBlock` event. The event carries the `code` parameter of the [*Markdom code block*](#domain-codeblock).
* If the [*Markdom block*](#domain-block) is a [*Markdom heading content*](#domain-headingblock), it is represented as an `onHeadingBlockBegin` event and an `onHeadingBlockEnd` event that frame the events that describe the sequence of [*Markdom contents*](#domain-content) the described *Markdom heading* consists of. Both events carry the `level` parameter of the described [*Markdom heading content*](#domain-headingblock).
* If the [*Markdom block*](#domain-block) is a [*Markdom division content*](#domain-divisionblock), it is represented as an `onDivisionBlock` event.
* If the [*Markdom block*](#domain-block) is an [*ordered Markdom list block*](#domain-orderedlistblock), it is represented as an `onOrderedListBlockBegin` event and an `onOrderedListBlockEnd` event that frame the events that describe the sequence of *Markdom list items* the described [*ordered Markdom list block*](#domain-orderedlistblock) consists of. Both events carry the `startIndex` parameter of the described [*ordered Markdom list block*](#domain-orderedlistblock).
* If the [*Markdom block*](#domain-block) is a [*Markdom paragraph content*](#domain-paragraphblock), it is represented as an `onParagraphBlockBegin` event and an `onParagraphBlockEnd` event that frame the events that describe the sequence of [*Markdom contents*](#domain-content) the described [*Markdom paragraph content*](#domain-paragraphblock) consists of.
* If the [*Markdom block*](#domain-block) is a [*Markdom quote content*](#domain-quoteblock), it is represented as an `onQuoteBlockBegin` event and an `onQuoteBlockEnd` event that frame the events that describe the sequence of [*Markdom blocks*](#domain-block) the described [*Markdom quote content*](#domain-quoteblock) consists of.
* If the [*Markdom block*](#domain-block) is an [*unordered Markdom list block*](#domain-unorderedlistblock), it is represented as an `onUNorderedListBlockBegin` event and an `onUnorderedListBlockEnd` event that frame the events that describe the sequence of *Markdom list items* the described [*unordered Markdom list block*](#domain-unorderedlistblock) consists of.

![](resource/markdom-events-block.png)

##### List Items {#api-handler-event-listitems}

A sequence of *Markdom list items* is represented by an `onListItemsBegin` event and an `onListItemEnd` event that frame the events that describe the *Markdom list items*. Consecutive *Markdom list items* are separated by an `onNextListItem` event.

![](resource/markdom-events-listitems.png)

##### List Item {#api-handler-event-listitem}

A *Markdom list item* is represented by an `onListItemBegin` event and an `onListItemEnd` event that frame the events that describe the sequence of [*Markdom blocks*](#domain-block) the described *Markdom list item* consists of. 

![](resource/markdom-events-listitem.png)

##### Contents {#api-handler-event-contents}

A sequence of [*Markdom contents*](#domain-content) is represented by an `onContentsBegin` event and an `onContentsEnd` event that frame the events that describe the *Markdom contents*. Consecutive [*Markdom contents*](#domain-content) are separated by an `onNextContent` event.

![](resource/markdom-events-contents.png)

##### Content {#api-handler-event-content}

A [*Markdom content*](#domain-content) is represented by an `onContentBegin` event and an `onContentEnd` event that frame the events that describe the *Markdom content*. Both events carry the `ContentType` value that corresponds to the type of the of the described [*Markdom content*](#domain-content) as a parameter.

* If the [*Markdom content*](#domain-content) is a [*Markdom code content*](#domain-codecontent), it is represented as an `onCodeContent` event. The event carries `code` parameter of the described [*Markdom code content*](#domain-codecontent).
* If the [*Markdom content*](#domain-content) is a [*Markdom emphasis content*](#domain-emphasiscontent), it is represented as an `onEmphasisContentBegin` event and an `onEmphasisContentEnd` event that frame the events that describe the sequence of [*Markdom contents*](#domain-content) the described *Markdom conent* consists of. Both events carry the `level` parameter of the described [*Markdom emphasis content*](#domain-emphasiscontent).
* If the [*Markdom content*](#domain-content) is a [*Markdom image content*](#domain-imagecontent), it is represented as an `onImageContent` event. The event carries the `uri`, `title` and `alternative` parameters of the described [*Markdom image content*](#domain-imagecontent).
If the [*Markdom content*](#domain-content) is a [*Markdom line break content*](#domain-linebreakcontent), it is represented as an `onLineBreakContent` event. The event carries the `hard` parameter of the described [*Markdom line break content*](#domain-linebreakcontent).
* If the [*Markdom content*](#domain-content) is a [*Markdom link content*](#domain-linkcontent), it is represented as an `onLinkContentBegin` event and an `onLinkContentEnd` event that frame the events that describe the sequence of [*Markdom contents*](#domain-content) the described [*Markdom link content*](#domain-linkcontent) consists of. Both events carry the `uri` parameter of the described [*Markdom link content*](#domain-linkcontent).

![](resource/markdom-events-content.png)

#### Interfaces {#api-handler-interface}

##### `Dispatcher` {#api-handler-dispatcher}
A `Dispatcher` is a component that is able to describe a [*Markdom document*](#domain-document) to a [`Handler`](#api-handler-handler) by dispatching a sequence of [*Markdom events*](#api-handler-event).

###### `handle` {#api-handler-dispatcher-handle}

A [`Dispatcher`](#api-handler-dispatcher) object must have a method with signature `Object handle(Handler handler)`.

This method must dispatch [*Markdom events*](#api-handler-event) that describe the [*Markdom document*](#domain-document) to `handler` in the correct order.
  
This method must return the [result](#api-handler-handler-getresult) of `handler`.

Calling this method multiple times on a [`Dispatcher`](#api-handler-dispatcher) that is not [repeatable](#api-handler-isrepeatable) has an undefined behavior, unless explicitly stated otherwise.
  
This method must fail if `handler` is not present.

###### `isRepeatable` {#api-handler-isrepeatable}

A [`Dispatcher`](#api-handler-dispatcher) object must have a method with signature `Boolean isRepeatable()`.
  
This method must return whether the [`Dispatcher`](#api-handler-dispatcher) object is able to describe a [*Markdom document*](#domain-document) multiple times (e.g. a [`Document`](#api-dom-document) object can describe itself multiple times, but a CommonMark parsing [`Handler`](#api-handler-handler) object that consumes its input can't).
  
##### `Handler` {#api-handler-handler}

A `Handler` is a component that is able to receive a a sequence of [*Markdom events*](#api-handler-event) that describe a [*Markdom document*](#domain-document) from a [`Dispatcher`](#api-handler-dispatcher) and calculate a result or causes side effects that corresponds to the described [*Markdom document*](#domain-document) (e.g. a [`Handler`](#api-handler-dispatcher) object can generate a [`Document`](#api-dom-document) object or write CommonMark text into a file).

Calling methods on a  [`Handler`](#api-handler-handler) in an order that doesn't properly describe a [*Markdom document*](#domain-document) has an undefined behavior.

###### `getResult` {#api-handler-handler-getresult}

A [`Handler`](#api-handler-handler) object must have a method with signature `Object getResult()`.

This method must return the calculated result that corresponds to the described [*Markdom document*](#domain-document) (e.g. a [`Handler`](#api-handler-dispatcher) object that generates a [`Document`](#api-dom-document) object returns that [`Document`](#api-dom-document) object and a [`Handler`](#api-handler-dispatcher) object that writes CommonMark text into a file returns `null`).

Calling this method before the [`onDocumentEnd`](#api-handler-handler-ondocumentend) event has been received yields an undefined result, unless explicitly stated otherwise.

###### `onBlockBegin` {#api-handler-handler-onblockbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onBlockBegin(BlockType type)`.

The `type` parameter determines the type of the represented [*Markdom block*](#domain-block).

The behavior of this method is undefined, If `type` is not present.

Calling this method must be must be, depending on the value of the `type` parameter, followed by a call to `onCodeBlock` or `onHeadinBlockBegin` or `onDivisionBlock` or `onOrderedListBlockBegin` or `onParagraphBlockBegin` or `onQuoteBlockBegin` or `onUnorderedListBlockBegin`. A corresponding call to `onBlockEnd` must eventually occur.

###### `onBlockEnd` {#api-handler-handler-onblockend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onBlockEnd(BlockType type)`.

The `type` parameter must have the same value as the corresponding call to `onBlockBegin`.

The behavior of this method is undefined, If `type` is not present.

A corresponding call to `onBlockBegin` must have occurred. Calling this method must be followed by a call to `onNextBlock` or `onBlocksEnd.

###### `onBlocksBegin` {#api-handler-handler-onblocksbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onBlocksBegin()`.

Calling this method must be must be followed by a call to `onBlockBegin` or `onBlocksEnd`. A corresponding call to `onBlocksEnd` must eventually occur.

###### `onBlocksEnd` {#api-handler-handler-onblocksend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onBlocksEnd()`.

A corresponding call to `onBlocksBegin` must have occurred. Calling this method must be followed by a call to `onDocumentEnd` or `onQuoteBlockEnd` or `onListItemEnd`.

###### `onCodeBlock` {#api-handler-handler-oncodeblock}

A [`Handler`](#api-handler-handler) object must have a method with signature `onCodeBlock(String code, String? hint)`.

The parameters determine the `code` parameter and the `hint` parameter of the represented [*Markdom code block*](#domain-codeblock).

The behavior of this method is undefined, If `code` is not present.

Calling this method must be must be followed by a call to `onBlockEnd`.

###### `onCodeContent` {#api-handler-handler-oncodecontent}

A [`Handler`](#api-handler-handler) object must have a method with signature `onCodeContent(String code)`.

The parameter determines the `code` parameter of the represented [*Markdom code content*](#domain-codecontent).

The behavior of this method is undefined, If `code` is not present.

Calling this method must be must be followed by a call to `onBlockEnd`.

###### `onContentBegin` {#api-handler-handler-oncontentbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onContenBegin(ContentType type)`.

The `type` parameter determines the type of the represented [*Markdom content*](#domain-content).

The behavior of this method is undefined, If `type` is not present.

Calling this method must be must be, depending on the value of the `type` parameter, followed by a call to `onCodeContent` or `onEmphasisContentBegin` or `onImageContent` or `onLineBreakContent` or `onLinkContentBegin` or `onTextContent`. A corresponding call to `onContentEnd` must eventually occur.

###### `onContentEnd` {#api-handler-handler-oncontentend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onContentEnd(ContentType type)`.

The `type` parameter must have the same value as the corresponding call to `onContentBegin`.

The behavior of this method is undefined, If `type` is not present.

A corresponding call to `onContentBegin` must have occurred. Calling this method must be followed by a call to `onNextContent` or `onContentsEnd.

###### `onContentsBegin` {#api-handler-handler-oncontentsbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onContentsBegin()`.

Calling this method must be must be followed by a call to `onContentBEgin` or `onContentsEnd`. A corresponding call to `onContentsEnd` must eventually occur.

###### `onContentsEnd` {#api-handler-handler-oncontentsend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onContentsEnd()`.

A corresponding call to `onContentsBegin` must have occurred. Calling this method must be followed by a call to `onHeadingBlockEnd` or `onParagraphBlockEnd` or `onEmphasisContentEnd` or `onLinkContentEnd`.

###### `onDivisionBlock` {#api-handler-handler-ondivisionblock}

A [`Handler`](#api-handler-handler) object must have a method with signature `onDivisionBlock()`.

Calling this method must be must be followed by a call to `onBlockEnd`.

###### `onDocumentBegin` {#api-handler-handler-ondocumentbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onDocumentBegin()`.

This is the first method that must be called. Calling this method must be must be followed by a call to `onBlocksBegin`. A corresponding call to `onDocumentEnd` must eventually occur.

###### `onDocumentEnd` {#api-handler-handler-ondocumentend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onDocumentEnd()`.

A corresponding call to `onDocumentBegin` must have occurred. Calling this method may be followed by a call to `getResult`.

###### `onHeadingBlockBegin` {#api-handler-handler-onheadingblockbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onHeadingBlockBegin(HeadingLevel level)`.

The parameter determines the `level` parameter of the represented [*Markdom heading block*](#domain-headingblock).

The behavior of this method is undefined, If `level` is not present.

Calling this method must be must be followed by a call to `onContentsBegin`. A corresponding call to `onHeadingBlockEnd` must eventually occur.

###### `onHeadingBlockEnd` {#api-handler-handler-onheadingblockend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onHeadingBlockEnd(HeadingLevel level)`.

The `level` parameter must have the same value as the corresponding call to `onHeadingBlockBegin`.

The behavior of this method is undefined, If `level` is not present.

A corresponding call to `onHeadingBlockBegin` must have occurred. Calling this method must be followed by a call to `onBlockEnd`.

###### `onEmphasisContentBegin` {#api-handler-handler-onemphasiscontentbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onEmphasisContentBegin(EmphasisLevel level)`.

The parameter determines the `level` parameter of the represented [*Markdom emphasis content*](#domain-emphasiscontent).

The behavior of this method is undefined, If `level` is not present.

Calling this method must be must be followed by a call to `onContentsBegin`. A corresponding call to `onEmphasisContentEnd` must eventually occur.

###### `onEmphasisContentEnd` {#api-handler-handler-onemphasiscontentend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onEmphasisContentEnd(EmphasisLevel level)`.

The `level` parameter must have the same value as the corresponding call to `onEmphasisContentBegin`.

The behavior of this method is undefined, If `level` is not present.

A corresponding call to `onEmphasisContentBegin` must have occurred. Calling this method must be followed by a call to `onContentEnd`.

###### `onImageContent` {#api-handler-handler-onimagecontent}

A [`Handler`](#api-handler-handler) object must have a method with signature `onImageContent(String uri, String? title, String? alternative)`.

The parameters determine the `uri` parameter and the `title` parameter and the `alternative` parameter of the represented [*Markdom image content*](#domain-imagecontent).

The behavior of this method is undefined, If `uri` is not present.

The behavior of this method is undefined, If `uri` is not a valid URI reference.

Calling this method must be must be followed by a call to `onBlockEnd`.

###### `onLineBreakContent` {#api-handler-handler-onlinebreakcontent}

A [`Handler`](#api-handler-handler) object must have a method with signature `onLineBreakContent(Boolean hard)`.

The parameter determines the `hard` parameter of the represented [*Markdom line break content*](#domain-linebreakcontent).

The behavior of this method is undefined, If `hard` is not present.

Calling this method must be must be followed by a call to `onBlockEnd`.

###### `onLinkContentBegin` {#api-handler-handler-onlinkcontentbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onLinkContentBegin(String uri)`.

The parameter determines the `uri` parameter of the represented [*Markdom link content*](#domain-linkcontent).

The behavior of this method is undefined, If `uri` is not present.

The behavior of this method is undefined, If `uri` is not a valid URI reference.

Calling this method must be must be followed by a call to `onContentsBegin`. A corresponding call to `onLinkContentEnd` must eventually occur.

###### `onLinkContentEnd` {#api-handler-handler-onlincontentend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onLinkContentEnd(String uri)`.

The `uri` parameter must have the same value as the corresponding call to `onLinkContentBegin`.

The behavior of this method is undefined, If `uri` is not present.

The behavior of this method is undefined, If `uri` is not a valid URI reference.

A corresponding call to `onLinkContentBegin` must have occurred. Calling this method must be followed by a call to `onContentEnd`.

###### `onListItemBegin` {#api-handler-handler-onlistitembegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onListItemBegin()`.

Calling this method must be must be followed by a call to `onBlocksBegin`. A corresponding call to `onListItemEnd` must eventually occur.

###### `onListItemEnd` {#api-handler-handler-onlistitemend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onListItemEnd()`.

A corresponding call to `onListItemBegin` must have occurred. Calling this method must be followed by a call to `onNextListItem` or `onListItemsEnd.

###### `onListItemsBegin` {#api-handler-handler-onlistitemsbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onListItemsBegin()`.

Calling this method must be must be followed by a call to `onListItemBegin` or `onListItemsEnd`. A corresponding call to `onListItemsEnd` must eventually occur.

###### `onListItemsEnd` {#api-handler-handler-onlistitemsend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onListItemsEnd()`.

A corresponding call to `onListItemsBegin` must have occurred. Calling this method must be followed by a call to `onOrderedListBlockEnd` or `onUnorderedListBlockEnd`.

###### `onNextBlock` {#api-handler-handler-onnextblock}

A [`Handler`](#api-handler-handler) object must have a method with signature `onNextBlock()`.

Calling this method must be must be followed by a call to `onBlockBegin`.

###### `onNextContent` {#api-handler-handler-onnextcontent}

A [`Handler`](#api-handler-handler) object must have a method with signature `onNextContent()`.

Calling this method must be must be followed by a call to `onContentBegin`.

###### `onNextListItem` {#api-handler-handler-onnextlistitem}

A [`Handler`](#api-handler-handler) object must have a method with signature `onNextListItem()`.

Calling this method must be must be followed by a call to `onListItemBegin`.

###### `onOrderedListBlockBegin` {#api-handler-handler-onorderedlistblockbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onOrderedListBlockBegin(Integer startIndex)`.

The parameter determines the `startIndex` parameter of the represented [*ordered Markdom list block*](#domain-orderedlistblock).

The behavior of this method is undefined, If `startIndex` is not present.

The behavior of this method is undefined, If `startIndex` is negative.

Calling this method must be must be followed by a call to `onListItemsBegin`. A corresponding call to `onOrderedListBlockEnd` must eventually occur.

###### `onOrderedListBlockEnd` {#api-handler-handler-onorderedlistblockend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onOrderedListBlockEnd(Integer startIndex)`.

The `startIndex` parameter must have the same value as the corresponding call to `onOrderedListBlockBegin`.

The behavior of this method is undefined, If `startIndex` is not present.

The behavior of this method is undefined, If `startIndex` is negative.

A corresponding call to `onOrderedListBlockBegin` must have occurred. Calling this method must be followed by a call to `onBlockEnd`.

###### `onParagraphBlockBegin` {#api-handler-handler-onquoteblockbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onParagraphBlockBegin()`.

Calling this method must be must be followed by a call to `onContentsBegin`. A corresponding call to `onParagraphBlockEnd` must eventually occur.

###### `onParagraphBlockEnd` {#api-handler-handler-onquoteblockend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onParagraphBlockEnd()`.

A corresponding call to `onParagraphBlockBegin` must have occurred. Calling this method must be followed by a call to `onBlockEnd`.

###### `onQuoteBlockBegin` {#api-handler-handler-onquoteblockbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onQuoteBlockBegin()`.

Calling this method must be must be followed by a call to `onBlocksBegin`. A corresponding call to `onQuoteBlockEnd` must eventually occur.

###### `onQuoteBlockEnd` {#api-handler-handler-onquoteblockend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onQuoteBlockEnd()`.

A corresponding call to `onQuoteBlockBegin` must have occurred. Calling this method must be followed by a call to `onBlockEnd`.

###### `onUnorderedListBlockBegin` {#api-handler-handler-onunorderedlistblockbegin}

A [`Handler`](#api-handler-handler) object must have a method with signature `onUnorderedListBlockBegin()`.

Calling this method must be must be followed by a call to `onListItemsBegin`. A corresponding call to `onUnorderedListBlockEnd` must eventually occur.

###### `onUnorderedListBlockEnd` {#api-handler-handler-onunorderedlistblockend}

A [`Handler`](#api-handler-handler) object must have a method with signature `onUnorderedListBlockEnd()`.

A corresponding call to `onUnorderedListBlockBegin` must have occurred. Calling this method must be followed by a call to `onBlockEnd`.

###### `onTextContent` {#api-handler-handler-ontextcontent}

A [`Handler`](#api-handler-handler) object must have a method with signature `onTextContent(String text)`.

The parameter determines the `text` parameter of the represented [*Markdom text content*](#domain-textcontent).

The behavior of this method is undefined, If `text` is not present.

Calling this method must be must be followed by a call to `onBlockEnd`.

#### Additional information

#### Example Document {#api-handler-example}

The following sequence of events describes the [example document](#example):

```
onDocumentBegin()
  onBlocksBegin()
    onBlockBegin(HEADING)
    onHeadingBlockBegin(LEVEL_1)
      onContentsBegin()
        onContentBegin(TEXT)
        onTextContent("Markdom")
        onContentEnd(TEXT)
      onContentsEnd()
    onHeadingBlockEnd(LEVEL_1)
    onBlockEnd(HEADING)
  onNextBlock()
    onBlockBegin(ORDERED_LIST)
    onOrderedListBlockBegin(1)
      onListItemsBegin()
        onListItemBegin()
          onBlocksBegin()
            onBlockBegin(PARAGRAPH)
            onParagraphBlockBegin()
              onContentsBegin()
                onContentBegin(LINK)
                onLinkContentBegin("#Bar")
                  onContentsBegin()
                    onContentBegin(TEXT)
                    onTextContent("Foo")
                    onContentEnd(TEXT)
                  onContentsEnd()
                onLinkContentEnd("#Bar")
                onContentEnd(LINK)
              onContentsEnd()
            onParagraphBlockEnd()
            onBlockEnd(PARAGRAPH)
          onBlocksEnd()
        onListItemEnd()
      onNextListItem()
        onListItemBegin()
          onBlocksBegin()
            onBlockBegin(PARAGRAPH)
            onParagraphBlockBegin()
              onContentsBegin()
                onContentBegin(TEXT)
                onTextContent("Lorem ipsum")
                onContentEnd(TEXT)
              onNextContent()
                onContentBegin(LINE_BREAK)
                onLineBreakContent(true)
                onContentEnd(LINE_BREAK)
              onNextContent()
                onContentBegin(CODE)
                onCodeContent("dolor sit amet")
                onContentEnd(CODE)
              onContentsEnd()
            onParagraphBlockEnd()
            onBlockEnd(PARAGRAPH)
          onBlocksEnd()
        onListItemEnd()
      onNextListItem()
        onListItemBegin()
          onBlocksBegin()
            onBlockBegin(QUOTE)
            onQuoteBlockBegin()
              onBlocksBegin()
                onBlockBegin(PARAGRAPH)
                onParagraphBlockBegin()
                  onContentsBegin()
                    onContentBegin(EMPHASIS)
                    onEmphasisContentBegin(LEVEL_1)
                      onContentsBegin()
                        onContentBegin(TEXT)
                        onTextContent("Baz")
                        onContentEnd(TEXT)
                      onContentsEnd()
                    onEmphasisContentEnd(LEVEL_1)
                    onContentEnd(EMPHASIS)
                  onContentsEnd()
                onParagraphBlockEnd()
                onBlockEnd(PARAGRAPH)
              onBlocksEnd()
            onQuoteBlockEnd()
            onBlockEnd(QUOTE)
          onBlocksEnd()
        onListItemEnd()
      onListItemsEnd()
    onOrderedListBlockEnd(1)
    onBlockEnd(ORDERED_LIST)
  onNextBlock()
    onBlockBegin(CODE)
    onCodeBlock("goto 11", "")
    onBlockEnd(CODE)
  onBlocksEnd()
onDocumentEnd()
```

## Data representations {#data}

### JSON {#data-json}

#### Example Document {#data-json-example}

The following JSON document represents the [example document](#example):

```json
{
  "$schema": "http://schema.markdom.io/markdom-1.0.json#",
  "version": "1.0",
  "blocks": [
    {
      "type": "Heading",
      "level": 1,
      "contents": [
        {
          "type": "Text",
          "text": "Markdom"
        }
      ]
    },
    {
      "type": "OrderedList",
      "startIndex": 1,
      "items": [
        {
          "blocks": [
            {
              "type": "Paragraph",
              "contents": [
                {
                  "type": "Link",
                  "uri": "#Bar",
                  "contents": [
                    {
                      "type": "Text",
                      "text": "Foo"
                    }
                  ]
                }
              ]
            }
          ]
        },
        {
          "blocks": [
            {
              "type": "Paragraph",
              "contents": [
                {
                  "type": "Text",
                  "text": "Lorem ipsum"
                },
                {
                  "type": "LineBreak",
                  "hard": true
                },
                {
                  "type": "Code",
                  "code": "dolor sit amet"
                }
              ]
            }
          ]
        },
        {
          "blocks": [
            {
              "type": "Quote",
              "blocks": [
                {
                  "type": "Paragraph",
                  "contents": [
                    {
                      "type": "Emphasis",
                      "level": 1,
                      "contents": [
                        {
                          "type": "Text",
                          "text": "Baz"
                        }
                      ]
                    }
                  ]
                }
              ]
            }
          ]
        }
      ]
    },
    {
      "type": "Code",
      "code": "goto 11"
    }
  ]
}
```

### YAML {#data-yaml}

#### Example Document {#data-yaml-example}

```yaml
---
version: '1.0'
blocks:
- type: Heading
  level: 1
  contents:
  - type: Text
    text: Markdom
- type: OrderedList
  startIndex: 1
  items:
  - blocks:
    - type: Paragraph
      contents:
      - type: Link
        uri: "#Bar"
        contents:
        - type: Text
          text: Foo
  - blocks:
    - type: Paragraph
      contents:
      - type: Text
        text: Lorem ipsum
      - type: LineBreak
        hard: true
      - type: Code
        code: dolor sit amet
  - blocks:
    - type: Quote
      blocks:
      - type: Paragraph
        contents:
        - type: Emphasis
          level: 1
          contents:
          - type: Text
            text: Baz
- type: Code
  code: goto 11
```

### XML {#data-xml}

#### Example Document {#data-xml-example}

The following XML document represents the [example document](#example):

```xml
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<Document version="1.0" xmlns="http://schema.markdom.io/markdom-1.0.xsd">
  <Heading level="1">
    <Text>Markdom</Text>
  </Heading>
  <OrderedList startIndex="1">
    <ListItem>
      <Paragraph>
        <Link uri="#Bar">
          <Text>Foo</Text>
        </Link>
      </Paragraph>
    </ListItem>
    <ListItem>
      <Paragraph>
        <Text>Lorem ipsum</Text>
        <LineBreak hard="true"/>
        <Code/>
      </Paragraph>
    </ListItem>
    <ListItem>
      <Quote>
        <Paragraph>
          <Emphasis level="1">
            <Text>Baz</Text>
          </Emphasis>
        </Paragraph>
      </Quote>
    </ListItem>
  </OrderedList>
  <Code>goto 11</Code>
</Document>
```

## Text representations {#text}

### HTML {#text-html}

#### Example Document {#text-html-example}

The following HTML document represents the [example document](#example):


### CommonMark {#text-cm}