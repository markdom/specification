![](resource/markdom-mark-positive.png)

---

# Markdom {#markdom}

Markdom is lightweight specification for simple rich text. Markdom is not a markup language like Markdown or it's many dialects itself. Markdom is a set of definitions that aims to standardize the handling, processing and serialization of simple rich text across software and language boundaries. Markdom tries to fill a gap left open by existing technologies.

The three main intents of Markdom are

1. to define the lowest common denominator for simple rich text,
1. to allow easy translation of algorithms working working with simple rich text from one programming language to another (by providing a standardized API) and
1. to simplify the transfer of simple rich text between different software components (by specifying the serialization in common data transfer formats like JSON and XML).

The name Markdom is a composition of [**mark**up](https://en.wikipedia.org/wiki/Markup_language) and [**do**main **m**odel](https://en.wikipedia.org/wiki/Domain_model).

## Overview {#overview}

A piece of simple rich text is called a Markdom document. This specification covers multiple aspects related to Markdom documents:

* The [domain section](#domain) covers the general structure and the supported formatting instructions of Markdom documents.
* The [API section](#api) covers programming interfaces to programmatically create, modify and process Markdom documents.
* The [data section](#data) covers the representation of Markdom documents in common data transfer formats.
* The [markup section](#markup) covers the representation of Markdom documents in common markup languages.

## Why Markdom? {#why}

Markdom was created to be an answer to a simple, but as yet unanswered [question](https://stackoverflow.com/questions/34955835/cross-device-rich-text-formatting):

1. Allow a human to create simple rich text in a human friendly manner (e.g. by writing Markdown text in a website).
1. Transmit the simple rich text to different platforms (e.g. from the website to a web server and from the web server to a native smartphone app).
1. Display the simple rich text on different platforms in a native way (e.g. by rendering it in a native smartphone app).

The latter includes at least Android apps (using [Spanned Strings](https://developer.android.com/reference/android/text/Spanned.html)), iOS apps (using [Attributed Strings](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSAttributedString_Class/)) and websites (using HTML).

![](resource/markdom-overview.png)

### Why not HTML? {#why-nothtml}

Allowing humans to enter some HTML with a suitable [WYSISYG editor](https://www.tinymce.com/) or as raw HTML (for *experts*) is a commonly used approach in content management systems, blogs and other websites. This allows the user the enter everything from simple unformatted text to arbitrarily complex formatted text. This is all well and good, but if all that's asked for is to allow a user to enter text with some links, enumerations and and the occasional emphasis, HTML is way to powerful.

Allowing HTML as user input restricts the usability of the content to places that can handle HTML (e.g. websites or web views). Using the content in a text view of a native smartphone app or to generate a PDF file might be [impossible](https://leanpub.com/markua/read#no-inline-html).

Allowing HTML as user input might also cause security issues (e.g. embedded JavaScript or rogue `form`-elements), mess up the design of a website (embedded CSS, rogue `<marquee>`-elements), reduce the usability of a website (`style="color:#ff00ff;"`) or mess with the SEO of a website (rogue `table`-elements, misplaced semantic elements).

### Why not Markdown? {#why-notmarkdown}

Markdown is introduced with the following [description](https://daringfireball.net/projects/markdown/):

> Markdown is a text-to-HTML conversion tool for web writers. Markdown allows you to write using an easy-to-read, easy-to-write plain text format, then convert it to structurally valid XHTML (or HTML).

The intended limitation to be nothing more than a text-to-HTML conversion tool is recognizable in almost all Markdown implementations. Only very few Markdown implementations offer conversion to something that is not HTML. Almost no Markdown implementation allows conversion to a domain model that is open to programmatic introspection or manipulation.

Since Markdown was only released as an implementation and without a formal specification, different implementations [disagree](http://johnmacfarlane.net/babelmark2/?text=%3E+*+A+quote%0A%3E+Another+quote) over the meaning of edge cases like the following snippet:

```
> * A quote
> Another quote
```

Additionally, many Markdown implementations use custom flavors of the Markdown syntax or offer custom extensions.

As a result, it is [almost](https://uncodin.github.io/bypass/) impossible to find a set of markdown implementations that generate Spanned Strings for Android, Attributed Strings for iOS and HTML for websites, that interpret a given Markdown text identically and generate equal outputs.

### Why not CommonMark? {#why-notcommonmark}

[CommonMark](http://commonmark.org/) is a specification for Markdown text and compatible CommonMark implementations exist for [different languages](https://github.com/jgm/CommonMark/wiki/List-of-CommonMark-Implementations).

This solved one major problem with Markdown, but not the others:

* CommonMark allows the integration of arbitrary HTML with all the above mentioned problems.
* CommonMark implementations are often nothing more than text-to-HTML converters.
* CommonMark implementations tend to offer custom extensions which defeats the purpose of standardization.

### So, why Markdom? {#why-markdom}

Markdom tries to overcome the mentioned shortcomings of HTML and markup languages like Markdown and CommonMark by introducing a  standard for simple rich text that is unambiguous and easy to process.

The set of possible formatting instructions has been chosen with the intent, that the largest possible number of applications can produce a reasonable output. This includes the lack of arbitrarily configurable content like HTML. If the original input includes such content, it is the responsibility of the authoritative component to reject the input or to process the input and reduce it to appropriate formatting instructions.

### Markdom Use Cases {#why-usecases}

A typical system  using Markdom may encompass the following use cases:

1. A user creates content:

    * The server allows a user to enter CommonMark text.
    * The server parses and interprets the CommonMark text with a suitable libraryand generates a Markdom document.
    * The server generates a XML representation of the Markdom document and stores is for later use.

1. The user returns to edit the content:

    * The server loads the previously stored XML representation and creates a Markdom document.
    * The server creates CommonMark text of the Markdom document and presents it to the user.

1. Another user visits the website to view the content:

    * The server loads the previously stored XML representation and creates a Markdom document.
    * The server creates a HTML representation of the Markdom document and delivers it to the visitor.

1. Another user uses a smartphone app to view the content:

    * The server loads the previously stored XML representation and creates a Markdom document.
    * The server creates a JSON representation of the Markdom document and delivers it to the app.
    * The app receives the JSON representation and creates a Markdom document.
    * The app creates a native representation of the Markdom document and displays it.

Only the server has to handle the content as CommonMark text. The app receives an unambiguous and easy to parse JSON representation. Allowing additional types of input (e.g. Textile) only requires changing the server.

Other use cases might require the server to interpret uploaded HTML or DOC files to create initial versions of a stored Markdom document, to generate a PDF representation of a stored Markdom document or to generate Markdom documents on the fly with information fetched from a database and a suitable templating mechanism.

## Domain {#domain}

A *Markdom document* represents the entirety of a rich text document, including the actual text content and formatting instructions. A *Markdom document* contains of a sequence of *Markdom blocks*.

A *Markdom block* represents a portion of a rich text document that serves a specific purpose. Each type of *Markdom block* has the necessary properties that describe it's content. Some *Markdom blocks*, e.g. a *Markdom block* representing a block quote, contain further *Markdom blocks*. Other *Markdom blocks*, e.g. a *Markdom block* representing a paragraph, contain *Markdom content*.

A *Markdom content* represents a portion of rich text that serves a specific purpose. Each type of *Markdom content* has the necessary properties that describe it's content. Some *Markdom contents*, e.g. a *Markdom content* representing a link, contain further *Markdom content*. A *Markdom content* never contains a *Markdom block*.

Put another way, a *Markdom document* is represented as a tree of *Markdom nodes*. Starting with a root node that represents an entire rich text document, each *Markdom node* represents a portion of the corresponding *Markdom document*.

### Nodes {#domain-node}

A [*Markdom node*] is either a

* [*Markdom document*], or a
* [*Markdom block*], or a
* [*Markdom list item*], or a
* [*Markdom content*].

#### Document {#domain-document}

A [*Markdom document*] is a [*Markdom node*] that represents the entirety of a rich text document. A [*Markdom document*] is the root node of a tree of [*Markdom nodes*].

A [*Markdom document*] contains a sequence of [*Markdom blocks*] named `blocks`.

The `blocks` sequence shouldn't be empty.

#### Block {#domain-block}

A [*Markdom block*] is a [*Markdom node*] that represents a portion of a rich text document.

A [*Markdom block*] is either a

* [*Markdom code block*], or a
* [*Markdom comment block*], or a
* [*Markdom division block*], or a
* [*Markdom heading block*], or an
* [*ordered Markdom list block*], or a
* [*Markdom paragraph block*], or a
* [*Markdom quote block*], or an
* [*unordered Markdom list block*].

##### Code Block {#domain-codeblock}

A [*Markdom code block*] is a [*Markdom block*] that represents a portion of plain text (e.g. source code) that may be augmented with meaningful syntax highlighting. Implementations that generate output that is displayed to humans should display the text as preformatted text in a monospaced font.

A [*Markdom code block*] has a mandatory string parameter named `code` and an optional string parameter named `hint`.

Values of the `code` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters other then `LINE_FEED` (`\n`) or `CHARACTER_TABULATION` (`\t`). Values of the `code` parameter shouldn't be empty.

Values of the `hint` parameter, if present, should be the common name of a programming or markup language in lowercase (i.e `java`, `php`, `json`, `html`, ...) and are intended to be used by implementations to augment the preformatted text with meaningful syntax highlighting. Values of the `code` parameter, if present, shouldn't be empty.

##### Comment Block {#domain-commentblock}

A [*Markdom comment block*] is a [*Markdom block*] that represents a portion of plain text that is not meant to be displayed. Implementations that generate output that is displayed to humans should ignore the text.

A [*Markdom comment block*] has a mandatory string parameter named `comment`.

Values of the `comment` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters other then `LINE_FEED` (`\n`) or `CHARACTER_TABULATION` (`\t`). Values of the `comment` parameter shouldn't be empty.

##### Division Block {#domain-divisionblock}

A [*Markdom division block*] is a [*Markdom block*] that represents a division or thematic break. Implementations that generate output that is displayed to humans should display a horizontal rule.

##### Heading Block {#domain-headingblock}

A [*Markdom heading block*] is a [*Markdom block*] that represents a headline. Implementations that generate output that is displayed to humans should display the headline text as an appropriately emphasized and separated paragraph.

A [*Markdom heading block*] has a mandatory integer parameter named `level` and a sequence of [*Markdom contents*] named `contents`.

Valid values of the `level` parameter are 1 to 6, where a lower value indicates a higher precedence of the headline.

The `contents` sequence shouldn't be empty.

##### Ordered List Block {#domain-orderedlistblock}

An [*ordered Markdom list block*] is a [*Markdom block*] that represents an ordered list (enumeration) of list items. Each list item contains further portions of the [*Markdom document*]. Implementations that generate output that is displayed to humans should indent all list items and display the index of each list item in front of it.

An [*ordered Markdom list block*] has a mandatory integer parameter `startIndex` and a sequence of [*Markdom list items*] named `items`.

Valid values of the `startIndex` parameter are positive integers. The value of the `startIndex` parameter is the index of the first item in the `items` sequence. Following items in the `items` sequence have successive indices.

The `items` sequence shouldn't be empty.

##### Paragraph Block {#domain-paragraphblock}

A [*Markdom paragraph block*] is a *markdom block* that represents a paragraph. Implementations that generate output that is displayed to humans should display the paragraph text as an appropriately separated paragraph.

A [*Markdom paragraph block*] has a sequence of [*Markdom contents*] named `contents`.

The `contents` sequence shouldn't be empty.

##### Quote Block {#domain-quoteblock}

A [*Markdom quote block*] is a [*Markdom block*] that represents a block quote. The quote contains further portions of the [*Markdom document*]. Implementations that generate output that is displayed to humans should emphasize (e.g. a vertical line, indentation, slanted font) the quote appropriately.

A [*Markdom quote block*] has a sequence of [*Markdom blocks*] named `blocks`.

The `blocks` sequence shouldn't be empty.

##### Unordered List Block {#domain-unorderedlistblock}

An *unordered Markdom list block* is a [*Markdom block*] that represents an unordered list (bullet list) of list items. Each list item contains further portions of the [*Markdom document*]. Implementations that generate output that is displayed to humans should indent all list items and display an appropriate symbol (e.g. a bullet) in front of each item.

An [*unordered Markdom list block*] has a sequence of [*Markdom list items*] named `items`.

The `items` sequence shouldn't be empty.

#### List Item {#domain-listitem}

A [*Markdom list item*] represents a list item. The list item contains further portions of the [*Markdom document*]. Implementations that generate output that is displayed to humans should display the list item according to the corresponding An [*ordered Markdom list block*]  or An [*unordered Markdom list block*] .

A [*Markdom list item*] has a sequence of [*Markdom blocks*] named `blocks`.

The `blocks` sequence shouldn't be empty.

#### Content {#domain-content}

A [*Markdom content*] represents a portion of rich text that makes up a [*Markdom paragraph block*] or a [*Markdom heading block*].

A [*Markdom content*] is either a

* [*Markdom code content*], or an
* [*Markdom emphasis content*], or an
* [*Markdom image content*], or an
* [*Markdom line break content*], or a
* [*Markdom link content*], or a
* [*Markdom text content*].

##### Code Content {#domain-codecontent}

A [*Markdom code content*] is a [*Markdom content*] that represents a portion of plain text (e.g. source code). Implementations that generate output that is displayed to humans should display the text as preformatted text in a monospaced font.

A [*Markdom code content*] has a mandatory string parameter named `code`.

Values of the `code` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters. Values of the `code` parameter shouldn't be empty.

##### Emphasis Content {#domain-emphasiscontent}

An [*Markdom emphasis content*] is a [*Markdom content*] that represents emphasized text. Implementations that generate output that is displayed to humans should display the text appropriately emphasized (e.g. italic or bold).

A [*Markdom emphasis content*] has a mandatory integer parameter named `level` and a sequence of [*Markdom contents*] named `contents`.

Valid values of the `level` parameter are 1 and 2, where a higher value indicates a higher importance of the emphasized text.

The `contents` sequence shouldn't be empty.

##### Image Content {#domain-imagecontent}

An [*Markdom image content*] is a [*Markdom content*] that represents an image, including a title text and an alternative text. Implementations that generate output that is displayed to humans should display the linked image with the title text or, if the linked image couldn't be resolved, the alternative text as if the [*Markdom image content*] was a [*Markdom text content*].

A [*Markdom link content*] has a mandatory string parameter named `uri` and an optional string parameter named `title` and an optional string parameter named `alternative`.

Valid values of the `uri` parameter are valid [URI references](https://tools.ietf.org/html/rfc3986#section-4.1). Values of the `uri` parameter should link to an image resource (e.g. a JPEG or PNG file). Actual applications that process [*Markdom documents*] may impose further constraints.

Values of the `title` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters.  Values of the `title` parameter, if present, shouldn't be empty.

Values of the `alternative` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters. Values of the `alternative` parameter, if present, shouldn't be empty.

##### Line Break Content {#domain-linebreakcontent}

A [*Markdom line break content*] is a [*Markdom content*] that represents a line break. A line break is either a *hard line break* or a *soft line break*. Implementations that generate output that is displayed to humans should display only hard line breaks. Implementations that generate output that is processed as source code, e.g. as Markdown text, should also display soft line breaks in a way that doesn't introduce line breaks during further processing of the generated source code.

A [*Markdom line break content*] has a mandatory boolean parameter named `hard`.

##### Link Content {#domain-linkcontent}

A [*Markdom link content*] is a [*Markdom content*] that represents text with a link, including a title. Implementations that generate output that is displayed to humans should display the text appropriately emphasized (e.g. underlined or in a different color) and provide an appropriate form of interaction that allows the human to open the link.

A [*Markdom link content*] has a mandatory string parameter named `uri` and an optional string parameter named `title` and a sequence of [*Markdom contents*] named `contents`.

Valid values of the `uri` parameter are valid [URI references](https://tools.ietf.org/html/rfc3986#section-4.1). Actual applications that process [*Markdom documents*] may impose further constraints.

Values of the `title` parameter should not contain any [control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters.

The `contents` sequence shouldn't be empty. A [*Markdom content*] in the `contents` sequence must not be another [*Markdom link content*] or, recursively, contain another [*Markdom link content*].

##### Text Content {#domain-textcontent}

A [*Markdom text content*] is a [*Markdom content*] that represents a portion of text. Implementations that generate output that is displayed to humans should display the text as text in an appropriate font.

A [*Markdom text content*] has a mandatory string parameter named `text`.

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

The rich text document consists of a heading, an ordered list with three list items and some plain text. The first list item of the ordered list contains a link. The second list item of the unordered list contains text, a hard line break and some plain text. The third list item of the ordered list contains a quoted emphasized text.

The following image shows a tree of [*Markdom nodes*], i.e. a [*Markdom document*], that represents the same rich text document:

![](resource/markdom-example.png)

The same [*Markdom document*] can be, for instance, represented as an [object graph](#api-dom-example) or a [succession of events](#api-handler-example), using a data exchange format like [JSON](#data-json-example) or [XML](#data-xml-example) or using a markup language like [HTML document](#markup-html-example).

## APIs {#api}

This specification describes two distinct APIs:

* The [Domain Model API](#api-dom) contains a set of interfaces that describe how a [*Markdom document*] is represented as an object graph and methods to compose and consume such an object graph.

   The Domain Model API allows to create a representation of a [*Markdom document*] in memory, which can then be examined, modified and further processed.

* The [Handler API](#api-dom) contains an interface that describes how a [*Markdom document*] is represented as a succession of events.

  The Handler API allows to process a [*Markdom document*] on the fly without the necessity to create an object graph. Events that describe a [*Markdom document*] might be dispatched by an object graph from the Domain Model API or by a specific event dispatcher implementation that process a [data representation](#data) or a [markup representation](#markup) of a [*Markdom document*].

This specifications primarily covers the interfaces and enumerations for both APIs. A concrete implementation of this specification for a given programming language should consist of corresponding interface definitions as well as an concrete implementation of the Domain Model API interfaces and some commonly useful concrete implementations of the Handler API.

Depending on the programming language, it might be sensible to divide the interfaces, enumerations and concrete implementations into multiple packages.

* A *Common* package that contains the common enumerations and a base exception for all Markdom related operations.
* A *Handler* package that contains the interfaces and commonly useful concrete implementations of the Handler API.
* A *Model* package that depends contains the interfaces and enumerations of the Domain Model API. This allows for different concrete implementations of the Domain Model API.
* A *Model* reference implementation package that contains a concrete implementation of the Domain Model API.
* Several *Handler* implementation packages for different tasks some of which may depend on the *Model* package.

![](resource/markdom-packages.png)

It is commonly recommended to implement an algorithms that processes a [*Markdom document*] as a [`Handler`] object rather than a method that directly processes a [`Document`]. This allows to the algorithm to be used in different scenarios and, potentially, without the need do construct a [*Markdom document*] in memory. The Domain Model API should generally only be used if it is necessary to temporarily store a [*Markdom document*] in memory or to programmatically modify a [*Markdom document*] before it is processed with a [`Handler`].

### Common {#api-common}

#### Considerations {#api-common-considerations}

It is safe to assume that some of the requirements from this specification don't go along well with established conventions or formal rules of a given programming language. This includes, but is not limited to naming conventions (e.g. prefixes and suffixes for interfaces, traits, abstract classes and methods).

Implementers should find a reasonable compromise between the two. In doubt, programming language idiosyncrasies should be preferred over requirements from this specification, as long as the general spirit of this specification is preserved (e.g. adding or removing prefixes or suffixes is okay, but unnecessary changes to names are not).

The return type of methods is therefor omitted where possible. Implementers should select a return type that best fits the spirit of the programming language.

This should ensure that the implementation is acceptable to programmers familiar with that programming language and that implementations (e.g. a particular [`Handler`] object) are easily translatable into other programming languages.

It is recommended that implementations contain a section about such implementation considerations in their `README` file.

#### Dependencies {#api-common-dependencies}

Both Markdom APIs require support from the particular programming language a concrete implementation is written in. This specification is written with an object orientated programming language in mind, but it should be little effort to interpret this specification for a programming languages that follows another programming paradigm.

##### Mandatory and optional {#api-common-optional}

The Markdom APIs use mandatory and optional parameters. Mandatory parameters must always have a value. Optional parameters may or may not have a value. Mandatory parameters are hereafter notes as `Type name` where `Type` is a value type and `name` is the parameter name. Optional parameters are hereafter notes as `Type? name` where `Type` is a value type and `name` is the parameter name.

There are common implementations for mandatory and optional parameters.

1. For a programming languages that has `null` values and can't enforce non-`null` values for a method parameter, `null` may be used to indicate that an optional parameter has no value. Explicit `null`-checks should be implemented for mandatory parameters.
1. For a Programming languages that has `null` values and can enforce non-`null` values for a method parameter, `null` may be used to indicate that an optional parameter has no value. No explicit `null`-checks must be implemented for mandatory parameters.
1. For a Programming languages that has `null` values an explicit `Optional` type may be used to for an optional parameter. Explicit `null`-checks should be implemented for all parameters.
1. For a Programming languages that doesn't have `null` values an explicit `Optional` type must be used to for an optional parameter. No explicit `null`-checks must be implemented for mandatory parameters.


##### Values {#api-common-values}

The Markdom APIs use parameters and return values that represent values of simple types. Specifically for boolean values, numbers, Unicode character sequences and [URI references](https://tools.ietf.org/html/rfc3986#section-4.1). Such types are hereafter noted as `Boolean`, `Integer`, `String` and `Uri` respectively.

This should usually be implemented with appropriate primitive or value types.

##### Iterables {#api-common-iterable}

The Markdom APIs use parameters and return values that represent a linear ordered concatenation of values of a given type that can be iterated over in that order. Such types are hereafter noted as `Type...` where `Type` is the value type.

This should usually be implemented as a data structure that is eligible to be processed in a `foreach`-loop.

As a method parameter, this may also be implemented as a variable length argument list.

##### Sequences {#api-common-sequence}

The Markdom APIs use parameters and return values that represent a modifiable linear ordered concatenation of values of values of a given type that allows random access. Such types are hereafter noted as `[Type]` where `Type` is the value type.

This should usually be implemented as a data structure that has at least the following (or comparable) methods:

* `Boolean isEmpty()`
* `Integer getSize()`
* `Boolean contains(Object object)`
* `Integer indexOf(Object object)`
* `Object get(int index)`
* `set(int index, Object object)`
* `add(Object object)`
* `add(int index, Object... objects)`
* `addAll(Object object)`
* `addAll(int index, Object.. objects)`
* `remove(Object object)`
* `removeAll(Object... objects)`
* `clear()`has

#### Enumerations {#api-common-enumerations}

Markdom APIs have the following enumerations:

##### `BlockType` {#api-common-blocktype}

The `BlockType` enum indicates the type of a [*Markdom block*] and has the following constants:

* `CODE`,
* `COMMENT`
* `DIVISION`,
* `HEADING`,
* `ORDERED_LIST`,
* `PARAGRAPH`,
* `QUOTE`,
* `UNORDERED_LIST`.

##### `ContentType` {#api-common-contenttype}

The `ContentType` enum indicates the type of a [*Markdom content*] and has the following constants:

* `CODE`,
* `EMPHASIS`,
* `IMAGE`,
* `LINE_BERAK`,
* `LINK`,
* `TEXT`.

##### `EmphasisLevel` {#api-common-emphasislevel}

The `EmphasisLevel` enum indicates the level of a [*Markdom emphasis content*] object and has the following constants:

* `LEVEL_1`,
* `LEVEL_2`.

##### `HeadingLevel` {#api-common-headinglevel}

The `HeadingLevel` enum indicates the level of a [*Markdom heading block*] object and has the following constants:

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

A `Block` object is a [`Node`] object that represents a [*Markdom block*].

###### Constructors {#api-dom-block-constructor}

An implementation of `Block` should have a constructor with signature `Block()`.

###### `getBlockType` {#api-dom-block-getblocktype}

A `Block` object must have a method with signature `BlockType getBlockType()`.

This method must return the [`BlockType`] value that corresponds to the type of the represented [*Markdom block*].

##### `BlockParent` {#api-dom-blockparent}

A `BlockParent` object is a [`Node`] object that represents a [*Markdom node*] that contains [*Markdom blocks*].

An implementation of `BlockParent` must have a final and initially empty companion [`Sequence`] object for the [`Block`] objects that are associated with the `BlockParent` object.

Any structural modification (insert, remove, clear, replace) to the companion [`Sequence`] object must reflect the fact, that a [`Block`] object that is added to the companion [`Sequence`] object is attached to the `BlockParent` object until is is removed from the companion [`Sequence`] object.

Attaching a [`Block`] object to the `BlockParent` object  must fail
* if the [`Block`] object is not present, or
* if the [`Block`] object is already attached to a `BlockParent` object, or
* if attaching the [`Block`] object to the `BlockParent` object would create a [cycle](#api-dom-detecting-cycles) in the tree of Markdom nodes that the `BlockParent` object is part of.

###### Constructors {#api-dom-blockparent-constructor}

An implementation of `BlockParent` should have a constructor with signature `BlockParent()`.

For convenience, an implementation of `BlockParent` should have a constructor with signature `BlockParent(Block... blocks)` that delegates to `addBlocks(Block... blocks)`.

###### `getBlockParentType` {#api-dom-listblock-getblockparenttype}

A `BlockParent` object must have a method with signature `BlockParentType getBlockParentType()`.

This method must return the [`BlockParentType`] value that corresponds to the type of the `BlockParent` object.

###### `getBlocks` {#api-dom-blockparent-getblocks}

A `BlockParent` object must have a method with signature `[Block] getBlocks()`.

This method must return the companion [`Sequence`] object.

###### `addBlock` {#api-dom-blockparent-addblock}

For convenience, a `BlockParent` object should have a method with signature `addBlock(Block block)`.

This method must add `block` at the end of the companion [`Sequence`] object. This attaches `block` to the `BlockParent` object.

This method must fail if adding `block` to the companion [`Sequence`] object failed.

###### `addBlocks` {#api-dom-blockparent-addblocks}

For convenience, a `BlockParent` object should have a method with signature `addBlocks(Block... blocks)`.

This method must add all [`Block`] objects from `blocks` in the given order at the end of the companion [`Sequence`] object, as if `addBlock(Block block)` has been called repeatedly for all [`Block`] objects from `blocks`.  This attaches all [`Block`] objects from `blocks` to the `BlockParent` object.

This method must fail if `blocks` is not present.

This method must fail if adding any [`Block`] object from `blocks` to the companion [`Sequence`] object failed.

Because this method is a short hand for repeated calls to `addBlock(Block block)`, it must add all prior [`Block`] objects from `blocks` to the companion [`Sequence`] object, if it fails because of a violating [`Block`] object from `blocks`.

##### `CodeBlock` {#api-dom-codeblock}

A `CodeBlock` objects is a [`Block`] object that represents a [*Markdom code block*].

The initial value of the `code` parameter should be the empty string. The initial value of the `hint` parameter should be not present.

###### Constructors {#api-dom-codeblock-constructor}

An implementation of `CodeBlock` should have a constructor with signature `CodeBlock()`.

For convenience, an implementation of `CodeBlock` should have a constructor with signature `CodeBlock(String code)` that delegates to `setCode(String code)`.

For convenience, an implementation of `CodeBlock` should have a constructor with signature `CodeBlock(String code, String hint)` that delegates to `setCode(String code)` and `setHint(String hint)`.

###### `getCode` {#api-dom-codeblock-getcode}

A `CodeBlock` object must have a method with signature `String getCode()`.

The method must return the value of the `code` parameter of the represented [*Markdom code block*].

###### `setCode` {#api-dom-codeblock-setcode}

A `CodeBlock` object must have a method with signature `setCode(String code)`.

This method must set the value of the `code` parameter of the represented [*Markdom code block*].

This method must fail if `code` is not present.

###### `getHint` {#api-dom-codeblock-gethint}

A `CodeBlock` object must have a method with signature `String? getHint()`.

The method must return the optional `hint` parameter of the represented [*Markdom code block*].

###### `setHint` {#api-dom-codeblock-sethint}

A `CodeBlock` object must have a method with signature `setHint(String? hint)`.

This method must set the value of the `hint` parameter of the represented [*Markdom code block*].
  	
##### `CodeContent` {#api-dom-codecontent}

A `CodeContent` object is a [`Content`] object that represents a [*Markdom code content*].

The initial value of the value of the `code` parameter should be the empty string.

###### Constructors {#api-dom-codecontent-constructors}

An implementation of `CodeContent` should have a constructor with signature `CodeContent()`.

For convenience, an implementation of `CodeContent` should have a constructor with signature `CodeContent(String code)` that delegates to `setCode(String code)`.

###### `getCode` {#api-dom-codecontent-getcode}

A `CodeContent` object must have a method with signature `String getCode()`.

The method must return the value of the `code` parameter of the represented [*Markdom code content*].

###### `setCode` {#api-dom-codecontent-setcode}

A `CodeContent` object must have a method with signature `setCode(String code)`.

This method must set the value of the `code` parameter of the represented [*Markdom code content*].

This method must fail if `code` is not present.

##### `CommentBlock` {#api-dom-commentblock}

A `CommentBlock` objects is a [`Block`] object that represents a [*Markdom comment block*].

The initial value of the `comment` parameter should be the empty string.

###### Constructors {#api-dom-commentblock-constructor}

An implementation of `CommentBlock` should have a constructor with signature `CommentBlock()`.

For convenience, an implementation of `CommentBlock` should have a constructor with signature `CommentBlock(String comment)` that delegates to `setCode(String comment)`.

###### `getComment` {#api-dom-commentblock-getcode}

A `CommentBlock` object must have a method with signature `String getComment()`.

The method must return the value of the `comment` parameter of the represented [*Markdom comment block*].

###### `setComment` {#api-dom-commentblock-setcode}

A `CommentBlock` object must have a method with signature `setComment(String code)`.

This method must set the value of the `comment` parameter of the represented [*Markdom comment block*].

This method must fail if `code` is not present.

##### `Content` {#api-dom-content}

A `Content` object is a [`Node`] object that represents a [*Markdom content*].

###### Constructors {#api-dom-content-constructor}

An implementation of `Content` should have a constructor with signature `Content()`.

###### `getContentType` {#api-dom-content-getcontenttype}

A `Content` object must have a method with signature `ContentType getContentType()`.

This method must return the [`ContentType`] value that corresponds to the type of the represented [*Markdom content*].

###### `hasBlock` {#api-dom-node-hasblock}

A `Content` object must have a method with signature `Boolean hasBlock()`.

This method must return whether the path to the root of the tree of [Markdom nodes](#domain-node) the `Content` object is part of contains a [`Block`] object.

###### `getBlock` {#api-dom-content-getblock}

A `Content` object must have a method with signature `Block? getBlock()`.

This method must return the nearest [`Block`] object in a path to the root of the tree of [Markdom nodes](#domain-node) the `Content` object is part of or no value, if no such [`Block`] object exists, or no value, otherwise.

##### `ContentParent` {#api-dom-contentparent}

A `ContentParent` object is a [`Node`] object that represents a [*Markdom node*] that contains [*Markdom contents*].

An implementation of `ContentParent` must have a final and initially empty companion [`Sequence`] object for the [`Content`] objects that are associated with the `ContentParent` object.

Any structural modification (insert, remove, clear, replace) to the companion [`Sequence`] object must reflect the fact, that a [`Content`] object that is added to the companion [`Sequence`] object is attached to the `ContentParent` object until is is removed from the companion [`Sequence`] object.

Attaching a [`Content`] object to the `ContentParent` object  must fail
* if the [`Content`] object is not present, or
* if the [`Content`] object is already attached to a `ContentParent` object, or
* if attaching the [`Content`] object to the `ContentParent` object would create a [cycle](#api-dom-detecting-cycles) in the tree of Markdom nodes that the `ContentParent` object is part of.

###### Constructors {#api-dom-contentparent-constructor}

An implementation of `ContentParent` should have a constructor with signature `ContentParent()`.

For convenience, an implementation of `ContentParent` should have a constructor with signature `ContentParent(Content... contents)` that delegates to `addContents(Content... contents)`.

###### `getContentParentType` {#api-dom-listblock-getcontentparenttype}

A `ContentParent` object must have a method with signature `ContentParentType getContentParentType()`.

This method must return the [`ContentParentType`] value that corresponds to the type of the [`ContentParent`] object.

###### `getContents` {#api-dom-contentparent-getcontents}

A `ContentParent` object must have a method with signature `[Content] getContents()`.

This method must return the companion [`Sequence`] object.

###### `addContent` {#api-dom-contentparent-addcontent}

For convenience, a `ContentParent` object should have a method with signature `addContent(Content content)`.

This method must add `content` at the end of the companion [`Sequence`] object. This attaches `content` to the `ContentParent` object.

This method must fail if adding `content` to the companion [`Sequence`] object failed.

###### `addContents` {#api-dom-contentparent-addcontents}

For convenience, a `ContentParent` object should have a method with signature `addContents(Content... contents)`.

This method must add all [`Content`] objects from `contents` in the given order at the end of the companion [`Sequence`] object, as if `addContent(Content content)` has been called repeatedly for all [`Content`] objects from `contents`. This attaches all [`Content`] objects from `contents` to the companion [`Sequence`] object.

This method must fail if `contents` is not present.

This method must fail if adding any [`Content`] object from `contents` to the companion [`Sequence`] object failed.

Because this method is a short hand for repeated calls to `addContent(Content content)`, it must add all prior [`Content`] objects from `contents` to the companion [`Sequence`] object, if it fails because of a violating [`Content`] object from `contents`.

##### `ContentParentBlock` {#api-dom-contentparentblock}

A `ContentParentBlock` object is a [`Block`] object and a [`ContentParent`] object that represents a [*Markdom block*] that contains [*Markdom content*].

###### Constructors {#api-dom-contentparentblock-constructor}

An implementation of `ContentParentBlock` should have a constructor with signature `ContentParentBlock()`.

For convenience, an implementation of `ContentParentBlock` should have a constructor with signature `ContentParentBlock(Content... contents)` that delegates to `ContentParent#addContents(Content... content)`.

##### `ContentParentContent` {#api-dom-contentparentcontent}

A `ContentParentContent` object is a [`Content`] object and a [`ContentParent`] object that represents a [*Markdom content*] that contains [*Markdom content*].

###### Constructors {#api-dom-contentparentcontent-constructor}

An implementation of `ContentParent` should have a constructor with signature `ContentParentContent()`.

For convenience, an implementation of `ContentParentContent` should have a constructor with signature `ContentParentContent(Content... contents)` that delegates to `ContentParent#addContents(Content... content)`.

##### `DivisionBlock` {#api-dom-divisionblock}

A `DivisionBlock` object is a [`Block`] object that represents a [*Markdom division block*].

###### Constructors {#api-dom-divisionblock-constructors}

An implementation of `DivisionBlock` should have a constructor with signature `DivisionBlock()`

##### `Document` {#api-dom-document}

A `Document` object is a [`BlockParent`] object and a [`Dispatcher`] object that represents a [*Markdom document*].

###### Constructors {#api-dom-document-constructor}

For convenience, an implementation of `Document` should have a constructor with signature `Document(Block... blocks)` that delegates to `BlockParent#addBlocks(Block... blocks)`.

##### `EmphasisContent` {#api-dom-emphasiscontent}

An `EmphasisContent` object is a [`ContentParentContent`] object that represents a [*Markdom emphasis content*].

The initial value of the `level` parameter should be `LEVEL_1`.

###### Constructors {#api-dom-emphasiscontent-constructor}

An implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent()`.

For convenience, an implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent(EmphasisLevel level)` that delegates to `setLevel(EmphasisLevel level)`.

For convenience, an implementation of `EmphasisContent` should have a constructor with signature `EmphasisContent(EmphasisLevel level, Content... contents)` that delegates to `setLevel(EmphasisLevel level)` and `ContentParent#addContents(Content... contents)`.

###### `getLevel` {#api-dom-emphasiscontent-getlevel}

An `EmphasisContent` object must have a method with signature `EmphasisLevel getLevel()`.

This method must return the value of the `level` parameter of the represented [*Markdom emphasis content*].

###### `setLevel` {#api-dom-emphasiscontent-setlevel}

An `EmphasisContent` object must have a method with signature `setLevel(EmphasisLevel level)`.

This method must set the value of the `level` parameter of the represented [*Markdom emphasis content*].

This method must fail if `level` is not present.

##### `HeadingBlock` {#api-dom-headingblock}

A `HeadingBlock` object is a [`ContentParentBlock`] object that represents a [*Markdom heading block*].

The initial value of the `level` parameter should be `LEVEL_1`.

###### Constructors {#api-dom-headingblock-constructor}

An implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock()`.

For convenience, an implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock(HeadingLevel level)` that delegates to `setLevel(HeadingLevel level)`.

For convenience, an implementation of `HeadingBlock` should have a constructor with signature `HeadingBlock(HeadingLevel level, Content... contents)` that delegates to `setLevel(HeadingLevel level)` and `ContentParent#addContents(Contents... contents)`.

###### `getLevel` {#api-dom-headingblock-getlevel}

A `HeadingBlock` object must have a method with signature `HeadingLevel getLevel()`.

This method must return the value of the `level` parameter of the represented [*Markdom heading block*].

###### `setLevel` {#api-dom-headingblock-setlevel}

A `HeadingBlock` object must have a method with signature `setLevel(HeadingLevel level)`.

This method must set the value of the `level` parameter of the represented [*Markdom heading block*].

This method must fail if `level` is not present.

##### `ImageContent` {#api-dom-imagecontent}

An `ImageContent` object is a [`Content`] object that represents a [*Markdom image content*].

The initial value of the `uri` parameter should be the empty string. The initial value of the `title` parameter should be not present. The initial value of the `alternative` parameter should be not present.

###### Constructors {#api-dom-imagecontent-constructor}

An implementation of `LinkContent` should have a constructor with signature `LinkContent()`.

For convenience, an implementation of `ImageContent` should have a constructor with signature `ImageContent(Uri uri)` that delegates to `setUri(Uri uri)`.

For convenience, an implementation of `ImageContent` should have a constructor with signature `ImageContent(Uri uri, String? title, String? alternative)` that delegates to `setUri(Uri uri)` and `setTitle(String? title)` and `setAlternative(String? alternative)`.

###### `getUri` {#api-dom-imagecontent-geturi}

A `ImageContent` object must have a method with signature `String getUri()`.

This method must return the value of the `uri` parameter of the represented [*Markdom image content*].

###### `setUri` {#api-dom-imagecontent-seturi}

An `ImageContent` object must have a method with signature `setUri(Uri uri)`.

This method must set the value of the `uri` parameter of the represented [*Markdom image content*].

This method must fail if `uri` is not present.

This method must fail if value of the `uri` is not a valid [URI reference](https://tools.ietf.org/html/rfc3986#section-4.1).

###### `getTitle` {#api-dom-imagecontent-gettitle}

A `ImageContent` object must have a method with signature `String getTitle()`.

This method must return the value of the `title` parameter of the represented [*Markdom image content*].

###### `setTitle`{#api-dom-imagecontent-settitle}

An `ImageContent` object must have a method with signature `setTitle(String? title)`.

This method must set the value of the `title` parameter of the represented [*Markdom image content*].

###### `getAlternative` {#api-dom-imagecontent-getalternative}

A `ImageContent` object must have a method with signature `String getAlternative()`.

This method must return the value of the `alternative` parameter of the represented [*Markdom image content*].

###### `setAlternative` {#api-dom-imagecontent-setalternative}

An `ImageContent` object must have a method with signature `setAlternative(String? alternative)`.

This method must set the value of the `alternative` parameter of the represented [*Markdom image content*].

##### `LineBreakContent` {#api-dom-linebreakcontent}

A `LineBreakContent` object is a [`Content`] object that represents a [*Markdom line break content*].

The initial value of the `hard` parameter should be `false`.

###### Constructors {#api-dom-linebreakcontent-constructor}

An implementation of `LineBreakContent` should have a constructor with signature `LineBreakContent()`.

For convenience, an implementation of `LineBreakContent` should have a constructor with signature `LinkContent(Boolean hard)` that delegates to `setHard(Boolean hard)`.

###### `isHard` {#api-dom-linebreakcontent-ishard}

A `LineBreakContent` object must have a method with signature `Boolean isHard()`.

This method must return the value of the `hard` parameter of the represented [*Markdom line break content*].

###### `setHard` {#api-dom-linebreakcontent-sethard}

A `LineBreakContent` object must have a method with signature `setHard(Boolean hard)`.

This method must set the value of the `hard` parameter of the represented [*Markdom line break content*].

This method must fail if `hard` is not present.

##### `LinkContent` {#api-dom-linkcontent}

A `LinkContent` object is a [`ContentParentContent`] object that represents a [*Markdom link content*]. The initial value of the `title` parameter should be not present.

The initial value of the `uri` parameter should be the empty string.

###### Constructors {#api-dom-linkcontent-constructor}

An implementation of `LinkContent` should have a constructor with signature `LinkContent()`.

For convenience, an implementation of `LinkContent` should have a constructor with signature `LinkContent(Uri uri)` that delegates to `setUri(Uri uri)`.

For convenience, an implementation of `LinkContent` should have a constructor with signature `LinkContent(Uri uri, String? title)` that delegates to `setUri(Uri uri)` and `setTitle(String? title)`.

For convenience, an implementation of `LinkContent` should have a constructor with signature `LinkContent(Uri uri, String? title, Content... contents)` that delegates to `setUri(Uri uri)` and `setTitle(String? title)` and `ContentParent#addContents(Content... contents)`.

###### `getUri` {#api-dom-imagecontent-geturi}

A `LinkContent` object must have a method with signature `String getUri()`.

This method must return the value of the `uri` parameter of the represented [*Markdom link content*].

###### `setUri` {#api-dom-linkcontent-seturi}

A `LinkContent` object must have a method with signature `setUri(Uri uri)`.

This method must set the value of the `uri` parameter of the represented [*Markdom link content*].

This method must fail if `uri` is not present.

This method must fail if value of the `uri` is not a valid [URI reference](https://tools.ietf.org/html/rfc3986#section-4.1).

###### `getTitle` {#api-dom-linkcontent-gettitle}

A `LinkContent` object must have a method with signature `String getTitle()`.

This method must return the value of the `title` parameter of the represented [*Markdom link content*].

###### `setTitle`{#api-dom-linkcontent-settitle}

An `LinkContent` object must have a method with signature `setTitle(String? title)`.

This method must set the value of the `title` parameter of the represented [*Markdom link content*].

##### `ListBlock` {#api-dom-listblock}

A `ListBlock` object is a [`Block`] object that represents a *Markdom list block*.

An implementation of `ListBlock` must have a final and initially empty companion [`Sequence`] object for the [`ListItem`] objects that are associated with the `ListBlock` object.

Any structural modification (insert, remove, clear, replace) to the companion [`Sequence`] object must reflect the fact, that a [`ListItem`] object that is added to the companion [`Sequence`] object is attached to the `ListBlock` object until is is removed from the companion [`Sequence`] object.

Attaching a [`ListItem`] object to the `ListBlock` object  must fail
* if the [`ListItem`] object is not present, or
* if the [`ListItem`] object is already attached to a `ListBlock` object, or
* if attaching the [`ListItem`] object to the `ListBlock` object would create a [cycle](#api-dom-detecting-cycles) in the tree of Markdom nodes that the `ListBlock` object is part of.

###### Constructors {#api-dom-listblock-constructor}

An implementation of `ListBlock` should have a constructor with signature `ListBlock()`.

For convenience, an implementation of `ListBlock` should have a constructor with signature `ListBlock(ListItem... items)` that delegates to `addItems(ListItem... items)`.

###### `getListBlockType` {#api-dom-listblock-getlistblocktype}

A [`ListBlock`] object must have a method with signature `ListBlockType getListBlockType()`.

This method must return the [`ListBlockType`] value that corresponds to the type of the `ListBlock` object.

###### `getItems` {#api-dom-listblock-getitems}

A [`ListItem`] object must have a method with signature `[ListItem] getItems()`.

This method must return the companion [`Sequence`] object.

###### `addItem` {#api-dom-listblock-additem}

For convenience, a `ListBlock` object should have a method with signature `addItem(ListItem item)`.

This method must add `item` at the end of the companion [`Sequence`] object. This attaches `item` to the `ListBlock` object.

This method must fail if adding `item` to the companion [`Sequence`] object failed.

###### `addItems` {#api-dom-listblock-additems}

For convenience, a `ListBlock` object should have a method with signature `addItems(ListItem... items)`.

This method must add all [`ListItem`] objects from `items` in the given order at the end of the companion [`Sequence`] object, as if `addItem(ListItem item)` has been called repeatedly for all [`ListItem`] objects from `items`.  This attaches all [`ListItem`] objects from `items` to the `ListBlock` object.

This method must fail if `items` is not present.

This method must fail if adding any [`ListItem`] object from `items` to the  companion [`Sequence`] object failed.

Because this method is a short hand for repeated calls to `addItem(ListItem item)`, it must add all prior [`ListItem`] objects from `items` to the companion [`Sequence`] object, if it fails because of a violating [`ListItem`] object from `items`.

##### `ListItem` {#api-dom-listitem}

A `ListItem` object is a [`BlockParent`] object that represents a [*Markdom list item*].

###### Constructors {#api-dom-listitem-constructor}

An implementation of `ListItem` should have a constructor with signature `ListItem()`.

For convenience, an implementation of `ListItem` should have a constructor with signature `ListItem(Block... blocks)` that delegates to `BlockParent#addBlocks(Block... blocks)`.

##### `Node` {#api-dom-node}

A `Node` object represents a [*Markdom node*].

###### `getNodeType` {#api-dom-node-getnodetype}

A `Node` object must have a method with signature `NodeType getNodeType()`.

This method must return the [`NodeType`] value that corresponds to the type of the `Node` object.

###### `hasParent` {#api-dom-node-hasparent}

A `Node` object must have a method with signature `Boolean hasParent()`.

This method must return whether the `Node` object has a parent `Node` object.

Specifically, this method must return `true`
* if the `Node` object is a [`Block`] object that is currently attached to a [`BlockParent`] object, or
* if the `Node` object is a [`ListItem`] object that is currently attached to a [`ListBlock`] object, or
* if the `Node` object is a [`Content`] object that is currently attached to a [`ContentParent`] object, or
`false`, otherwise.

###### `getParent` {#api-dom-node-getparent}

A `Node` object must have a method with signature `Node? getParent()`.

This method must return the parent `Node` object of the `Node` object.

Specifically, this method must return
* the [`BlockParent`] object it is currently attached to, if the `Node` object is a [`Block`] object, or
* the [`ListBlock`] object it is currently attached to, if the `Node` object is a [`ListItem`] object, or
* the [`ContentParent`] object it is currently attached, if the `Node` object is a [`Content`] object, or
no value, otherwise.

###### `hasDocument` {#api-dom-node-hasdocument}

A `Node` object must have a method with signature `Boolean hasDocument()`.

This method must return whether the root of the tree of [Markdom nodes](#domain-node) the `Node` object is part of is a [`Document`] object.

###### `getDocument` {#api-dom-node-getdocument}

A `Node` object must have a method with signature `Document? getDocument()`.

This method must return the root of the tree of [Markdom nodes](#domain-node) the `Node` object is part of, if it is a [`Document`] object, or no value, otherwise.

###### `getIndex` {#api-dom-node-getindex}

A `Node` object must have a method with signature `Integer? getIndex()`.

This method must return the index of the `Node` object in the [`Iterable`] object of [child](#api-dom-node-getchildren) `Node` object of the parent `Node`.

Specifically, this method must return
* the index in the companion [`Sequence`] object of the [`BlockParent`] object it is currently attached to, if the `Node` object is a [`Block`] object, or
* the index in the companion [`Sequence`] object of the [`ListBlock`] object it is currently attached to, if the `Node` object is a [`ListItem`] object, or
* the index in the companion [`Sequence`] object of the [`ContentParent`] object it is currently attached to, if the `Node` object is a [`Content`] object.

###### `hasChildren` {#api-dom-node-haschildren}

A `Node` object must have a method with signature `Boolean hasChildren()`.

This method must return whether the `Node` object has child `Node` objects.

Specifically, this method must return `true`
* if the `Node` object is a [`BlockParent`] object and currently has a [`Block`] object attached to it, or
* if the `Node` object is a [`ListBlock`] object and currently has a [`ListItem`] object attached to it, or
* if the `Node` object is a [`ContentParent`] object and currently has a [`Content`] object attached to it, or
`false`, otherwise.

###### `countChildren` {#api-dom-node-countchildren}

A `Node` object must have a method with signature `Integer countChildren()`.

This method must return the number of child `Node` objects the `Node` object has.

Specifically, this method must return
* the number of attached  [`Block`] objects, if the `Node` object is a [`BlockParent`] object, or
* the number of attached  [`ListItem`] objects, if the `Node` object is a [`ListBlock`] object, or
* the number of attached  [`Content`] objects, if the `Node` object is a [`ContentParent`] object, or
`0`, otherwise.

###### `getChildren` {#api-dom-node-getchildren}

A `Node` object must have a method with signature `Node... getChildren()`.

This method must return an [`Iterable`] object that yields the child `Node` objects of the `Node` object.

Specifically, this method must return
* an [`Iterable`] object for the companion [`Sequence`] object, if the `Node` object is a [`BlockParent`] object, or
* an [`Iterable`] object for the companion [`Sequence`] object, if the `Node` object is a [`ListBlock`] object, or
* an [`Iterable`] object for the companion [`Sequence`] object, if the `Node` object is a [`ContentParent`] object, or
* an empty [`Iterable`] object, otherwise.

##### `OrderedListBlock` {#api-dom-orderedlistblock}

An [`OrderedListBlock`] object is a [`ListBlock`] object that represents an [*ordered Markdom list block*].

The initial value of the `startIndex` parameter should be `1`.

###### Constructors {#api-dom-orderedlistblock-constructor}

An implementation of `OrderedListBlock` should have a constructor with signature `OrderedListBlock()`.

For convenience, an implementation of `OrderedListBlock` should have a constructor with signature `OrderedListBlock(Integer startIndex)` that delegates to `setStartIndex(Integer startIndex)`.

For convenience, an implementation of `OrderedListBlock` should have a constructor with signature `OrderedListBlock(Integer startIndex, ListItem... items)` that delegates to `setStartIndex(Integer startIndex)` and `ContentParent#addContents(Content... contents)`.

###### getStartIndex {#api-dom-orderedlistblock-getstartindex}

An `OrderedListBlock` object must have a method with signature `Integer getStartIndex()`.

This method must return the value of the `startIndex` parameter of the represented [*ordered Markdom list block*].

###### setStartIndex {#api-dom-orderedlistblock-setstartindex}

An `OrderedListBlock` object must have a method with signature `setStartIndex(Integer startIndex)`.

This method must set the value of the `startIndex` parameter of the represented [*ordered Markdom list block*].

This method must fail if `startIndex` is not present.

This method must fail if value of the `startIndex` is negative.

##### `ParagraphBlock` {#api-dom-paragraphblock}

A `ParagraphBlock` object is a [`ContentParentBlock`] object that represents a [*Markdom paragraph block*].

###### Constructors {#api-dom-paragraphblock-constructor}

An implementation of `ParagraphBlock` should have a constructor with signature `ParagraphBlock()`.

For convenience, an implementation of `ParagraphBlock` should have a constructor with signature `ParagraphBlock(Content... contents)` that delegates to `ContentParentBlock#addContents(Contents... contents)`.

##### `QuoteBlock` {#api-dom-quoteblock}

A `QuoteBlock` object is a [`Block`] object and a [`BlockParent`] object that represents a [*Markdom quote block*].

###### Constructors {#api-dom-quoteblock-constructor}

An implementation of `QuoteBlock` should have a constructor with signature `QuoteBlock()`.

For convenience, an implementation of `QuoteBlock` should have a constructor with signature `QuoteBlock(Block... blocks)` that delegates to `BlockParent#addBlocks(Block... blocks)`.

##### `TextContent` {#api-dom-textcontent}

A `TextContent` object is [`ContentParentContent`] object that represents a [*Markdom text content*].

The initial value of the `text` parameter should be the empty string.

###### Constructors {#api-dom-textcontent-constructor}

An implementation of `TextContent` should have a constructor with signature `TextContent()`.

For convenience, an implementation of `TextContent` should have a constructor with signature `TextContent(String text)` that delegates to `setText(String text)`.

###### `getText` {#api-dom-textcontent-gettext}

A `TextContent` object must have a method with signature `String getText()`.

This method must return the value of the `text` parameter of the represented [*Markdom text content*].

###### `setText` {#api-dom-textcontent-settext}

A `TextContent` object must have a method with signature `setText(String text)`.

This method must set the value of the `text` parameter of the represented [*Markdom text content*].

This method must fail if `text` is not present.

##### `UnorderedListBlock` {#api-dom-unorderedlistblock}

An `UnorderedListBlock` object is a [`ListBlock`] object that represents an *unordered Markdom list block*.

###### Constructors {#api-dom-unorderedlistblock-constructor}

An implementation of `UnorderedListBlock` should have a constructor with signature `UnorderedListBlock()`.

For convenience, an implementation of `UnorderedListBlock` should have a constructor with signature `UnorderedListBlock(ListItem... items)` that delegates to  `ListBlock#addItems(ListItem... items)`.

#### Enumerations {#api-enumerations}

The [Domain Model API](#api-dom) has the following enumerations:

##### `BlockParentType` {#api-dom-blockparenttype}

The `BlockParentType` enum indicates the type of a [`BlockParent`] object and has the following constants:

* `DOCUMENT`,
* `QUOTE_BLOCK`,
* `LIST_ITEM`.

##### `ContentParentType` {#api-dom-contentparenttype}

The `ContentParentType` enum indicates the type of a [`ContentParent`] object and has the following constants:

* `HEADING_BLOCK`,
* `PARAGRAPH_BLOCK`,
* `EMPHASIS_CONTENT`,
* `LINK_CONTENT`.

##### `NodeType` {#api-dom-nodetype}

The `NodeType` enum indicates the type of a [`Node`] object and has the following constants:

* `DOCUMENT`,
* `BLOCK`,
* `LIST_ITEM`,
* `CONTENT`.

##### `ListBlockType` {#api-dom-listblocktype}

The `ListBlockType` enum indicates the type of a [`ListBlock`] object and has the following constants:

* `ORDERED`,
* `UNORDERED`.

#### Example Document {#api-dom-example}

The following [Domain Model API](#api-dom) object graph represents the [example document](#example):

![](resource/markdom-objectgraph.png)

Every [`BlockParent`], [`ListBlock`] or [`ContentParent`] object has a reference to its companion [`Sequence`] object. A companion [`Sequence`] object holds references to the [children](#api-dom-node-getchildren) of the corresponding [`BlockParent`], [`ListBlock`] or [`ContentParent`] object. Each child has a reference to its [parent](#api-dom-node-getparent).

#### Up navigation {#api-dom-downnavigation}

The following image shows the possible methods to navigate from an [`Node`] object upwards from the leaf [`TextContent`] object with `text` value `Baz` of the [example document](#example).

![](resource/markdom-navigation-up.png)

Every [`Node`] object in a [Domain Model API](#api-dom) object graph that is not a [`Document`] object has a reference to its [parent](#api-dom-node-getparent), which is

* a [`BlockParent`] object, if the [`Node`] object is a [`Block`] object,
* a [`ListBlock`] object, if the [`Node`] object is a [`ListItem`] object, or
* a [`ContentParent`] object, if the [`Node`] object is a [`Content`] object.

Depending on the kind of the parent, either `getBlockParentType`, `getListBlockType` or `getContentParentType` can be used to find out the actual type of the parent.

For example: Consider a method that gets a leaf [`Node`] object as a parameter without any knowledge about it. Calling `getNodeType()` reveals that is it a [`Content`] object. Calling `getContentType()` reveals that it is a [`TextContent`] object. Calling `getParent()` returns the parent as a [`ContentParent`] object. Calling `getContentParentType()` on the parent reveals that the parent is an [`EmphasisContent`] object; and so on.

Each [`Node`] object in a Domain Model API object graph can, through its predecessors, retrieve the root [`Document`] object. Each [`Content`] object in a Domain Model API object graph can, through its predecessors, retrieve its [`Block`] object.

#### Down navigation {#api-dom-upnavigation}

The following image shows the possible methods to navigate from an [`Node`] object downwards to the leaf [`TextContent`] object with `text` value `Baz` of the [eample document](#example).

![](resource/markdom-navigation-down.png)

Every [`Node`] object in a [Domain Model API](#api-dom) object graph that is parent object has a reference to its companion [`Sequence`] object which has references to the [children](#api-dom-node-getchildren) of the [`Node`] object.

For example: Consider a method that gets the root `Node` as a parameter without any knowledge about it. Calling `getNodeType()` reveals that is it a [`Document`] object. Calling `getBlocks` returns the companion [`Sequence`] object. Calling `getSize()` reveals that the companion `Sequence` object contains three [`Block`] objects. Calling `get(1)` returns the second child. Calling `getBlockType()` on the second child reveals that it is an [`OrderedListBlock`] object; and so on.

#### Detecting cycles {#api-dom-detectingcycles}

If a [`Block`], [`ListItem`] or [`Content`] object is about to be attached to a [`BlockParent`], [`ListBlock`] or [`ContentParent`] object respectively, it is necessary to check, whether this would create a cycle.

If the designated parent object is part of a [*Markdom document*] (i.e. a tree of [`Node`] objects which has a root [`Document`] object), it is not possible to create a cycle, because repeatedly retrieving the [parent](#api-dom-node-getparent) will eventually yield the root [`Document`] object which by definition doesn't [have](#api-dom-node-hasparent) a parent object and thus ending the path after a finite number of repetitions.

If the designated parent object is not part of a [*Markdom document*] (and therefore part of a tree fragment where one [`Node`] object that isn't a [`Document`] object doesn't have a parent), it is possible to create a cycle. The object that is about to be attached doesnt't currently have a parent or otherwise it wouldn't be eligible to be attached to the a parent. The only object in the tree fragment the designated parent is a part of that doesn't have a parent is the root object of that tree fragment. It is therefore necessary and sufficient to check whether the object that is about to be attached is not the root of the tree fragment that the designated parent is a part of, to ensure that no cycle is created.

This can be accomplished by repeatedly retrieving the parent of the designated parent until a [`Node`] object that doesn't have a parent is found and check whether the last [`Node`] object in the chain of parent is the object that is about to be added.

For example: Assuming a [`QuoteBlock`] object is about to be attached to a [`BlockParent`] object. That [`BlockParent`] object might be another [`QuoteBlock`] object which is attached to a [`ListItem`] object which is attached to an [`UnorderedListBlock`] object which is attached to a [`QuoteBlock`] object which isn't attached to a [`BlockParent`] object.  The last [`QuoteBlock`] object in the chain of parents might be the same [`QuoteBlock`] object that is about to be attached. Attaching the [`QuoteBlock`] object to its designated parent would therefore create a cycle. The following image illustrates this example.

![](resource/markdom-cycles.png)

### Handler API {#api-handler}

The Handler API represents a [*Markdom document*] as a succession of events.

#### Events {#api-handler-events}

The Handler API defines several events and the order in which these must occur to successfully describe a [*Markdom document*].

To help different handler implementations to process the described [*Markdom document*] easily, some of the events carry redundant information:
 * Every event concerning a [*Markdom node*] of a polymorph kind (i.e. [*Markdom blocks*] and [*Markdom contents*]) is reported twice. Once in a general form and once in a specific form with the specific parameters. This allows to execute general or specific behavior when necessary.
 * Every event that is part of a pair (i.e. opening and closing events) and has parameters, carries the same values for the parameters as its counterpart. This usually eliminates the need to remember previously reported values.

##### Document {#api-handler-document}

A [*Markdom document*] is represented by an `onDocumentBegin` event and an `onDocumentEnd` event that frame the events that describe the sequence of [*Markdom blocks*] the described [*Markdom document*] consists of.

![](resource/markdom-events-document.png)

##### Blocks {#api-handler-blocks}

A sequence of [*Markdom blocks*] is represented by an `onBlocksBegin` event and an `onBlocksEnd` event that frame the events that describe the [*Markdom blocks*]. Consecutive [*Markdom blocks*] are separated by an `onNextBlock` event.

![](resource/markdom-events-blocks.png)

##### Block {#api-handler-block}

A [*Markdom block*] is represented by an `onBlockBegin` event and an `onBlockEnd` event that frame the events that describe the [*Markdom block*]. Both events carry the [`BlockType`] value that corresponds to the type of the described [*Markdom block*] as a parameter.

* If the [*Markdom block*] is a [*Markdom code block*], it is represented as an `onCodeBlock` event. The event carries the `code` parameter of the [*Markdom code block*].
* If the [*Markdom block*] is a [*Markdom comment block*], it is represented as an `onCommentBlock` event. The event carries the `comment` parameter of the [*Markdom comment block*].
* If the [*Markdom block*] is a [*Markdom heading block*], it is represented as an `onHeadingBlockBegin` event and an `onHeadingBlockEnd` event that frame the events that describe the sequence of [*Markdom contents*] the described [*Markdom heading block*] consists of. Both events carry the `level` parameter of the described [*Markdom heading block*].
* If the [*Markdom block*] is a [*Markdom division block*], it is represented as an `onDivisionBlock` event.
* If the [*Markdom block*] is an [*ordered Markdom list block*], it is represented as an `onOrderedListBlockBegin` event and an `onOrderedListBlockEnd` event that frame the events that describe the sequence of [*Markdom list items*] the described [*ordered Markdom list block*] consists of. Both events carry the `startIndex` parameter of the described [*ordered Markdom list block*].
* If the [*Markdom block*] is a [*Markdom paragraph block*], it is represented as an `onParagraphBlockBegin` event and an `onParagraphBlockEnd` event that frame the events that describe the sequence of [*Markdom contents*] the described [*Markdom paragraph block*] consists of.
* If the [*Markdom block*] is a [*Markdom quote block*], it is represented as an `onQuoteBlockBegin` event and an `onQuoteBlockEnd` event that frame the events that describe the sequence of [*Markdom blocks*] the described [*Markdom quote block*] consists of.
* If the [*Markdom block*] is an *unordered Markdom list block*, it is represented as an `onUNorderedListBlockBegin` event and an `onUnorderedListBlockEnd` event that frame the events that describe the sequence of [*Markdom list items*] the described *unordered Markdom list block* consists of.

![](resource/markdom-events-block.png)

##### List Items {#api-handler-listitems}

A sequence of [*Markdom list items*] is represented by an `onListItemsBegin` event and an `onListItemEnd` event that frame the events that describe the [*Markdom list items*]. Consecutive [*Markdom list items*] are separated by an `onNextListItem` event.

![](resource/markdom-events-listitems.png)

##### List Item {#api-handler-listitem}

A [*Markdom list item*] is represented by an `onListItemBegin` event and an `onListItemEnd` event that frame the events that describe the sequence of [*Markdom blocks*] the described [*Markdom list item*] consists of.

![](resource/markdom-events-listitem.png)

##### Contents {#api-handler-contents}

A sequence of [*Markdom contents*] is represented by an `onContentsBegin` event and an `onContentsEnd` event that frame the events that describe the [*Markdom contents*]. Consecutive [*Markdom contents*] are separated by an `onNextContent` event.

![](resource/markdom-events-contents.png)

##### Content {#api-handler-content}

A [*Markdom content*] is represented by an `onContentBegin` event and an `onContentEnd` event that frame the events that describe the [*Markdom content*]. Both events carry the [`ContentType`] value that corresponds to the type of the of the described [*Markdom content*] as a parameter.

* If the [*Markdom content*] is a [*Markdom code content*], it is represented as an `onCodeContent` event. The event carries `code` parameter of the described [*Markdom code content*].
* If the [*Markdom content*] is a [*Markdom emphasis content*], it is represented as an `onEmphasisContentBegin` event and an `onEmphasisContentEnd` event that frame the events that describe the sequence of [*Markdom contents*] the described [*Markdom emphasis content*] consists of. Both events carry the `level` parameter of the described [*Markdom emphasis content*].
* If the [*Markdom content*] is a [*Markdom image content*], it is represented as an `onImageContent` event. The event carries the `uri`, `title` and `alternative` parameters of the described [*Markdom image content*].
If the [*Markdom content*] is a [*Markdom line break content*], it is represented as an `onLineBreakContent` event. The event carries the `hard` parameter of the described [*Markdom line break content*].
* If the [*Markdom content*] is a [*Markdom link content*], it is represented as an `onLinkContentBegin` event and an `onLinkContentEnd` event that frame the events that describe the sequence of [*Markdom contents*] the described [*Markdom link content*] consists of. Both events carry the `uri` and `title` parameters of the described [*Markdom link content*].

![](resource/markdom-events-content.png)

#### Interfaces {#api-handler-interface}

##### `Dispatcher` {#api-handler-dispatcher}
A `Dispatcher` object is a component that is able to describe a [*Markdom document*] to a `Handler` by dispatching a sequence of [*Markdom events*].

###### `handle` {#api-handler-dispatcher-handle}

A `Dispatcher` object must have a method with signature `Object handle(Handler handler)`.

This method must dispatch [*Markdom events*] that describe the [*Markdom document*] to `handler` in the correct order.

This method must return the [result](#api-handler-handler-getresult) of `handler`.

Calling this method multiple times on a `Dispatcher` object that is not [reussable](#api-handler-isreusable) has an undefined behavior, unless explicitly stated otherwise.

This method must fail if `handler` is not present.

This method must fail if the `Dispatcher` object is not [reussable](#api-handler-dispatcher-isreusable) and has already been used.

###### `isReusable` {#api-handler-dispatcher-isreusable}

A `Dispatcher` object must have a method with signature `Boolean isReusable()`.

This method must return whether the `Dispatcher` object is able to [describe](#api-handler-dispatcher-handle) [*Markdom documents*] to multiple [`Handler`] objects (e.g. a [`Document`] object can describe itself multiple times, but a CommonMark parsing [`Handler`] object that consumes its input can't).

If the `Dispatcher` object is able to [describe](#api-handler-dispatcher-handle) [*Markdom documents*] to multiple [`Handler`] objects, it is not guaranteed that the described [*Markdom documents*] are equal because the underlying data source (e.g. a [`Document`] object) may have been modified.

##### `Handler` {#api-handler-handler}

A `Handler` object is a component that is able to receive a a sequence of [*Markdom events*] that describe a [*Markdom document*] (e.g. from a [`Dispatcher`] object) and calculate a result or causes side effects that corresponds to the described [*Markdom document*] (e.g. a `Handler` object can generate a [`Document`] object or write CommonMark text into a file).

Calling methods on a `Handler` object in an order that doesn't properly describe a [*Markdom document*] has an undefined behavior.

###### `getResult` {#api-handler-handler-getresult}

A `Handler` object must have a method with signature `Object getResult()`.

This method must return the calculated result that corresponds to the described [*Markdom document*] (e.g. a `Handler` object that generates a [`Document`] object returns that [`Document`] object and a `Handler` object that writes CommonMark text into a file returns `null`).

Calling this method before `onDocumentend()` has been called yields an undefined result, unless explicitly stated otherwise.

###### `onBlockBegin` {#api-handler-handler-onblockbegin}

A `Handler` object must have a method with signature `onBlockBegin(BlockType type)`.

This event represents the general form of the begin of a [*Markdom block*]. The `type` parameter determines the type of the represented [*Markdom block*].

Calling this method must be, depending on the value of the `type` parameter, followed by a call to `onCodeBlock` or `onCommentBlock` or `onHeadingBlockBegin` or `onDivisionBlock` or `onOrderedListBlockBegin` or `onParagraphBlockBegin` or `onQuoteBlockBegin` or `onUnorderedListBlockBegin`. A corresponding call to `onBlockEnd` with the same `type` value must eventually occur.

The behavior of this method is undefined, if `type` is not present.

###### `onBlockEnd` {#api-handler-handler-onblockend}

A `Handler` object must have a method with signature `onBlockEnd(BlockType type)`.

This event represents the general form of the end of a [*Markdom block*]. The `type` parameter determines the type of the represented [*Markdom block*].

A corresponding call to `onBlockBegin` with the same `type` value must have occurred. Calling this method must be followed by a call to `onNextBlock` or `onBlocksEnd`.

The behavior of this method is undefined, if `type` is not present.

###### `onBlocksBegin` {#api-handler-handler-onblocksbegin}

A `Handler` object must have a method with signature `onBlocksBegin()`.

This event represents the begin of a sequence ob [*Markdom blocks*].

Calling this method must be followed by a call to `onBlockBegin` or `onBlocksEnd`. A corresponding call to `onBlocksEnd` must eventually occur.

###### `onBlocksEnd` {#api-handler-handler-onblocksend}

A `Handler` object must have a method with signature `onBlocksEnd()`.

This event represents the end of a sequence ob [*Markdom blocks*].

A corresponding call to `onBlocksBegin` must have occurred. Calling this method must be followed by a call to `onDocumentEnd` or `onQuoteBlockEnd` or `onListItemEnd`.

###### `onCodeBlock` {#api-handler-handler-oncodeblock}

A `Handler` object must have a method with signature `onCodeBlock(String code, String? hint)`.

This event represents a [*Markdom code block*] with the given values for the `code` and `hint` parameters.

Calling this method must be followed by a call to `onBlockEnd`.

The behavior of this method is undefined, if `code` is not present.

###### `onCodeContent` {#api-handler-handler-oncodecontent}

A `Handler` object must have a method with signature `onCodeContent(String code)`.

This event represents a [*Markdom code content*] with the given value for the `code` parameter.

Calling this method must be followed by a call to `onContentEnd`.

The behavior of this method is undefined, if `code` is not present.

###### `onCommentBlock` {#api-handler-handler-oncommentblock}

A `Handler` object must have a method with signature `onCommentBlock(String comment)`.

This event represents a [*Markdom comment block*] with the given value for the `comment` parameter.

Calling this method must be followed by a call to `onBlockEnd`.

The behavior of this method is undefined, if `code` is not present.

###### `onContentBegin` {#api-handler-handler-oncontentbegin}

A `Handler` object must have a method with signature `onContenBegin(ContentType type)`.

This event represents the general form of the begin of a [*Markdom content*]. The `type` parameter determines the type of the represented [*Markdom content*].

Calling this method must be, depending on the value of the `type` parameter, followed by a call to `onCodeContent` or `onEmphasisContentBegin` or `onImageContent` or `onLineBreakContent` or `onLinkContentBegin` or `onTextContent`. A corresponding call to `onContentEnd` with the same `type` value must eventually occur.

The behavior of this method is undefined, if `type` is not present.

###### `onContentEnd` {#api-handler-handler-oncontentend}

A `Handler` object must have a method with signature `onContentEnd(ContentType type)`.

This event represents the general form of the end of a [*Markdom content*]. The `type` parameter determines the type of the represented [*Markdom content*].

A corresponding call to `onContentBegin` with the same `type` value must have occurred. Calling this method must be followed by a call to `onNextContent` or `onContentsEnd.

The behavior of this method is undefined, if `type` is not present.

###### `onContentsBegin` {#api-handler-handler-oncontentsbegin}

A `Handler` object must have a method with signature `onContentsBegin()`.

This event represents the begin of a sequence ob [*Markdom contents*].

Calling this method must be followed by a call to `onContentBEgin` or `onContentsEnd`. A corresponding call to `onContentsEnd` must eventually occur.

###### `onContentsEnd` {#api-handler-handler-oncontentsend}

A `Handler` object must have a method with signature `onContentsEnd()`.

This event represents the end of a sequence ob [*Markdom contents*].

A corresponding call to `onContentsBegin` must have occurred. Calling this method must be followed by a call to `onHeadingBlockEnd` or `onParagraphBlockEnd` or `onEmphasisContentEnd` or `onLinkContentEnd`.

###### `onDivisionBlock` {#api-handler-handler-ondivisionblock}

A `Handler` object must have a method with signature `onDivisionBlock()`.

This event represents a [*Markdom division block*].

Calling this method must be followed by a call to `onBlockEnd`.

###### `onDocumentBegin` {#api-handler-handler-ondocumentbegin}

A `Handler` object must have a method with signature `onDocumentBegin()`.

This event represents the begin of a [*Markdom document*].

This is the first method that must be called. Calling this method must be followed by a call to `onBlocksBegin`. A corresponding call to `onDocumentEnd` must eventually occur.

###### `onDocumentEnd` {#api-handler-handler-ondocumentend}

A `Handler` object must have a method with signature `onDocumentEnd()`.

This event represents the end of a [*Markdom document*].

A corresponding call to `onDocumentBegin` must have occurred. Calling this method may be followed by a call to `getResult`.

###### `onHeadingBlockBegin` {#api-handler-handler-onheadingblockbegin}

A `Handler` object must have a method with signature `onHeadingBlockBegin(HeadingLevel level)`.

This event represents the begin of a [*Markdom heading block*] with the given value for the `level` parameter.

Calling this method must be followed by a call to `onContentsBegin`. A corresponding call to `onHeadingBlockEnd` with the same `level` value must eventually occur.

The behavior of this method is undefined, if `level` is not present.

###### `onHeadingBlockEnd` {#api-handler-handler-onheadingblockend}

A `Handler` object must have a method with signature `onHeadingBlockEnd(HeadingLevel level)`.

This event represents the end of a [*Markdom heading block*] with the given value for the `level` parameter.

A corresponding call to `onHeadingBlockBegin` with the same `level` value must have occurred. Calling this method must be followed by a call to `onBlockEnd`.

The behavior of this method is undefined, if `level` is not present.

###### `onEmphasisContentBegin` {#api-handler-handler-onemphasiscontentbegin}

A `Handler` object must have a method with signature `onEmphasisContentBegin(EmphasisLevel level)`.

This event represents the begin of a [*Markdom emphasis content*] with the given value for the `level` parameter.

Calling this method must be followed by a call to `onContentsBegin`. A corresponding call to `onEmphasisContentEnd` with the same `level` value must eventually occur.

The behavior of this method is undefined, if `level` is not present.

###### `onEmphasisContentEnd` {#api-handler-handler-onemphasiscontentend}

A `Handler` object must have a method with signature `onEmphasisContentEnd(EmphasisLevel level)`.

This event represents the end of a [*Markdom emphasis content*] with the given value for the `level` parameter.

A corresponding call to `onEmphasisContentBegin` with the same `level` value must have occurred. Calling this method must be followed by a call to `onContentEnd`.

The behavior of this method is undefined, if `level` is not present.

###### `onImageContent` {#api-handler-handler-onimagecontent}

A `Handler` object must have a method with signature `onImageContent(Uri uri, String? title, String? alternative)`.

This event represents a [*Markdom image content*] with the given values for the `uri`, `title` and `alternative` parameters.

Calling this method must be followed by a call to `onContentEnd`.

The behavior of this method is undefined, if `uri` is not present.

The behavior of this method is undefined, if `uri` is not a valid [URI reference](https://tools.ietf.org/html/rfc3986#section-4.1).

###### `onLineBreakContent` {#api-handler-handler-onlinebreakcontent}

A `Handler` object must have a method with signature `onLineBreakContent(Boolean hard)`.

This event represents a [*Markdom line break content*] with the given value for the `hard` parameter.

Calling this method must be followed by a call to `onContentEnd`.

The behavior of this method is undefined, if `hard` is not present.

###### `onLinkContentBegin` {#api-handler-handler-onlinkcontentbegin}

A `Handler` object must have a method with signature `onLinkContentBegin(Uri uri, String? title)`.

This event represents the begin of a [*Markdom link content*] with the given value for the `uri` and `title` parameters.

Calling this method must be followed by a call to `onContentsBegin`. A corresponding call to `onLinkContentEnd` with the same `uri` value must eventually occur.

The behavior of this method is undefined, if `uri` is not present.

The behavior of this method is undefined, if `uri` is not a valid [URI reference](https://tools.ietf.org/html/rfc3986#section-4.1).

###### `onLinkContentEnd` {#api-handler-handler-onlincontentend}

A `Handler` object must have a method with signature `onLinkContentEnd(Uri uri, String? title)`.

This event represents the begin of a [*Markdom link content*] with the given value for the `uri` and `title` parameters.

A corresponding call to `onLinkContentBegin` with the same `uri` value and the same `title` value must have occurred. Calling this method must be followed by a call to `onContentEnd`.

The behavior of this method is undefined, if `uri` is not present.

The behavior of this method is undefined, if `uri` is not a valid [URI reference](https://tools.ietf.org/html/rfc3986#section-4.1).

###### `onListItemBegin` {#api-handler-handler-onlistitembegin}

A `Handler` object must have a method with signature `onListItemBegin()`.

This event represents the begin of a [*Markdom list item*].

Calling this method must be followed by a call to `onBlocksBegin`. A corresponding call to `onListItemEnd` must eventually occur.

###### `onListItemEnd` {#api-handler-handler-onlistitemend}

A `Handler` object must have a method with signature `onListItemEnd()`.

This event represents the end of a [*Markdom list item*].

A corresponding call to `onListItemBegin` must have occurred. Calling this method must be followed by a call to `onNextListItem` or `onListItemsEnd`.

###### `onListItemsBegin` {#api-handler-handler-onlistitemsbegin}

A `Handler` object must have a method with signature `onListItemsBegin()`.

This event represents the begin of a sequence of [*Markdom list items*].

Calling this method must be followed by a call to `onListItemBegin` or `onListItemsEnd`. A corresponding call to `onListItemsEnd` must eventually occur.

###### `onListItemsEnd` {#api-handler-handler-onlistitemsend}

A `Handler` object must have a method with signature `onListItemsEnd()`.

This event represents the end of a sequence of [*Markdom list items*].

A corresponding call to `onListItemsBegin` must have occurred. Calling this method must be followed by a call to `onOrderedListBlockEnd` or `onUnorderedListBlockEnd`.

###### `onNextBlock` {#api-handler-handler-onnextblock}

A `Handler` object must have a method with signature `onNextBlock()`.

This event represents the continuation of a sequence of [*Markdom blocks*].

Calling this method must be followed by a call to `onBlockBegin`.

###### `onNextContent` {#api-handler-handler-onnextcontent}

A `Handler` object must have a method with signature `onNextContent()`.

This event represents the continuation of a sequence of [*Markdom contents*].

Calling this method must be followed by a call to `onContentBegin`.

###### `onNextListItem` {#api-handler-handler-onnextlistitem}

A `Handler` object must have a method with signature `onNextListItem()`.

This event represents the continuation of a sequence of [*Markdom list items*].

Calling this method must be followed by a call to `onListItemBegin`.

###### `onOrderedListBlockBegin` {#api-handler-handler-onorderedlistblockbegin}

A `Handler` object must have a method with signature `onOrderedListBlockBegin(Integer startIndex)`.

This event represents the begin of an [*ordered Markdom list block*] with the given value for the `startIndex` parameter.

Calling this method must be followed by a call to `onListItemsBegin`. A corresponding call to `onOrderedListBlockEnd`  with the same `startIndex` value must eventually occur.

The behavior of this method is undefined, if `startIndex` is not present.

The behavior of this method is undefined, if `startIndex` is negative.

###### `onOrderedListBlockEnd` {#api-handler-handler-onorderedlistblockend}

A `Handler` object must have a method with signature `onOrderedListBlockEnd(Integer startIndex)`.

This event represents the end of an [*ordered Markdom list block*] with the given value for the `startIndex` parameter.

A corresponding call to `onOrderedListBlockBegin` wit the same `startIndex` value must have occurred. Calling this method must be followed by a call to `onBlockEnd`.

The behavior of this method is undefined, if `startIndex` is not present.

The behavior of this method is undefined, if `startIndex` is negative.

###### `onParagraphBlockBegin` {#api-handler-handler-onquoteblockbegin}

A `Handler` object must have a method with signature `onParagraphBlockBegin()`.

This event represents the begin of a [*Markdom paragraph block*].

Calling this method must be followed by a call to `onContentsBegin`. A corresponding call to `onParagraphBlockEnd` must eventually occur.

###### `onParagraphBlockEnd` {#api-handler-handler-onquoteblockend}

A `Handler` object must have a method with signature `onParagraphBlockEnd()`.

This event represents the end of a [*Markdom paragraph block*].

A corresponding call to `onParagraphBlockBegin` must have occurred. Calling this method must be followed by a call to `onBlockEnd`.

###### `onQuoteBlockBegin` {#api-handler-handler-onquoteblockbegin}

A `Handler` object must have a method with signature `onQuoteBlockBegin()`.

This event represents the begin of a [*Markdom quote block*].

Calling this method must be followed by a call to `onBlocksBegin`. A corresponding call to `onQuoteBlockEnd` must eventually occur.

###### `onQuoteBlockEnd` {#api-handler-handler-onquoteblockend}

A `Handler` object must have a method with signature `onQuoteBlockEnd()`.

This event represents the end of a [*Markdom quote block*].

A corresponding call to `onQuoteBlockBegin` must have occurred. Calling this method must be followed by a call to `onBlockEnd`.

###### `onTextContent` {#api-handler-handler-ontextcontent}

A `Handler` object must have a method with signature `onTextContent(String text)`.

This event represents a [*Markdom text content*] with the given value for the `text`.

Calling this method must be followed by a call to `onBlockEnd`.

The behavior of this method is undefined, if `text` is not present.

###### `onUnorderedListBlockBegin` {#api-handler-handler-onunorderedlistblockbegin}

A `Handler` object must have a method with signature `onUnorderedListBlockBegin()`.

This event represents the begin of an *unordered Markdom list block*.

Calling this method must be followed by a call to `onListItemsBegin`. A corresponding call to `onUnorderedListBlockEnd` must eventually occur.

###### `onUnorderedListBlockEnd` {#api-handler-handler-onunorderedlistblockend}

A `Handler` object must have a method with signature `onUnorderedListBlockEnd()`.

This event represents the end of an *unordered Markdom list block*.

A corresponding call to `onUnorderedListBlockBegin` must have occurred. Calling this method must be followed by a call to `onBlockEnd`.

#### Example Document {#api-handler-example}

The following succession of events describes the [example document](#example):

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
                onLinkContentBegin("#Bar", "")
                  onContentsBegin()
                    onContentBegin(TEXT)
                    onTextContent("Foo")
                    onContentEnd(TEXT)
                  onContentsEnd()
                onLinkContentEnd("#Bar", "")
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

Formats that are eligible to be used as a data exchange format must have a well known and publicly documented syntax that leaves no leeway for in ambiguity. Representation and interpretation of such formats are always possible. This specification defines representations of [*Markdom documents*] for some data exchange formats.

### JSON {#data-json}

[JSON](http://www.json.org/) is a  data exchange format. The following sections describe how to represent a [*Markdom document*] as a JSON document. Interpreting a JSON document as a a [*Markdom document*] is always possible, if the JSON document is well-formed and a valid representation of a [*Markdom document*].

#### Schema {#data-json-schema}

A [JSON](http://json-schema.org/documentation.html) schema for JSON documents that represent a [*Markdom document*] exists and is located at [`http://schema.markdom.io/markdom-1.0.json#`](http://schema.markdom.io/markdom-1.0.json#).

#### Document {#markup-json-document}

A [*Markdom document*] must be represented as a JSON object.

The JSON object may have an entry with name `$schema` and value `http://schema.markdom.io/markdom-1.0.json#"` that allows automated schema validation.

The JSON object must have an entry with name `version` and value `1.0`.

The JSON object may have an entry with name `blocks`. The value of this entry, if present, must be a JSON array that contains the representations of the [*Markdom blocks*] the represented [*Markdom document*] consists of.

#### Block {#markup-json-block}

##### Code Block {#markup-json-codeblock}

A [*Markdom code block*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `Code`.

The JSON object must have an entry with name `code`. The value of this entry, must be a JSON string and contain the value of the `code` parameter.

The JSON object may have an entry with name `hint`. The value of this entry, if present, must be a JSON string that contains the value of `hint` parameter or a JSON null, if the `hint` parameter is not present.

##### Comment Block {#markup-json-commentblock}

A [*Markdom comment block*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `Comment`.

The JSON object must have an entry with name `comment`. The value of this entry, must be a JSON string and contain the value of the `comment` parameter.

##### Division Block {#markup-json-divisionblock}

A [*Markdom division block*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `Division`.

##### Heading Block {#markup-json-headingblock}

A [*Markdom heading block*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `Heading`.

The JSON object must have an entry with name `level`. The value of this entry, must be a JSON number and contain the integer value of the `level` parameter.

The JSON object may have an entry with name `contents`. The value of this entry, if present, must be a JSON array that contains the representations of the [*Markdom contents*] the represented [*Markdom heading block*] consists of.

##### Ordered List Block {#markup-json-orderedlistblock}

An [*ordered Markdom list block*] must be represented as a JSON object.

The object must have an entry with name `type` and value `OrderedList`.

The object must have an entry with name `startIndex`. The value of this entry, must be a JSON number and contain the value of the `startIndex` parameter.

The JSON object may have an entry with name `items`. The value of this entry, if present, must be a JSON array that contains the representations of the [*Markdom list items*] the represented [*ordered Markdom list block*] consists of.

##### Paragraph Block {#markup-json-paragraphblock}

A [*Markdom paragraph block*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `Paragraph`.

The JSON object may have an entry with name `contents`. The value of this entry, if present, must be a JSON array that contains the representations of the [*Markdom contents*] the represented [*Markdom paragraph block*] consists of.

##### Quote Block {#markup-json-quoteblock}

A [*Markdom quote block*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `Quote`.

The JSON object may have an entry with name `blocks`. The value of this entry, if present, must be a JSON array that contains the representations of the [*Markdom contents*] the represented [*Markdom quote block*] consists of.

##### Unordered List Block {#markup-json-unorderedlistblock}

An [*unordered Markdom list block*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `UnorderedList`.

The JSON object may have an entry with name `items`. The value of this entry, if present, must be a JSON array that contains the representations of the [*Markdom list items*] the represented [*unordered Markdom list block*] consists of.

#### List Item {#markup-json-listitem}

A [*Markdom list item*] must be represented as a JSON object.

The JSON object may have an entry with name `blocks`. The value of this entry, if present, must be a JSON array that contains the representations of the [*Markdom blocks*] the represented [*Markdom list item*] consists of.

#### Content {#markup-json-content}

##### Code Content {#markup-json-codecontent}

A [*Markdom code content*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `Code`.

The JSON object must have an entry with name `code`. The value of this entry, must be a JSON string and contain the value of the `code` parameter.

##### Emphasis Content {#markup-json-emphasiscontent}

A [*Markdom emphasis content*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `Emphasis`.

The JSON object must have an entry with name `level`. The value of this entry, must be a JSON string and contain the integer value of the `level` parameter.

The JSON object may have an entry with name `contents`. The value of this entry, if present, must be a JSON array that contains the representations of the [*Markdom contents*] the represented [*Markdom emphasis content*] consists of.

##### Image Content {#markup-json-imagecontent}

A [*Markdom image content*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `Image`.

The JSON object must have an entry with name `uri`. The value of this entry, must be a JSON string and contain the value of the `uri` parameter.

The JSON object may have an entry with name `title`. The value of this entry, if present, must be a JSON string that contains the value of `title` parameter or a JSON null, if the `title` parameter is not present.

The JSON object may have an entry with name `alternative`. The value of this entry, if present, must be a JSON string that contains the value of `alternative` parameter or a JSON null, if the `alternative` parameter is not present.

##### Line Break Content {#markup-json-linebreakcontent}

A [*Markdom line break content*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `LineBreak`.

The JSON object must have an entry with name `hard`. The value of this entry, must be a JSON boolean and contain the value of the `hard` parameter.

##### Link Content {#markup-json-linkcontent}

A [*Markdom link content*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `Link`.

The JSON object must have an entry with name `uri`. The value of this entry, must be a JSON string and contain the value of the `uri` parameter.

The JSON object may have an entry with name `title`. The value of this entry, if present, must be a JSON string that contains the value of `title` parameter or a JSON null, if the `title` parameter is not present.

The JSON object may have an entry with name `contents`. The value of this entry, if present, must be a JSON array that contains the representations of the [*Markdom contents*] the represented [*Markdom link content*] consists of.

##### Text Content {#markup-json-textcontent}

A [*Markdom text content*] must be represented as a JSON object.

The JSON object must have an entry with name `type` and value `Text`.

The JSON object must have an entry with name `text`. The value of this entry, must be a JSON string and contain the value of the `text` parameter.

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

[YAML](http://yaml.org/) is a data exchange format that is a superset of JSON. The description of[JSON representation](#data-json) is therefore also a description for YAML representations and can be used as such.

Any JSON document that represents a [*Markdom document*] is also a YAML document that represents the [*Markdom document*]. Interpreting a YAML document as a a [*Markdom document*] is always possible, if the YAML document is well-formed and a valid representation of a [*Markdom document*].

#### Tags {#data-tag}

It would be possible to use a YAML specific parts of the YAML specification that are not compatible with JSON. [Tags](http://www.yaml.org/spec/1.2/spec.html#id2784064) could be used to distinguish between the polymorphic [*Markdom blocks*] and [*Markdom contents*] instead of the `type` entries, but this would defeat compatibility between JSON and YAML representations.

#### Anchors {#data-anchor}

It is possible, but not very useful and not recommended, to use [anchors and aliases](http://www.yaml.org/spec/1.2/spec.html#id2785586) in a YAML representation of a [*Markdom document*].

As long as the data structure is still a tree, is should be possible to interpret it as a [*Markdom document*]. If the data structure is an acyclic graph, is may be possible to interpret it as a [*Markdom document*].

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

[XML](https://www.w3.org/XML/) is a markup language that is commonly used as a data exchange format. The following sections describe how to represent a [*Markdom document*] as a XML document. Interpreting a XML document as a a [*Markdom document*] is always possible, if the XML document is well-formed and a valid representation of a [*Markdom document*].

#### Schema {#data-json-schema}

A [XML Schema](https://www.w3.org/TR/xmlschema-1/) for XML documents that represent a [*Markdom document*] exists and is located at [`http://schema.markdom.io/markdom-1.0.xsd`](http://schema.markdom.io/markdom-1.0.xsd).

#### Namespace {#data-json-namespace}

The XML namespace used for a XML representation of a [*Markdom document*] is `http://schema.markdom.io/markdom-1.0.xsd`.

#### Document {#markup-xml-document}

A [*Markdom document*] is represented as a XML element with local name `Document`.

The XMl element must have an attribute with name `version` and value `1.0`.

The child node list of the XML element must contain the representations of the [*Markdom blocks*] the represented [*Markdom document*] consists of and may contain text nodes that contain only whitespace.

#### Block {#markup-xml-block}

##### Code Block {#markup-xml-codeblock}

A [*Markdom code block*] is represented as a XML element with local name `Code`.

The XML element may have an attribute with name `hint`. The value of this attribute, if present, must  be the value of `hint` parameter.

The child node list of the XML element may be empty if the value of the `code` parameter is the empty string, otherwise it must contain a text node that contains the value of the `code` parameter.

##### Comment Block {#markup-xml-commentblock}

A [*Markdom comment block*] is represented as a XML element with local name `Comment`.

The child node list of the XML element may be empty if the value of the `comment` parameter is the empty string, otherwise it must contain a text node that contains the value of the `comment` parameter.

##### Division Block {#markup-xml-divisionblock}

A [*Markdom division block*] is represented as a XML element with local name `Division`.

The child node list of the XML element may contain text nodes that contain only whitespace.

##### Heading Block {#markup-xml-headingblock}

A [*Markdom heading block*] is represented as a XML element with local name `Heading`.

The XML element must have an attribute with name `level`. The value of this attribute must be the integer  value of `level` parameter.

The child node list of the XML element must contain the representations of the [*Markdom content*] the represented [*Markdom heading block*] consists of and may contain text nodes that contain only whitespace.

##### Ordered List Block {#markup-xml-orderedlistblock}

An [*ordered Markdom list block*] is represented as a XML element with local name `OrderedList`.

The XML element must have an attribute with name `startIndex`. The value of this attribute must be the value of `startIndex` parameter.

The child node list of the XML element must contain the representations of the [*Markdom list items*] the represented [**ordered Markdom list block*] consists of and may contain text nodes that contain only whitespace.

##### Paragraph Block {#markup-xml-paragraphblock}

A [*Markdom paragraph block*] is represented as a XML element with local name `Paragraph`.

The child node list of the XML element must contain the representations of the [*Markdom content*] the represented [*Markdom paragraph block*] consists of and may contain text nodes that contain only whitespace.

##### Quote Block {#markup-xml-quoteblock}

A [*Markdom quote block*] is represented as a XML element with local name `Quote`.

The child node list of the XML element must contain the representations of the [*Markdom blocks*] the represented [*Markdom quote block*] consists of and may contain text nodes that contain only whitespace.

##### Unordered List Block {#markup-xml-unorderedlistblock}

An [*unordered Markdom list block*] is represented as a XML element with local name `UnorderedList`.

The child node list of the XML element must contain the representations of the [*Markdom list items*] the represented [*unordered Markdom list block*] consists of and may contain text nodes that contain only whitespace.

#### List Item {#markup-xml-listitem}

A [*Markdom list item*] is represented as a XML element with local name `ListItem`.

The child node list of the XML element must contain the representations of the [*Markdom blocks*] the represented [*Markdom list item*] consists of and may contain text nodes that contain only whitespace.

#### Content {#markup-xml-content}

##### Code Content {#markup-xml-codecontent}

A [*Markdom code content*] is represented as a XML element with local name `Code`.

The child node list of the XML element may be empty if the value of the `code` parameter is the empty string, otherwise it must contain a text node that contains the value of the `code` parameter.

##### Emphasis Content {#markup-xml-emphasiscontent}

A [*Markdom emphasis content*] is represented as a XML element with local name `Emphasis`.

The XML element must have an attribute with name `level`. The value of this attribute must be the integer value of `level` parameter.

The child node list of the XML element must contain the representations of the [*Markdom contents*] the represented [*Markdom emphasis content*] consists of and may contain text nodes that contain only whitespace.

##### Image Content {#markup-xml-imagecontent}

A [*Markdom image content*] is represented as a XML element with local name `Image`.

The XML element must have an attribute with name `uri`. The value of this attribute must be the value of `uri` parameter.

The XML element may have an attribute with name `title`. The value of this attribute, if present, must  be the value of `title` parameter.

The XML element may have an attribute with name `alternative`. The value of this attribute, if present, must  be the value of `alternative` parameter.

The child node list of the XML element may contain text nodes that contain only whitespace.

##### Line Break Content {#markup-xml-linebreakcontent}

A [*Markdom line break content*] is represented as a XML element with local name `LineBreak`.

The XML element must have an attribute with name `hard`. The value of this attribute must be `true` or `false`, depending on the value of `hard` parameter.

The child node list of the XML element may contain text nodes that contain only whitespace.

##### Link Content {#markup-xml-linkcontent}

A [*Markdom link content*] is represented as a XML element with local name `Link`.

The XML element must have an attribute with name `uri`. The value of this attribute must be the value of `uri` parameter.

The XML element may have an attribute with name `title`. The value of this attribute, if present, must  be the value of `title` parameter.

The child node list of the XML element must contain the representations of the [*Markdom contents*] the represented [*Markdom link content*] consists of and may contain text nodes that contain only whitespace.

##### Text Content {#markup-xml-textcontent}

A [*Markdom text content*] is represented as a XML element with local name `Text`.

The child node list of the XML element may be empty if the value of the `text` parameter is the empty string, otherwise it must contain a text node that contains the value of the `text` parameter.

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

## Markup representations {#markup}

Because Markdom has been designed with the intent that the largest possible number of applications can produce a reasonable output, it should generally be possible to represent a [*Markdom document*] in a given rich text or markup language. This includes lightweight markup languages that use formatting instructions that closely resemble their intended purpose (e.g. CommonMark, ...), structural formatting languages (e.g. HTML, ...) and procedural formatting languages (e.g. LaTeX).

Interpreting a markup language as a [*Markdom document*] is generally not possible, unless the markup language has a set of formatting instructions that is similar to that of Markdom (e.g. CommonMark, ...).

### CommonMark {#markup-cm}

*This section discusses [CommonMark 0.26](http://spec.commonmark.org/0.26/).*

Markdom was designed with CommonMark in mind. It is possible to represent any [*Markdom document*] as CommonMark text with minimal changes, if any. CommonMark has direct support for all formatting instructions used in Markdom.

#### Document {#markup-cm-document}

No special considerations are necessary to represent a [*Markdom document*].

#### Block {#markup-cm-block}

##### Code Block {#markup-cm-codeblock}

It is recommended to represent [*Markdom code blocks*] as [fenced code blocks](http://spec.commonmark.org/0.26/#code-fence), because these work properly in any context.

Using an [indented code block](http://spec.commonmark.org/0.26/#indented-code-block) might yield a problem if a [*Markdom code block*] follows a complex block and the `code` parameter starts with whitespace. Fenced code block also allow the value of the `hint` parameter, if present, to be used.

Assume an [*unordered Markdom list block*] that is followed by a [*Markdom code block*]. [This](http://spec.commonmark.org/dingus/?text=*%20foo%0A%0A%60%60%60%0A%09bar%0Abaz%0A%60%60%60) representation yields the expected output:


    * foo

    ```
    	bar
    baz
    ```

[This](http://spec.commonmark.org/dingus/?text=*%20foo%0A%0A%09%09bar%0A%09baz) representation doesn't yield the expected output:

```
* foo

		bar
	baz
```

Representing a [*Markdom code block*] as a fenced code block is always possible. If the `code` parameter contains lines that consist of nothing but a succession of `BACKTICK` (` ` `) characters, the fence must be elongated accordingly:

    ````
    ```
    ````

[Control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters other then `LINE_FEED` (`\n`) or `CHARACTER_TABULATION` (`\t`) should be removed or replaced.

##### Comment Block {#markup-cm-codeblock}

A [*comment division block*] should be represented as a [HTML blocks](http://spec.commonmark.org/0.26/#html-blocks) containing a HTML comment.

[Control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters other then `LINE_FEED` (`\n`) or `CHARACTER_TABULATION` (`\t`) should be removed or replaced. Any appearance of the closing HTML comment delimiter (`-->`) must be modified, replaced or ignored.

##### Division Block {#markup-cm-divisionblock}

A [*Markdom division block*] should be represented as a [thematic break](http://spec.commonmark.org/0.26/#thematic-break).

##### Heading Block {#markup-cm-headingblock}

It is recommended to represent [*Markdom heading blocks*] as [ATX headings](http://spec.commonmark.org/0.26/#atx-heading), because these work properly in any context.

Using a [setext heading](http://spec.commonmark.org/0.26/#setext-headings) generally introduces inconsistency if  the `level` parameter of a [*Markdom heading block*] is larger than `2`.

[*Markdom line break contents*], contained in the contents a [*Markdom heading block*] consitst of, should be ignored and, if necessary, replaced with a single space character.

##### Ordered List Block {#markup-cm-orderedlistblock}

An [*ordered Markdom list block*] should be represented as a [ordered list](http://spec.commonmark.org/0.26/#ordered-list).

Because CommonMark doesn't support adjacent lists, it is necessary to place another [block](http://spec.commonmark.org/0.26/#blocks) between two adjacent [*ordered Markdom list block*] or [*unordered Markdom list block*]. A [*Markdom division block*] is recommended, because it introduced the least amount of clutter and is still used in its intended purpose (other than, for instance, a paragraph containing a zero width space character).

##### Paragraph Block {#markup-cm-paragraphblock}

A [*Markdom paragraph block*] should be represented as a [paragraph](http://spec.commonmark.org/0.26/#paragraphs).

##### Quote Block {#markup-cm-quoteblock}

A [*Markdom quote block*] should be represented as a [block quote](http://spec.commonmark.org/0.26/#block-quotes).

##### Unordered List Block {#markup-cm-unorderedlistblock}

An [*unordered Markdom list block*] should be represented as a [bullet list](http://spec.commonmark.org/0.26/#bullet-list).

#### List Item {#markup-cm-listitem}

A [*Markdom list items*] should be represented as a [list item](http://spec.commonmark.org/0.26/#list-items).

#### Content {#markup-cm-content}

##### Code Content {#markup-cm-codecontent}

A [*Markdom code content*] should be represented as a [code span](http://spec.commonmark.org/0.26/#code-spans).

Representing a [*Markdom code content*] as a code span is always possible. If the value of the `code` parameter contains succession of `BACKTICK` (` ` `) characters, the fence must be elongated accordingly:

    ``foo`bar``

If the value of the `code` parameter begins or ends with a `BACKTICK` (` ` `) character, leading or trailing whitespace can be added:

    `` `foo` ``

[Control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters in the value of the `code` parameter should be removed or replaced. `LINE_FEED` (`\n`) or `CHARACTER_TABULATION` (`\t`) should be treated as a single space character.

##### Emphasis Content {#markup-cm-emphasiscontent}

A [*Markdom emphasis content*] should be represented as an [emphasis or strong emphasis](http://spec.commonmark.org/0.26/#emphasis-and-strong-emphasis), depending on the value of the `level` parameter.

Consequent usage of `*` or `**` instead of `_` or `__` is recommended as the emphasis delimiter, because it simplifies the correct representation of the [*Markdom emphasis content*] if it is not surrounded by whitespace.

A [*Markdom emphasis content*] can be ignored if it contains nothing or only the empty string.

##### Image Content {#markup-cm-imagecontent}

A [*Markdom image content*] should be represented as an [image](http://spec.commonmark.org/0.26/#images).

[Control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters in the value of the `title` parameter, if present, should be removed or replaced. `LINE_FEED` (`\n`) or `CHARACTER_TABULATION` (`\t`) should be treated as a single space character.

The value of the `alternative` parameter, if present, should be used as if it was the `text` parameter of a [*Markdom text content*] and serve as the [image description](http://spec.commonmark.org/0.26/#image-description).

##### Line Break Content {#markup-cm-linebreakcontent}

A [*Markdom line break content*] should be represented as a [hard line break](http://spec.commonmark.org/0.26/#hard-line-breaks) or a [soft line break](http://spec.commonmark.org/0.26/#hard-line-breaks), depending on the value of the `hard` parameter.

##### Link Content {#markup-cm-linkcontent}

A [*Markdom link content*] should be represented as a [link](http://spec.commonmark.org/0.26/#links).

[Control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters in the value of the `title` parameter, if present, should be removed or replaced. `LINE_FEED` (`\n`) or `CHARACTER_TABULATION` (`\t`) should be treated as a single space character.

A [*Markdom link content*] that contains nothing or only the empty string should use the value of the `uri` parameter as if it was the `text` parameter of a [*Markdom text content*] and serve as the [link description](http://spec.commonmark.org/0.26/#link-destination). The [*Markdom link content*] can be ignored if the value of the `uri` parameter is the empty string and the `title` parameter is not present or the empty string.

##### Text Content {#markup-cm-textcontent}

A [*Markdom text content*] should be represented as [textual content](http://spec.commonmark.org/0.26/#textual-content).

[Control](http://www.fileformat.info/info/unicode/category/Cc/list.htm) characters in the value of the `text` parameter should be removed or replaced. `LINE_FEED` (`\n`) or `CHARACTER_TABULATION` (`\t`) should be treated as a single space character).

#### Adjacent lists {#markup-cm-interpretation-whitespace-adjacentlists}

Because CommonMark doesn't support adjacent lists, it is necessary to place another [block](http://spec.commonmark.org/0.26/#blocks) between two adjacent [*ordered Markdom list block*] or [*unordered Markdom list block*]. A [*Markdom comment block*] or a [*Markdom division block*] is recommended, because it introduced the least amount of clutter and is still used in its intended purpose (other than, for instance, a paragraph containing a zero width space character).

#### Whitespace handling {#markup-cm-interpretation-whitespace}

##### Whitespace compression {#markup-cm-interpretation-whitespace-compression}

Consecutive whitespace characters should be compressed to a single space character.

##### Whitespace removal {#markup-cm-interpretation-whitespace-removal}

The representation of the content of a [*Markdom paragraph block*] should not begin with whitespace, because the additional indentation can lead to unintended side effects.

Assume an [*unordered Markdom list block*] that is followed by a [*Markdom paragraph block*] with content that starts with two space characters. [This](http://spec.commonmark.org/dingus/?text=*%20foo%0A%0Abar) representation yields the expected output:

    * foo

    bar

[This](http://spec.commonmark.org/dingus/?text=*%20foo%0A%0A%20%20bar) representation doesn't yield the expected output:

    * foo

      bar

For consistency, whitespace that follows a [*Markdom line break content*] should also be removed.

##### Whitespace shift {#markup-cm-interpretation-whitespace-shift}

Because of manner the [delimiter run](http://spec.commonmark.org/0.26/#delimiter-run) works, it is not possible for the content of a [*Markdom emphasis content*] or a [*Markdom link content*] to begin or end with a whitespace character. It is therefore recommended to shift whitespace to the surrounding of the [*Markdom content*].

Assuming a [*Markdom emphasis content*] contains a [*Markdom text content*] with `⎵bar⎵` (`⎵` indicates a space character) as the value, surrounded by other [*Markdom text contents*]. This shouldn't be represented as

    `foo* bar *baz`

but should be corrected to

    `foo *bar* baz`


A more complex example with nested [*Markdom contents*] and whitespace characters that should be compressed is

    `foo* [ ** lorem **ipsum ](#bar) *baz`

which should be corrected to

    `foo *[**lorem** ipsum](#bar)* baz`

#### Interpretation {#markup-cm-interpretation}

It is generally possible to interpret CommonMark text as a [*Markdom document*] as long as an actual interpreter is available, that produces a programmatically processable output (e.g. a domain model, a walker or visitor, or events). The mapping of CommonMark formatting instructions to [*Markdom nodes*] is (almost) straight forward because the structure of a [*Markdom document*] has no special edge cases that need to be considered.

##### HTML  {#markup-cm-interpretation-html}

CommonMark allows [HTML blocks](http://spec.commonmark.org/0.26/#html-blocks) and [raw HTML](http://spec.commonmark.org/0.26/#raw-html) as part of a CommonMark document while Markdom explicitly doesn't.

A HTML block containing only a HTML comment should be interpreted as a [*Markdom comment block*].

How to handle a CommonMark document that contains any other HTML is application dependent. Possibilities include
* rejecting the CommonMark document (e.g. display an error message to the editor),
* treat the raw HTML as a [*Markdom code content*] or [*Markdom text content*], or
* removing the HTML altogether.

##### Image description {#markup-cm-interpretation-imagedescription}

CommonMark allows arbitrary [inline](http://spec.commonmark.org/0.26/#inline) elements as an [image description](http://spec.commonmark.org/0.26/#image-description) while Markdom only allows plain text as the value of the `alternative` attribute of a [*Markdom image content*].

The image description should be reduced to plain text as if it would be used as the `alt` attribute of an `img` element when converting the CommonMark document to HTML.

#### Example Document {#markup-cm-example}

The following CommonMark document represents the [example document](#example):

    # Markdom
    
    1.
       [Foo](#Bar)
    2.
       Lorem ipsum
       `dolor sit amet`
    3.
       > *Baz*
    
    ```
    goto 11
    ```

### HTML {#markup-html}

Representing a [*Markdom document*] as a HTML document is always possible. HTML has direct support for all formatting instructions used in Markdom.

#### Document {#markup-html-document}

Depending on the application, a [*Markdom document*] should be represented as the node list that contains the representations of the [*Markdom blocks*] the [*Markdom document*] consists of and may be enclosed by arbitrary HTML markup up to the point where the representation is an entire HTML document.

#### Block {#markup-html-block}

##### Code Block {#markup-html-codeblock}

A [*Markdom code block*] should be represented as a `<code>` element nested into a `<pre>` element.

##### Comment Block {#markup-html-commentblock}

A [*Markdom comment block*] should be represented as a HTML comment or ignored, depending of the actual application.

##### Division Block {#markup-html-divisionblock}

A [*Markdom division block*] should be represented as a `<hr>` element.

##### Heading Block {#markup-html-headingblock}

A [*Markdom heading block*] should be represented as a `<h1>` to `<h6>` element, depending on the value of the `level` parameter.

##### Ordered List Block {#markup-html-orderedlistblock}

An [*ordered Markdom list block*] should be represented as an `<ol>` element with the value of the `startIndex` parameter as its `start` attribute.

##### Paragraph Block {#markup-html-paragraphblock}

A [*Markdom paragraph block*] should be represented as a `<p>` element.

##### Quote Block {#markup-html-quoteblock}

A [*Markdom quote block*] should be represented as a `<blockquote>` element.

##### Unordered List Block {#markup-html-unorderedlistblock}

An [*unordered Markdom list block*] should be represented as an `<ul>` element.

#### List Item {#markup-html-listitem}

A [*Markdom list item*] should be represented as a `<li>` element.

#### Content {#markup-html-content}

##### Code Content {#markup-html-codecontent}

A [*Markdom code content*] should be represented as a `<code>` element with the value of the `code` parameter as its text content.

##### Emphasis Content {#markup-html-emphasiscontent}

A [*Markdom emphasis content*] should be represented as a `<em>` element or a `<strong>` element, depending on the value of the `level` parameter.

##### Image Content {#markup-html-imagecontent}

A [*Markdom image content*] should be represented as an `<img>` element with the value of the `uri` parameter as its `href` attribute, the value of the `title` parameter, if present, as its `title` attribute and the value of the `alternative` parameter, if present, as its `alt` attribute.

##### Line Break Content {#markup-html-linebreakcontent}

A [*Markdom line break content*] should be represented as a `<br>` element if the value of the `hard` attribute is `true` and ignored otherwise.

##### Link Content {#markup-html-linkcontent}

A [*Markdom link content*] should be represented as an `<a>` element with the value of the `uri` parameter as its `href` attribute and the value of the `title` parameter, if present, as its `title` attribute.

##### Text Content {#markup-html-textcontent}

A [*Markdom text content*] should be represented as a text node with the value of the `text` parameter as its content.

#### Interpretation {#markup-html-interpretation}

It is generally not simple or even possible to interpret a given HTML document as a [*Markdom document*].

#### Example Document {#markup-html-example}

The following HTML 5 document represents the [example document](#example):

```
<!doctype html>
<html>
  <head>
    <title>Example</title>
  </head>
  <body>
    <h1>Markdom</h1>
    <ol start="1">
      <li>
        <p>
          <a href="#Bar">Foo</a>
        </p>
      </li>
      <li>
        <p>Lorem ipsum<br>
          <code>dolor sit amet</code>
        </p>
      </li>
      <li>
        <blockquote>
          <p>
            <em>Baz</em>
          </p>
        </blockquote>
      </li>
    </ol>
    <pre><code>goto 11</code></pre>
  </body>
</html>
```

The following XHTML 5 document represents the [example document](#example):

```
<?xml version="1.0" encoding="UTF-8" standalone="no"?>
<!DOCTYPE html>
<html xmlns="http://www.w3.org/1999/xhtml">
  <head>
    <title>Example</title>
  </head>
  <body>
    <h1>Markdom</h1>
    <ol start="1">
      <li>
        <p>
          <a href="#Bar">Foo</a>
        </p>
      </li>
      <li>
        <p>Lorem ipsum<br/>
          <code>dolor sit amet</code>
        </p>
      </li>
      <li>
        <blockquote>
          <p>
            <em>Baz</em>
          </p>
        </blockquote>
      </li>
    </ol>
    <pre><code>goto 11</code></pre>
  </body>
</html>
```

[*Markdom node*]: #domain-node
[*Markdom nodes*]: #domain-node
[*Markdom document*]: #domain-document
[*Markdom documents*]: #domain-document
[*Markdom block*]: #domain-block
[*Markdom blocks*]: #domain-block
[*Markdom code block*]: #domain-codeblock
[*Markdom code blocks*]: #domain-codeblock
[*Markdom comment block*]: #domain-commentblock
[*Markdom comment blocks*]: #domain-commentblock
[*Markdom division block*]: #domain-divisionblock
[*Markdom heading block*]: #domain-headingblock
[*Markdom heading blocks*]: #domain-headingblock
[*ordered Markdom list block*]: #domain-orderedlistblock
[*Markdom paragraph block*]: #domain-paragraphblock
[*Markdom quote block*]: #domain-quoteblock
[*unordered Markdom List block*]: #domain-unorderedlistblock
[*Markdom list item*]: #domain-listitem
[*Markdom list items*]: #domain-listitem
[*Markdom content*]: #domain-content
[*Markdom contents*]: #domain-content
[*Markdom code content*]: #domain-codecontent
[*Markdom emphasis content*]: #domain-emphasiscontent
[*Markdom image content*]: #domain-imagecontent
[*Markdom line break content*]: #domain-linebreakcontent
[*Markdom line break contents*]: #domain-linebreakcontent
[*Markdom link content*]: #domain-linkcontent
[*Markdom text content*]: #domain-textcontent
[*Markdom text contents*]: #domain-textcontent
[*Markdom event*]: api-handler-event
[*Markdom events*]: api-handler-event

[`BlockType`]: #api-common-blocktype
[`ContentType`]: #api-common-contenttype
[`EmphasisLevel`]: #api-common-emphasislevel
[`HeadingLevel`]: #api-common-headinglevel
[`BlockParentType`]: #api-dom-blockparenttype
[`ContentParentType`]: #api-dom-contentparenttype
[`NodeType`]: #api-dom-nodetype
[`ListBlockType`]: #api-dom-listblocktype
[`Block`]: #api-dom-block
[`BlockParent`]: #api-dom-blockparent
[`CodeBlock`]: #api-dom-codeblock
[`CodeContent`]: #api-dom-codecontent
[`Content`]: #api-dom-content
[`ContentParent`]: #api-dom-contentparent
[`ContentParentBlock`]: #api-dom-contentparentblock
[`ContentParentContent`]: #api-dom-contentparentcontent
[`DivisionBlock`]: #api-dom-divisionblock
[`Document`]: #api-dom-document
[`EmphasisContent`]: #api-dom-emphasiscontent
[`HeadingBlock`]: #api-dom-headingblock
[`ImageContent`]: #api-dom-imagecontent
[`LineBreakContent`]: #api-dom-linebreakcontent
[`LinkContent`]: #api-dom-linkcontent
[`ListBlock`]: #api-dom-listblock
[`ListItem`]: #api-dom-listitem
[`Node`]: #api-dom-node
[`OrderedListBlock`]: #api-dom-orderedlistblock
[`ParagraphBlock`]: #api-dom-paragraphblock
[`QuoteBlock`]: #api-dom-quoteblock
[`TextContent`]: #api-dom-textcontent
[`UnorderedListBlock`]: #api-dom-unorderedlistblock
[`Handler`]: #api-handler-handler
[`Dispatcher`]: #api-handler-dispatcher
[`Sequence`]: #api-common-sequence
[`Iterable`]: #api-common-iterable
