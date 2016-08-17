![logo](resource/markdom-mark-positive-inline.png)

---

# Markdom

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

    # Markdom
    
    1. [Foo](#Bar)
    2. *Lorem ipsum*\
       `goto 11`
    3. > Baz
    
    ```
    goto 11
    ```


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

The following image shows the interfaces that are part of the Domain Object API:

![](resource/markdom-nodes.png)

*Interfaces with bold names are eminently suitable to be implmented as final classes, interfaces with italic names are eminently suitable to be implmented as abstract classes and interfaces with normal names are eminently suitable to be implmented as traits or mixins.*

##### `Block` {#api-dom-block}

A [`Block`](#api-dom-block) object is a [`Node`](#api-dom-node) object that represents a [*Markdom block*](#domain-block).

###### Constructors {api-dom-block-constructor}

An implementation of `Block` should have a constructor with signature `Block()`.

###### `getBlockType` {api-dom-block-getblocktype}

A [`Block`](#api-dom-block) object must have a method with signature `BlockType getBlockType()`.  

This method must return the `BlockType` value that correspondes to the type of the [`Block`](#api-dom-block) object.

##### `BlockParent` {#api-dom-blockparent}

A [`BlockParent`](#api-dom-blockparent) object is a [`Node`](#api-dom-node) object that represents a [*Markdom node*](#domain-node) that contains [*Markdom blocks*](#domain-block).

An implementation of `BlockParent` must have a final and initially empty `Sequence` of [`Block`](#api-dom-block) objects that is associated with the [`BlockParent`](#api-dom-blockparent) object.

Any structural modification (insert, remove, clear, replace) to the associated `Sequence` of [`Block`](#api-dom-block) objects must reflect the fact, that a [`Block`](#api-dom-block) object that is added to the associated `Sequence` object is attached to the [`BlockParent`](#api-dom-blockparent) object until is is removed from the associated `Sequence` of [`Block`](#api-dom-block) objects.

Attaching a [`Block`](#api-dom-block) object to the [`BlockParent`](#api-dom-blockparent) object  must fail
* if the [`Block`](#api-dom-block) object is not present, or  
* if the [`Block`](#api-dom-block) object is already attached to a [`BlockParent`](#api-dom-blockparent) object, or 
* if attaching the [`Block`](#api-dom-block) object to the [`BlockParent`](#api-dom-blockparent) object would create a cycle in the tree of Markdom nodes that the [`BlockParent`](#api-dom-blockparent) object is part of.

###### Constructors {api-dom-blockparent-constructor}

An implementation of `BlockParent` should have a constructor with signature `BlockParent()`.

For convenience, an implementation of `BlockParent` should have a constructor with signature `BlockParent(Block... blocks)` that takes that takes an array of [`Block`](#api-dom-block) objects named `blocks` and delegates to `addBlocks(Block... blocks)`.

###### `getBlocks` {api-dom-blockparent-getblocks}

A [`Block`](#api-dom-block) object must have a method with signature `Sequence getBlocks()`.

This method must return the associated `Sequence` of [`Block`](#api-dom-block) objects.

###### `addBlock` {api-dom-blockparent-addblock}

For convenience, a [`BlockParent`](#api-dom-blockparent) object should have a method with signature `addBlock(Block block)` that takes a [`Block`](#api-dom-block) object named `block`.

This method must add `block` at the end of the associated `Sequence` of [`Block`](#api-dom-block) objects. This attaches `block` to the [`BlockParent`](#api-dom-blockparent) object.

This method must fail if add `block` to the  associated `Sequence` of [`Block`](#api-dom-block) objects failed.

###### `addBlocks` {api-dom-blockparent-addblocks}

For convenience, a [`BlockParent`](#api-dom-blockparent) object should have a method with signature `addBlocks(Block... blocks)` that takes an array of [`Block`](#api-dom-block) objects named `blocks`.

This method must add all [`Block`](#api-dom-block) objects from `blocks` in the given order at the end of the `Sequence` of [`Block`](#api-dom-block) objects of the [`BlockParent`](#api-dom-blockparent) object, as if `addBlock(Block block)` has been called repeatedly for all [`Block`](#api-dom-block) objects from `blocks`.  This attaches all [`Block`](#api-dom-block) objects from `blocks` to the [`BlockParent`](#api-dom-blockparent) object.
  
This method must fail if `blocks` is not present.
This method must fail if adding any [`Block`](#api-dom-block) object from `blocks` to the  associated `Sequence` of [`Block`](#api-dom-block) objects failed.
  
Because this method is a short hand for repeated calls to `addBlock(Block block)`, it must add all prior [`Block`](#api-dom-block) objects from `blocks` to the  associated `Sequence` of [`Block`](#api-dom-block) objects, if it fails beacause of a violating [`Block`](#api-dom-block) object from `blocks`.

##### `CodeBlock` {#api-dom-codeblock}

A [`CodeBlock`](#api-dom-codeblock) objects is a [`Block`](#api-dom-block) object that represents a [*Markdom code block*](#domain-codeblock).

###### Constructors {api-dom-codeblock-constructor}

An implementation of `CodeBlock` should have a constructor with signature `CodeBlock()` that set the code of the [`CodeBlock`](#api-dom-codeblock) object to the empty string ant the hint of the `Code` object to be not present.

For convenience, an implementation of `CodeBlock` should have a constructor with signature `CodeBlock(String code)` that takes a `String` named `code` and delegates to `setCode(String code)`.

For convenience, an implementation of `CodeBlock` should have a constructor with signature `CodeBlock(String code, String hint)` that takes a `String` named `code` and a `String` named `hint` and delegates to `setCode(String code)` and `setHint(String hint)`.

###### `getCode` {api-dom-codeblock-getcode}

A [`CodeBlock`](#api-dom-codeblock) object must have a method with signature `String getCode()`.

The method must return the code of the [`CodeBlock`](#api-dom-codeblock) object.

###### `setCode` {api-dom-codeblock-setcode}

A [`CodeBlock`](#api-dom-codeblock) object must have a method with signature `setCode(String code)` that takes a `String` named `code`.

This method must set the code of the [`CodeBlock`](#api-dom-codeblock) object. 
  
This method must fail if `code` is not present.

###### `getHint` {api-dom-codeblock-gethint}

A [`CodeBlock`](#api-dom-codeblock) object must have a method with signature `String? getHint()`.

The method must return the optional hint of the [`CodeBlock`](#api-dom-codeblock) object.

###### `setHint` {api-dom-codeblock-sethint}
  
A [`CodeBlock`](#api-dom-codeblock) object must have a method with signature `setHint(String? hint)` that takes an optional `String` named `hint`.

This method must set the hint of the [`CodeBlock`](#api-dom-codeblock) object. 
 	
##### `CodeContent` {#api-dom-codecontent}

A [`CodeContent`](#api-dom-codecontent) object is a [`Content`](#api-dom-content) object that represents a [*Markdom code content*](#domain-codecontent).

###### Constructors {#api-dom-codecontent-constructors}

An implementation of `CodeContent` should have a constructor with signature `CodeContent()` that set the code of the [`CodeContent`](#api-dom-codecontent) object to the empty string.

For convenience, an implementation of `CodeContent` should have a constructor with signature `CodeContent(String code)` that takes a `String` named `code` and delegates to `setCode(String code)`.

###### `getCode` {#api-dom-codecontent-getcode}

A [`CodeContent`](#api-dom-codecontent) object must have a method with signature `String getCode()`.

The method must return the code of the [`CodeContent`](#api-dom-codecontent) object.

###### `setCode` {#api-dom-codecontent-setcode}

A [`CodeContent`](#api-dom-codecontent) object must have a method with signature `setCode(String code)` that takes a `String` named `code`.

This method must set the code of the [`CodeContent`](#api-dom-codecontent) object. 
  
This method must fail if `code` is not present.

##### `Content` {#api-dom-content}

A Content` object is a [`Node`](#api-dom-node) object that represents a [*Markdom content*](#domain-content).

###### Constructors {#api-dom-content-constructor} 

An implementation of `Content` should have a constructor with signature `Content()`.

###### `getContentType` {#api-dom-content-getcontenttype}

A [`Content`](#api-dom-content) object must have a method with signature `ContentType getContentType()`.  

This method must return the `ContentType` value that correspondes to the type of the [`Content`](#api-dom-content) object.

###### `getBlock` {#api-dom-content-getblock}

A [`Content`](#api-dom-content) object must have a method with signature `Block getBlock()`.

This method must return the nearest [`Block`](#api-dom-block) object object the `Content` in the tree of Markdom nodes  the [`Content`](#api-dom-content) object is part of, which is the [`Block`](#api-dom-block) object of the `ContentParent` this [`Content`](#api-dom-content) object is currently attached to.

This method must fail if the [`Content`](#api-dom-content) object doesn't [have](#api-dom-node-hasparent) a [parent](#api-dom-node-getparent) [`ContentParent`](#api-dom-node) object.
   
##### `ContentParentBlock` {#api-dom-contentparentblock}

A [`ContentParentBlock`](#api-dom-contentparentblock) object is a [`Block`](#api-dom-block) object and a [`ContentParent`](#api-dom-contentparent) object that represents a [*Markdom block*](#domain-block) that contains [*Markdom content*](#domain-content).

###### Constructors {#api-dom-contentparentblock-constructor}

An implementation of `ContentParentBlock` should have a constructor with signature `ContentParentBlock()`.

For convenience, an implementation of `ContentParentBlock` should have a constructor with signature `ContentParentBlock(Content... contents)` that takes that takes an array of [`Content`](#api-dom-content) objects named `contents` and delegates to `ContentParent#addContents(Content... content)`.

##### `ContentParentContent` {#api-dom-contentparentcontent}

A [`ContentParentContent`](#api-dom-contentparentcontent) object is a [`Content`](#api-dom-content) object and a [`ContentParent`](#api-dom-contentparent) object that represents a [*Markdom content*](#domain-content) that contains [*Markdom content*](#domain-content).

###### Constructors {#api-dom-contentparentcontent-constructor}

An implementation of `ContentParent` should have a constructor with signature `ContentParentContent()`.

For convenience, an implementation of `ContentParentContent` should have a constructor with signature `ContentParentContent(Content... contents)` that takes that takes an array of [`Content`](#api-dom-content) objects named `contents` and delegates to `ContentParent#addContents(Content... content)`.

##### `ContentParent` {#api-dom-contentparent}

A [`ContentParent`](#api-dom-contentparent) object is a [`Node`](#api-dom-node) object that represents a [*Markdom node*](#domain-node) that contains [*Markdom contents*](#domain-content).

An implementation of `ContentParent` must have a final and initially empty `Sequence` of [`Content`](#api-dom-content) objects that is associated with the [`ContentParent`](#api-dom-contentparent) object.

Any structural modification (insert, remove, clear, replace) to the associated `Sequence` of [`Content`](#api-dom-content) objects must reflect the fact, that a [`Content`](#api-dom-content) object that is added to the associated `Sequence` object is attached to the [`ContentParent`](#api-dom-contentparent) object until is is removed from the associated `Sequence` of [`Content`](#api-dom-content) objects.

Attaching a [`Content`](#api-dom-content) object to the [`ContentParent`](#api-dom-contentparent) object  must fail
* if the [`Content`](#api-dom-content) object is not present, or  
* if the [`Content`](#api-dom-content) object is already attached to a [`ContentParent`](#api-dom-contentparent) object, or 
* if attaching the [`Content`](#api-dom-content) object to the [`ContentParent`](#api-dom-contentparent) object would create a cycle in the tree of Markdom nodes that the [`ContentParent`](#api-dom-contentparent) object is part of.

###### Constructors {api-dom-contentparent-constructor}

An implementation of `ContentParent` should have a constructor with signature `ContentParent()`.

For convenience, an implementation of `ContentParent` should have a constructor with signature `ContentParent(Content... contents)` that takes that takes an array of [`Content`](#api-dom-content) objects named `contents` and delegates to `addContents(Content... contents)`.

###### `getContents` {api-dom-contentparent-getcontents}

A [`Content`](#api-dom-content) object must have a method with signature `Sequence getContents()`.

This method must return the associated `Sequence` of [`Content`](#api-dom-content) objects.

###### `addContent` {api-dom-contentparent-addcontent}

For convenience, a [`ContentParent`](#api-dom-contentparent) object should have a method with signature `addContent(Content content)` that takes a [`Content`](#api-dom-content) object named `content`.

This method must add `content` at the end of the associated `Sequence` of [`Content`](#api-dom-content) objects. This attaches `content` to the [`ContentParent`](#api-dom-contentparent) object.

This method must fail if add `content` to the  associated `Sequence` of [`Content`](#api-dom-content) objects failed.

###### `addContents` {api-dom-contentparent-addcontents}

For convenience, a [`ContentParent`](#api-dom-contentparent) object should have a method with signature `addContents(Content... contents)` that takes an array of [`Content`](#api-dom-content) objects named `contents`.

This method must add all [`Content`](#api-dom-content) objects from `contents` in the given order at the end of the `Sequence` of [`Content`](#api-dom-content) objects of the [`ContentParent`](#api-dom-contentparent) object, as if `addContent(Content content)` has been called repeatedly for all [`Content`](#api-dom-content) objects from `contents`.  This attaches all [`Content`](#api-dom-content) objects from `contents` to the [`ContentParent`](#api-dom-contentparent) object.
  
This method must fail if `contents` is not present.
This method must fail if adding any [`Content`](#api-dom-content) object from `contents` to the  associated `Sequence` of [`Content`](#api-dom-content) objects failed.
  
Because this method is a short hand for repeated calls to `addContent(Content content)`, it must add all prior [`Content`](#api-dom-content) objects from `contents` to the  associated `Sequence` of [`Content`](#api-dom-content) objects, if it fails beacause of a violating [`Content`](#api-dom-content) object from `contents`.
  
##### `DivisionBlock` {#api-dom-divisionblock}

A [`DivisionBlock`](#api-dom-divisionblock) object is a [`Block`](#api-dom-block) object that represents a [*Markdom division block*](#domain-divisionblock)

###### Constructors {#api-dom-divisionblock-constructors}

An implementation of `DivisionBlock` should have a constructor with signature `DivisionBlock()` 
  
##### `Document` {#api-dom-document}

A [`Document`](#api-dom-document) object is a [`BlockParent`](#api-dom-blockparent) object that represents a [*Markdom document*](#domain-document).

###### Constructors {#api-dom-document-constructor}

For convenience, an implementation of `Document` should have a constructor with signature `Document(Block... blocks)` that takes that takes an array of [`Block`](#api-dom-block) objects and delegates to `BlockParent#addBlocks(Block... blocks)`.

##### `EmphasisContent` {#api-dom-emphasiscontent}

An [`EmphasisContent`](#api-dom-emphasiscontent) object is a [`ContentParentContent`](#api-dom-contentparentcontent) object that represents a [*Markdom emphasis content*](#domain-emphasiscontent).  

###### Constructors {#api-dom-emphasiscontent-constructor}

An implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent()` that sets the level of the [`EmphasisContent`](#api-dom-emphasiscontent) object to `LEVEL_1`.

For convenience, an implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent(EmphasisLevel level)` that takes an `EmphasisLevel` named `level` and delegates to `setLevel(EmphasisLevel level)`.

For convenience, an implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent(EmphasisLevel level, Content... contents)` that takes an `EmphasisLevel` named `level` and an array of [`Content`](#api-dom-content) objects named `contents` and delegates to `setLevel(EmphasisLevel level)` and `ContentParent#addContents(Content... contents)`.

###### `getLevel` {#api-dom-emphasiscontent-getlevel}

An [`EmphasisContent`](#api-dom-emphasiscontent) object must have a method with signature `EmphasisLevel getLevel()`.

This method must return the level of the [`EmphasisContent`](#api-dom-emphasiscontent) object.

###### `setLevel` {#api-dom-emphasiscontent-setlevel}

An [`EmphasisContent`](#api-dom-emphasiscontent) object must have a method with signature `setLevel(EmphasisLevel level)` that takes an `EmphasisLevel` named `level`.

This method must set the level of the [`EmphasisContent`](#api-dom-emphasiscontent) object. 
  
This method must fail if `level` is not present.
   
##### `HeadingBlock` {#api-dom-headingblock}

A [`HeadingBlock`](#api-dom-headingblock) object is a [`ContentParentBlock`](#api-dom-contentparentblock) object that represents a [*Markdom heading blocks*](#domain-headingblock).  

###### Constructors {#api-dom-headingblock-constructor}

An implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock()` that sets the level of the [`HeadingBlock`](#api-dom-headingblock) object to `LEVEL_1`.

For convenience, an implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock(HeadingLevel level)` that takes a `HeadingLevel` named `level` and delegates to `setLevel(HeadingLevel level)`.

For convenience, an implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock(HeadingLevel level, Content... contents)` that takes a `HeadingLevel` named `level` and an array of [`Content`](#api-dom-content) objects named `contents` and delegates to `setLevel(HeadingLevel level)` and `ContentParent#addContents(Contents... contents)`.

###### `getLevel` {#api-dom-headingblock-getlevel}

A [`HeadingBlock`](#api-dom-headingblock) object must have a method with signature `HeadingLevel getLevel()`.

This method must return the level of the [`HeadingBlock`](#api-dom-headingblock) object.

###### `setLevel` {#api-dom-headingblock-setlevel}

A [`HeadingBlock`](#api-dom-headingblock) object must have a method with signature `setLevel(HeadingLevel level)` that takes a `HeadingLevel` named `level`.

This method must set the level of the [`HeadingBlock`](#api-dom-headingblock) object. 
  
This method must fail if `level` is not present.
   
##### `ImageContent` {#api-dom-imagecontent}

An [`ImageContent`](#api-dom-imagecontent) object is a [`Content`](#api-dom-content) object that represents a [*Markdom image content*](#domain-imagecontent).  

###### Constructors {#api-dom-imagecontent-constructor}

An implementation of `LinkContent` should have a constructor with signature `LinkContent()` that sets the uri of the [`ImageContent`](#api-dom-imagecontent) object to the empty string and the title of the [`ImageContent`](#api-dom-imagecontent) object to be not present and the alternative text of the [`ImageContent`](#api-dom-imagecontent) object to be not present.

For convenience, an implementation of `ImageContent` should have a constructor with signature `ImageContent(String uri)` that takes a `String` named `uri` and delegates to `setUri(String uri)`.

For convenience, an implementation of `ImageContent` should have a constructor with signature `ImageContent(String uri, String? title, String? alternative)` that takes a `String` named `uri` and an optional `String` named `title` and an optional `String` named `alternative `and delegates to `setUri(String uri)` and `setTitle(String? title)` and `setAlternative(String? alternative)`.

###### `getUri` {#api-dom-imagecontent-geturi}

A [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `String getUri()`.

This method must return the uri of the [`ImageContent`](#api-dom-imagecontent) object.

###### `setUri` {#api-dom-imagecontent-seturi}

An [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `setUri(String uri)` that takes a `String` named `level`.

This method must set the uri of the [`ImageContent`](#api-dom-imagecontent) object. 
  
This method must fail if `uri` is not present.

###### `getTitle` {#api-dom-imagecontent-gettitle}

A [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `String getTitle()`.

This method must return the title of the [`ImageContent`](#api-dom-imagecontent) object.

###### `setTitle`{#api-dom-imagecontent-settitle}
  
An [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `setTitle(String? title)` that takes an optional `String` named `title`.

This method must set the title of the [`ImageContent`](#api-dom-imagecontent) object.

###### `getAlternative` {#api-dom-imagecontent-getalternative}

A [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `String getAlternative()`.

This method must return the alternative text of the [`ImageContent`](#api-dom-imagecontent) object. 

###### `setAlternative` {#api-dom-imagecontent-setalternative}

An [`ImageContent`](#api-dom-imagecontent) object must have a method with signature `setAlternative(String? alternative)` that takes an optional `String` named `alternative`.

This method must set the alternative text of the [`ImageContent`](#api-dom-imagecontent) object.

##### `LineBreakContent` {#api-dom-linebreakcontent}

A [`LineBreakContent`](#api-dom-linebreakcontent) object is a [`Content`](#api-dom-content) object that represents a [*Markdom line break content*](#domain-linebreakcontent).  

###### Constructors {#api-dom-linebreakcontent-constructor}

An implementation of `LineBreakContent` should have a constructor with signature `LineBreakContent()` that sets the [`LineBreakContent`](#api-dom-linebreakcontent) object to represent a soft line break.

For convenience, an implementation of `LineBreakContent` should have a constructor with signature `LinkContent(Boolean hard)` that takes a `Boolean` named `hard` and delegates to `setHard(Boolean hard)`.

###### `isHard` {#api-dom-linebreakcontent-ishard}

A [`LineBreakContent`](#api-dom-linebreakcontent) object must have a method with signature `Boolean isHard()`.

This method must return whether the [`LinkContent`](#api-dom-linkcontent) object represents a hard line break or a soft line break.

###### `setHard` {#api-dom-linebreakcontent-sethard}

A [`LineBreakContent`](#api-dom-linebreakcontent) object must have a method with signature `setHard(Boolean hard)` that takes a `Boolean` named `hard`.

This method must set whether the [`LinkContent`](#api-dom-linkcontent) object represents a hard line break or a soft line break.
  
This method must fail if `hard` is not present.
   
##### `LinkContent` {#api-dom-linkcontent}

A [`LinkContent`](#api-dom-linkcontent) object is a [`ContentParentContent`](#api-dom-contentparentcontent) object that represents a [*Markdom link content*](#domain-linkcontent).

###### Constructors {#api-dom-linkcontent-constructor}

An implementation of `LinkContent` should have a constructor with signature `LinkContent()` that sets the uri of the [`LinkContent`](#api-dom-linkcontent) object to the empty string.

For convenience, an implementation of `LinkContent` should have a constructor with signature `LinkContent(String uri)` that takes a `String` named `uri` and delegates to `setUri(String uri)`.

For convenience, an implementation of `LinkContent` should have a constructor with signature `LinkContent(String uri, Content... contents)` that takes a `String` named `uri` and an array of [`Content`](#api-dom-content) objects named `contents` and delegates to `setUri(String uri)` and `ContentParent#addContents(Content... contents)`.

###### `getUri` {#api-dom-imagecontent-geturi}

A [`LinkContent`](#api-dom-linkcontent) object must have a method with signature `String getUri()`.

This method must return the uri of the [`LinkContent`](#api-dom-linkcontent) object.

###### `setUri` {#api-dom-linkcontent-seturi}

A [`LinkContent`](#api-dom-linkcontent) object must have a method with signature `setUri(String uri)` that takes a `String` named `level`.

This method must set the uri of the [`LinkContent`](#api-dom-linkcontent) object. 
  
This method must fail if `uri` is not present.
   
##### `ListBlock` {#api-dom-listblock}

A [`ListBlock`](#api-dom-listblock) object is a [`Block`](#api-dom-block) object that represents a [*Markdom list block*](#domain-listblock)  

An implementation of `ListBlock` must have a final and initially empty `Sequence` of [`ListItem`](#api-dom-listitem) objects that is associated with the [`ListBlock`](#api-dom-listblock) object.

Any structural modification (insert, remove, clear, replace) to the associated `Sequence` of [`ListItem`](#api-dom-listitem) objects must reflect the fact, that a [`ListItem`](#api-dom-listitem) object that is added to the associated `Sequence` object is attached to the [`ListBlock`](#api-dom-listblock) object until is is removed from the associated `Sequence` of [`ListItem`](#api-dom-listitem) objects.

Attaching a [`ListItem`](#api-dom-listitem) object to the [`ListBlock`](#api-dom-listblock) object  must fail
* if the [`ListItem`](#api-dom-listitem) object is not present, or  
* if the [`ListItem`](#api-dom-listitem) object is already attached to a [`ListBlock`](#api-dom-listblock) object, or 
* if attaching the [`ListItem`](#api-dom-listitem) object to the [`ListBlock`](#api-dom-listblock) object would create a cycle in the tree of Markdom nodes that the [`ListBlock`](#api-dom-listblock) object is part of.

###### Constructors {api-dom-listblock-constructor}

An implementation of `ListBlock` should have a constructor with signature `ListBlock()`.

For convenience, an implementation of `ListBlock` should have a constructor with signature `ListBlock(ListItem... items)` that takes that takes an array of [`ListItem`](#api-dom-listitem) objects named `items` and delegates to `addItems(ListItem... items)`.

###### `getListItems` {api-dom-listblock-getitems}

A [`ListItem`](#api-dom-listitem) object must have a method with signature `Sequence getListItems()`.

This method must return the associated `Sequence` of [`ListItem`](#api-dom-listitem) objects.

###### `addItem` {api-dom-listblock-additem}

For convenience, a [`ListBlock`](#api-dom-listblock) object should have a method with signature `addItem(ListItem item)` that takes a [`ListItem`](#api-dom-listitem) object named `item`.

This method must add `item` at the end of the associated `Sequence` of [`ListItem`](#api-dom-listitem) objects. This attaches `item` to the [`ListBlock`](#api-dom-listblock) object.

This method must fail if add `item` to the  associated `Sequence` of [`ListItem`](#api-dom-listitem) objects failed.

###### `addItems` {api-dom-listblock-additems}

For convenience, a [`ListBlock`](#api-dom-listblock) object should have a method with signature `addItems(ListItem... items)` that takes an array of [`ListItem`](#api-dom-listitem) objects named `items`.

This method must add all [`ListItem`](#api-dom-listitem) objects from `items` in the given order at the end of the `Sequence` of [`ListItem`](#api-dom-listitem) objects of the [`ListBlock`](#api-dom-listblock) object, as if `addItem(ListItem item)` has been called repeatedly for all [`ListItem`](#api-dom-listitem) objects from `items`.  This attaches all [`ListItem`](#api-dom-listitem) objects from `items` to the [`ListBlock`](#api-dom-listblock) object.
  
This method must fail if `items` is not present.
This method must fail if adding any [`ListItem`](#api-dom-listitem) object from `items` to the  associated `Sequence` of [`ListItem`](#api-dom-listitem) objects failed.
  
Because this method is a short hand for repeated calls to `addItem(ListItem item)`, it must add all prior [`ListItem`](#api-dom-listitem) objects from `items` to the  associated `Sequence` of [`ListItem`](#api-dom-listitem) objects, if it fails beacause of a violating [`ListItem`](#api-dom-listitem) object from `items`.

##### `ListItem` {#api-dom-listitem}

A [`ListItem`](#api-dom-listitem) object is a [`BlockParent`](#api-dom-blockparent) object that represents a [*Markdom list item*](#domain-listitem).

###### Constructors {#api-dom-listitem- constructor}

An implementation of `ListItem` should have a constructor with signature `ListItem()`.

For convenience, an implementation of `ListItem` should have a constructor with signature `ListItem(Block... blocks)` that takes an array of [`Block`](#api-dom-block) objects named `blocks` and delegates to `BlockParent#addBlocks(Block... blocks)`.

##### `Node` {#api-dom-node}

A [`Node`](#api-dom-node) object represents a [*Markdom node*](#domain-node).

###### `getNodeType` {#api-dom-node-getnodetype}

A [`Node`](#api-dom-node) object must have a method with signature `NodeType getNodeType()`.  

This method must return the [`NodeType`](#api-dom-nodetype) value that correspondes to the type of the [`Node`](#api-dom-node) object.

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

For convenience, an implementation of `OrderedListBlock` should have a constructor with signature `OrderedListBlock(Integer startIndex)` that takes an `Integer` named `startIndex` and delegates to `setStartIndex(Integer startIndex)`.

For convenience, an implementation of `OrderedListBlock` should have a constructor with signature `OrderedListBlock(Integer startIndex, ListItem... items)` that takes an `Integer` named `startIndex` and an array of [`ListItem`](#api-dom-listitem) objects named `items` and delegates to `setStartINdex(Integer startIndex)` and `ContentParent#addContents(Content... contents)`.

###### getStartIndex {#api-dom-orderedlistblock-getstartindex}

An [`OrderedListBlock`](#api-dom-orderedlistblock) object must have a method with signature `Integer getStratIndex()`.

This method must return the start index of the [`OrderedListBlock`](#api-dom-orderedlistblock) object.

###### setStartIndex {#api-dom-orderedlistblock-setstartindex}

An [`OrderedListBlock`](#api-dom-orderedlistblock) object must have a method with signature `setStartIndex(Integer startIndex)` that takes an `INteger` named `startIndex`.

This method must set the start index of the [`OrderedListBlock`](#api-dom-orderedlistblock) object. 
  
This method must fail if `startIndex` is not present.  
This method must fail if `startIndex` is negative.
  
##### `ParagraphBlock` {#api-dom-paragraphblock}

A [`ParagraphBlock`](#api-dom-paragraphblock) object is a [`ContentParentBlock`](#api-dom-contentparentblock) object that represents a [*Markdom paragraph block*](#domain-paragraphblock).

###### Constructors {#api-dom-paragraphblock-constructor}

An implementation of `ParagraphBlock` should have a constructor with signature `ParagraphBlock()`.

For convenience, an implementation of `ParagraphBlock` should have a constructor with signature `ParagraphBlock(Content... contents)` that takes an array of [`Content`](#api-dom-content) objects named `contents` and delegates to `ContentParentBlock#addContents(Contents... contents)`.

##### `QuoteBlock` {#api-dom-quoteblock}

A [`QuoteBlock`](#api-dom-quoteblock) object is a [`Block`](#api-dom-block) object and a [`BlockParent`](#api-dom-blockparent) object that represents a [*Markdom quote block*](#domain-quoteblock).

###### Constructors {#api-dom-quoteblock-paragraphblock}

An implementation of `QuoteBlock` should have a constructor with signature `QuoteBlock()`.

For convenience, an implementation of `QuoteBlock` should have a constructor with signature `QuoteBlock(Block... blocks)` that takes an array of [`Block`](#api-dom-block) objects named `blocks` and delegates to `BlockParent#addBlocks(Block... blocks)`.
    
##### `TextContent` {#api-dom-textcontent}

A [`TextContent`](#api-dom-textcontent) object is [`ContentParentContent`](#api-dom-contentparentcontent) object that represents a [*Markdom text content*](#domain-textcontent).

###### Constructors {#api-dom-textcontent-constructor}

An implementation of `TextContent` should have a constructor with signature `TextContent()` that sets the text of the [`TextContent`](#api-dom-textcontent) object to the empty string.
   
For convenience, an implementation of `TextContent` should have a constructor with signature `TextContent(String text)` that takes a `String` named `text` and delegates to `setText(String text)`.

###### `getText` {#api-dom-textcontent-gettext}

A [`TextContent`](#api-dom-textcontent) object must have a method with signature `String getText()`.

This method must return the text of the [`TextContent`](#api-dom-textcontent) object.

###### `setText` {#api-dom-textcontent-settext}

A [`TextContent`](#api-dom-textcontent) object must have a method with signature `setText(String text)` that takes a `String` named `text`.

This method must set the text of the [`TextContent`](#api-dom-textcontent) object. 
  
This method must fail if `text` is not present.
   
##### `UnorderedListBlock` {#api-dom-unorderedlistblock}

An [`UnorderedListBlock`](#api-dom-unorderedlistblock) object is a [`ListBlock`](#api-dom-listblock) object that represents an [*unordered Markdom list block*](#domain-unorderedlistblock).

###### Constructors {#api-dom-unorderedlistblock-constructor}

An implementation of `UnorderedListBlock` should have a constructor with signature `UnorderedListBlock()`.
   
For convenience, an implementation of `UnorderedListBlock` should have a constructor with signature `UnorderedListBlock(ListItem... items)` that takes an array of [`ListItem`](#api-dom-listitem) objects named `items` and delegates to  `ListBlock#addItems(ListItem... items)`.

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

![](resource/markdom-events.png)

### Combining the Domain Object API and the Handler API

// document is dispatcher, 

// markdom document markdom handler

## Data representations

### JSON

### YAML

### XML