__To `conceptual-model-specification` make the following changes:__

----

__add to 1.3.3__

> A "set" is defined as an unordered collection of data instances containing no duplicate data instances.
> When a property is defined in terms of a "list", the data type of the data instances in the list is also provided
>
> An "int" is defined as an integer value; its storage is not specified, but must be able to store at least all integers in the interval \[−1, 32767\]

----

__add to 2. a new subsection 2.8, as follows__


<a name="reasoning"/>

## 2.8 the "Reasoning" Data Type

The `Reasoning` data type describes the details of a reasoning network
used to develop and support the possible version of the past described by the 
`http://gedcomx.org/v1/Conclusion`s in the data.

### identifier

The identifier for the `Reasoning` data type is

`http://gedcomx.polygenea.org/v1/Reasoning`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
subjects | The subjects of study during research | List of [`http://gedcomx.polygenea.org/v1/SubjectNode`](#subject-node).  Order is preserved. | OPTIONAL.
properties | The information about single things encountered during research | List of [`http://gedcomx.polygenea.org/v1/Property`](#property).  Order is preserved. | OPTIONAL.
connections | The information about pairs things encountered during research | List of [`http://gedcomx.polygenea.org/v1/Connection`](#connection).  Order is preserved. | OPTIONAL.
tags | Metadata asserted about various other claims during research | List of [`http://gedcomx.polygenea.org/v1/Tag`](#tag).  Order is preserved. | OPTIONAL.
aggregates | Semantically-merged collections of subjects | List of [`http://gedcomx.polygenea.org/v1/AggregatedSubject`](#aggregated-subject).  Order is preserved. | OPTIONAL.
derivations | Free-text descriptions of reasoning processes | List of [`http://gedcomx.polygenea.org/v1/Derivation`](#derivation).  Order is preserved. | OPTIONAL.
inferences | Structured descriptions of reasoning processes | List of [`http://gedcomx.polygenea.org/v1/Inference`](#inference).  Order is preserved. | OPTIONAL.
expectations | Structured descriptions of trends, inference rules, and other guidelines about what a researcher expects to be true | List of [`http://gedcomx.polygenea.org/v1/Expectation`](#expectation).  Order is preserved. | OPTIONAL.
attribution | The attribution of this reasoning. | `http://gedcomx.org/Attribution` | OPTIONAL. If not provided, the attribution of the containing data set (e.g., file) of the `Reasoning` is assumed.


----

__Modify section 3.10, subsection "properties", adding the following row to the table:__

name  | description | data type | constraints
------|-------------|-----------|------------
supportingClaims | The list of references to the reasoning elements that support this conclusion. | List of [`http://gedcomx.polygenea.org/v1/ClaimReference`](#claim-reference). Order is preserved. | OPTIONAL.
refutingClaims | The list of references to the reasoning elements that contradict this conclusion. | List of [`http://gedcomx.polygenea.org/v1/ClaimReference`](#claim-reference). Order is preserved. | OPTIONAL.


----

__Add to 3. several new subsections, as follows__


<a name="reasoning-node"/>

## 3.22 The "ReasoningNode" Data Type

The `ReasoningNode` data type defines the abstract concept of a single atomic piece of the reasoning process.

### identifier

The identifier for the `ReasoningNode` data type is:

`http://gedcomx.polygenea.org/v1/ReasoningNode`


### properties

name  | description | data type | constraints
------|-------------|-----------|------------
id | An internal identifier for the data structure holding the reasoning data. | string | REQUIRED.  The id is to be used as a "fragment identifier" as defined by [RFC 3986, Section 3.5](http://tools.ietf.org/html/rfc3986#section-3.5). As such, the constraints of the id are provided in the definition of the media type (e.g. XML, JSON) of the data structure.
attribution | The attribution of this subject. | `http://gedcomx.org/Attribution` | OPTIONAL. If not provided, the attribution of the containing data set (i.e., `Reasoning`) of the `ReasoningNode` is assumed.

It is RECOMMENDED that the `id` of each `ReasoningNode` be the (upper-case) initial letter of the name of the `ReasoningNode`s element type (which will be one of {"A", "C", "D", "E", "I", "P", "S", "T"}),  followed by the (non-padded) decimal string representation of 0-based index of the `ReasoningNode` within its containing element.


<a name="claim"/>

## 3.23 The "Claim" Data Type

The `Claim` data type defines the abstract concept of a research _claim_, either made by a source or derived by a researcher.

### identifier

The identifier for the `Claim` data type is:

`http://gedcomx.polygenea.org/v1/Claim`

### extension

This data type extends the following data type:

`http://gedcomx.polygenea.org/v1/ReasoningNode`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
source | The attribution of this subject. | URI | REQUIRED. MUST resolve to an instance of one of the following types: `http://gedcomx.org/v1/SourceDescription`, [`http://gedcomx.polygenea.org/v1/Inference`](#inference), or [`http://gedcomx.polygenea.org/v1/Derivation`](#derivation).


<a name="claim-reference"/>

## 3.24 The "ClaimReference" Data Type

The `ClaimReference` data type defines a reference to a [`Claim`](#claim).

### identifier

The identifier for the "ClaimReference" data type is:

`http://gedcomx.polygenea.org/v1/ClaimReference`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
claim  | Reference to a `Claim`. | [URI](#uri) | REQUIRED. MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/Claim`](#claim).
attribution | The attribution of this source reference. | `http://gedcomx.org/Attribution` | OPTIONAL. If not provided, the attribution of the containing resource of the claim reference is assumed.


<a name="subject-node"/>

## 3.25 The "Subject Node" Data Type

The `SubjectNode` data type defines the abstract concept of a genealogical _subject_ attested by a single source.

The related `http://gedcomx.org/v1/Subject` data type represents something with unique and intrinsic identity, and only one `Subject` is expected to exist for any given entity.
The `http://gedcomx.polygenea.org/v1/SubjectNode` data type represents one `Subject` referenced in one source or one part of one source;
there are generally many `SubjectNode`s for each `Subject`, one for each reference to that subject known to exist.

### identifier

The identifier for the `SubjectNode` data type is:

`http://gedcomx.polygenea.org/v1/SubjectNode`

### extension

This data type extends the following data type:

`http://gedcomx.polygenea.org/v1/Claim`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
slug | Something to distinguish this subject from other subjects in the same source. | string | REQUIRED. 


<a name="property"/>

## 3.26 The "Property" Data Type

The `Property` data type defines a string-valued detail about a `ResearchNode`, most commonly a `SourceNode`.

The `http://gedcomx.polygenea.org/v1/Property` data type is approximately the research parallal to the `http://gedcomx.org/v1/Fact` data type in conclusions, but also serves additional purposes, such as identifying the "type" of a `SubjectNode`.

### identifier

The identifier for the `Property` data type is:

`http://gedcomx.polygenea.org/v1/Property`

### extension

This data type extends the following data type:

`http://gedcomx.polygenea.org/v1/Claim`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
key | Enumerated value identifying the type of property | Enumerated Value | REQUIRED. MUST identify a property type, and use of a [known property type](#known-property-types) is RECOMMENDED.
of | A reference to the `ReasoningNode` to which this property applies | URI | REQUIRED. MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/ReasoningNode`](#reasoning-node).
value | The value of the property. | string | REQUIRED.


<a name="connection"/>

## 3.27 The "Connection" Data Type

The `Connection` data type defines a directed pair-wise relationship between two  `ResearchNode`s.

The `http://gedcomx.polygenea.org/v1/Connection` data type is approximately the research parallal to the `http://gedcomx.org/v1/Relationship` and `http://gedcomx.org/v1/EventRole` data types in conclusions; but also serves additional purposes, such as identifying one node as the "update" of another.

### identifier

The identifier for the `Connection` data type is:

`http://gedcomx.polygenea.org/v1/Connection`

### extension

This data type extends the following data type:

`http://gedcomx.polygenea.org/v1/Claim`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
key | Enumerated value identifying the type of connection | Enumerated Value | REQUIRED. MUST identify a connection type, and use of a [known connection type](#known-connection-types) is RECOMMENDED.
of | A reference to the `ReasoningNode` to which this connection applies | URI | REQUIRED. MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/ReasoningNode`](#reasoning-node).
value | The value of the the connection | URI | REQUIRED. MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/ReasoningNode`](#reasoning-node).


<a name="tag"/>

## 3.28 The "Tag" Data Type

The `Tag` data type defines metadata asserted about various other claims during research.

Each tag type has a significantly different meaning; see the list of [known tag types](#known-tag-types) for details.

### identifier

The identifier for the `Tag` data type is:

`http://gedcomx.polygenea.org/v1/Tag`

### extension

This data type extends the following data type:

`http://gedcomx.polygenea.org/v1/Claim`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
type | Enumerated value identifying the type of connection | Enumerated Value | REQUIRED. MUST identify a connection type, and use of a [known tag type](#known-tag-types) is RECOMMENDED.
of | A list of references to jointly-tagged `ReasoningNode`s | List of URI.  Order is not preserved | REQUIRED. Each URI MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/ReasoningNode`](#reasoning-node).  Additional constraints may be added for particular tag `type`s.


<a name="aggregated-subject"/>

## 3.29 The "Aggregated Subject" Data Type

The `AggregatedSubject` data type is a semantic placeholder for the idea "the subject you get if you assume that all of these other subjects are the same".

Every `http://gedcomx.org/v1/Subject` that has two or more sources is conceptually an `http://gedcomx.polygenea.org/v1/AggregatedSubject`.

### identifier

The identifier for the `AggregatedSubject` data type is:

`http://gedcomx.polygenea.org/v1/AggregatedSubject`

### extension

This data type extends the following data type:

`http://gedcomx.polygenea.org/v1/ResearchNode`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
basis | A references to `Tag` with `key` "same" | URI | REQUIRED. MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/Tag`](#tag) that has `type` `http://gedcomx.polygenea.org/v1/Tag/same`.



<a name="derivation"/>

## 3.30 The "Derivation" Data Type

The `Derivation` data type stores free-text descriptions of reasoning processes.

### identifier

The identifier for the `Derivation` data type is:

`http://gedcomx.polygenea.org/v1/Derivation`

### extension

This data type extends the following data type:

`http://gedcomx.polygenea.org/v1/ResearchNode`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
support | A set of references to the antecedents of this derivation | List of URI. Order is not preserved. | REQUIRED. Each URI MUST resolve to an instance of either [`http://gedcomx.polygenea.org/v1/ResearchNode`](#research-node) or `http://gedcomx.org/v1/SourceDescription`.
reason | The user's explanation of why this derivation works | `http://gedcomx.org/TextValue` | OPTIONAL.  If not provided, the reason is assumed to be {"lang":"en", "value":"(undocumented)"}.


<a name="inference"/>

## 3.31 The "Inference" Data Type

The `Inference` data type stores structured, machine-understandable descriptions of single steps in the reasoning processes.

### identifier

The identifier for the `Inference` data type is:

`http://gedcomx.polygenea.org/v1/Inference`

### extension

This data type extends the following data type:

`http://gedcomx.polygenea.org/v1/ResearchNode`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
support | A list of references to the antecedents of this derivation | List of URI. Order is preserved. | REQUIRED. Each URI MUST resolve to an instance of either [`http://gedcomx.polygenea.org/v1/ResearchNode`](#research-node) or `http://gedcomx.org/v1/SourceDescription`.
reason | A reference to the `Expectation` that led to this inference | URI | REQUIRED. MUST resolve to an instance of `http://gedcomx.polygenea.org/Expectation`.  `support` SHOULD [match](#inference-matching-algorithm) the referenced `Expectation`.


<a name="expectation"/>

## 3.32 The "Expectation" Data Type

The `Expectation` data type stores a structured description of a trends, inference rule, or other guidelines about what a researcher expects to be true.

### identifier

The identifier for the `Expectation` data type is:

`http://gedcomx.polygenea.org/v1/Expectation`

### extension

This data type extends the following data type:

`http://gedcomx.polygenea.org/v1/ResearchNode`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
antecedent | A list of `NodeQuery`s describing what must be true for this `Expectation` to hold | List of `http://gedcomx.polygenea.org/v1/NodeQuery`. Order is preserved. | REQUIRED.
consequent | A list of `NodeTemplate`s describing what may be inferred if the `antecedent` is satisfied | List of `http://gedcomx.polygenea.org/v1/NodeTemplate`. Order is preserved. | REQUIRED.

The __index__ of a `NodeQuery` in its containing `Expectation` is defined to be the 0-based index of the node in the `antecedent` list.

The __index__ of a `NodeTemplate` in its containing `Expectation` is defined to be the 0-based index of the node in the `consequent` list plus the number of elements in the `antecedent` list.


<a name="node-template"/>

## 3.32 The "NodeTemplate" Data Type

The `NodeTemplate` data type describes how to construct a node given some context.

### identifier

The identifier for the `NodeTemplate` data type is:

`http://gedcomx.polygenea.org/v1/NodeTemplate`

### extension

For every concrete node type named _X_ extending `Claim`, there is a concrete node type named _X_`Template` extending `NodeTemplate`.

### properties

None; in particular, `NodeTemplate` (and hence its extension nodes) does not have either an `attribution` nor a `source` property.

Nodes that extend `NodeTemplate` MUST use type int instead of type `URI` anywhere type `URI` is required for the corresponding type that extends `Claim`.  The integer values MUST be strictly smaller than the index of the `NodeTemplate` in its containing `Expectation` and MUST be greater than or equal to 0.

Nodes that extend `NodeTemplate` MAY use a function type as the value of a string or 


<a name="node-template"/>

## 3.32 The "NodeTemplate" Data Type

The `NodeTemplate` data type describes how to construct a node given some context.

### identifier

The identifier for the `NodeTemplate` data type is:

`http://gedcomx.polygenea.org/v1/NodeTemplate`

### extension

For every concrete node type named _X_ extending `Claim`, there is a concrete node type named _X_`Template` extending `NodeTemplate`.

### properties

None; in particular, `NodeTemplate` (and hence its extension nodes) does not have either an `attribution` nor a `source` property.

Nodes that extend `NodeTemplate` MUST use type int instead of type `URI` anywhere type `URI` is required for the corresponding type that extends `Claim`.  The integer values MUST be strictly smaller than the index of the `NodeTemplate` in its containing `Expectation` and must be greater than or equal to −1.

Nodes that extend `NodeTemplate` MAY use a function type 
