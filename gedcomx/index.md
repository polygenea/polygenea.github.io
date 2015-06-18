__To `conceptual-model-specification` make the following changes:__

----

__add to 1.3.3__

> A "set" is defined as an unordered collection of data instances containing no duplicate data instances.
> When a property is defined in terms of a "set", the data type of the data instances in the set is also provided
>
> An "integer" is defined as an integer numerical value; its storage is not specified, but must be able to store at least all integers between −1 and 32766, inclusive.

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
subjects | The subjects of study during research | Set of [`http://gedcomx.polygenea.org/v1/SubjectNode`](#subject-node). | OPTIONAL.
properties | The information about single things encountered during research | Set of [`http://gedcomx.polygenea.org/v1/Property`](#property). | OPTIONAL.
connections | The information about pairs things encountered during research | Set of [`http://gedcomx.polygenea.org/v1/Connection`](#connection). | OPTIONAL.
tags | Metadata asserted about various other claims during research | Set of [`http://gedcomx.polygenea.org/v1/Tag`](#tag). | OPTIONAL.
aggregates | Semantically-merged collections of subjects | Set of [`http://gedcomx.polygenea.org/v1/AggregatedSubject`](#aggregated-subject). | OPTIONAL.
outrefs | Descriptions of external sources in a format suitable for reasoning | Set of [`http://gedcomx.polygenea.org/v1/OutRef`](#outref). | OPTIONAL.
derivations | Free-text descriptions of reasoning processes | Set of [`http://gedcomx.polygenea.org/v1/Derivation`](#derivation). | OPTIONAL.
inferences | Structured descriptions of reasoning processes | Set of [`http://gedcomx.polygenea.org/v1/Inference`](#inference). | OPTIONAL.
expectations | Structured descriptions of trends, inference rules, and other guidelines about what a researcher expects to be true | Set of [`http://gedcomx.polygenea.org/v1/Expectation`](#expectation). | OPTIONAL.
attribution | The attribution of this reasoning. | `http://gedcomx.org/Attribution` | OPTIONAL. If not provided, the attribution of the containing data set (e.g., file) of the `Reasoning` is assumed.

### constraints

A single data set SHOULD NOT contain more than a single `Reasoning` element.
If it contains more than one, the `id` property of the various `ReasoningNode`s inside each must be unique.

----

__Modify section 3.10, subsection "properties", adding the following row to the table:__

name  | description | data type | constraints
------|-------------|-----------|------------
support | The list of references to the reasoning elements that support this conclusion. | Set of [`http://gedcomx.polygenea.org/v1/ClaimReference`](#claim-reference). | OPTIONAL.
contradiction | The list of references to the reasoning elements that contradict this conclusion. | Set of [`http://gedcomx.polygenea.org/v1/ClaimReference`](#claim-reference). | OPTIONAL.


----

__Add to 3. several new subsections, as follows__


<a name="reasoning-node"/>

## 3.22 The "ReasoningNode" Data Type

The `ReasoningNode` data type defines the abstract concept of a single atomic element in the reasoning process.

### identifier

The identifier for the `ReasoningNode` data type is:

`http://gedcomx.polygenea.org/v1/ReasoningNode`


### properties

name  | description | data type | constraints
------|-------------|-----------|------------
id | An internal identifier for the data structure holding the reasoning data. | string | REQUIRED.  The id is to be used as a "fragment identifier" as defined by [RFC 3986, Section 3.5](http://tools.ietf.org/html/rfc3986#section-3.5). As such, the constraints of the id are provided in the definition of the media type (e.g. XML, JSON) of the data structure.
attribution | The attribution of this subject. | `http://gedcomx.org/Attribution` | OPTIONAL. If not provided, the attribution of the containing data set (i.e., `Reasoning`) of the `ReasoningNode` is assumed.

It is RECOMMENDED that the `id` of each `ReasoningNode` be the (lower-case) initial letter of the name of the `ReasoningNode`s element type (which will be one of {"a", "c", "d", "e", "i", "p", "s", "t"}),  followed by the (non-padded) decimal string representation of 0-based index of the `ReasoningNode` within its containing element.
For example, the second `Inference` node inside a `Reasoning` element's `inference` property would have an `id` of "i1".

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
attribution | The attribution of this claim reference. | `http://gedcomx.org/Attribution` | OPTIONAL. If not provided, the attribution of the containing resource of the claim reference is assumed.


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

The `Property` data type defines a string-valued detail about a `ResearchNode`, most commonly a `SubjectNode`.

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



<a name="outref"/>

## 3.33 The "OutRef" Data Type

The `OutRef` data type stores a structured description of a source external to the reasoning itself.
It is an approximate parallel of the `http://gedcomx.org/v1/SourceCitation` element, but organized in a way that supports `Expectation`s and automated reasoning.

### identifier

The identifier for the `OutRef` data type is:

`http://gedcomx.polygenea.org/v1/OutRef`

### extension

This data type extends the following data type:

`http://gedcomx.polygenea.org/v1/ResearchNode`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
parents | A set of `OutRefRef`s describing relationships between this `OutRef` and other `OutRef`s | List of [`http://gedcomx.polygenea.org/v1/OutRefRef`](#outref-ref). Order is not preserved. | OPTIONAL. If not provided, assumed to be an empty set.
details | A set of `KeyValue`s describing individual features of this `OutRef` | List of [`http://gedcomx.polygenea.org/v1/KeyValue`](#key-val). Order is not preserved. | OPTIONAL. If not provided, assumed to be an empty set.


<a name="outref-ref"/>

## 3.34 The "OutRefRef" Data Type

The `OutRefRef` data type stores a named reference to an `OutRef`.
They are used in the `parents` property of [`OutRef`](#outref) nodes.

### identifier

The identifier for the `OutRefRef` data type is:

`http://gedcomx.polygenea.org/v1/OutRefRef`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
key | the relationship between to the listed `OutRef` | Enumerated Value | REQUIRED. MUST identify a source relationship type, and use of a [known source relationship type](#known-source-relationship-types) is RECOMMENDED.
value | an `OutRef`s having this relationship | URI | REQUIRED. MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/OutRef`](#outref).



<a name="key-val"/>


## 3.35 The "KeyVal" Data Type

The `KeyVal` data type stores a generic key:value pair of strings.
They are used in the `details` property of [`OutRef`](#outref) nodes.

### identifier

The identifier for the `KeyVal` data type is:

`http://gedcomx.polygenea.org/v1/KeyVal`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
key | the topic of this attribute | Enumerated Value | REQUIRED. MUST identify a source detail type, and use of a [known source detail type](#known-source-relationship-types) is RECOMMENDED.
value | the value of this attribute | string | REQUIRED.  Constraints on values may be added for particular `key`s.






<a name="key-int"/>

## 3.36 The "KeyInt" Data Type

The `KeyInt` data type stores a key:value pair where the value is an integer.
They are used in place of `OutRefRef` values in `OutRefQuery` elements.

### identifier

The identifier for the `KeyInt` data type is:

`http://gedcomx.polygenea.org/v1/KeyInt`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
key | the relationship between to the listed `OutRef` | Enumerated Value | REQUIRED. MUST identify a source relationship type, and use of a [known source relationship type](#known-source-relationship-types) is RECOMMENDED.
value | an `OutRef`s having this relationship | integer | REQUIRED. MUST either be `-1` or a non-negative integer smaller than the index of the containing `OutRefQuery` within the containing `Expectation`.



<a name="key-predicate"/>


## 3.37 The "KeyPredicate" Data Type

The `KeyPreducate` data type stores a generic key:value pair where the value is a `StringPredicate`.
They are used in place of `KeyVal` values in `OutRefQuery` elements.

### identifier

The identifier for the `KeyPredicate` data type is:

`http://gedcomx.polygenea.org/v1/KeyPredicate`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
key | the topic of this attribute | Enumerated Value | REQUIRED. MUST identify a source detail type, and use of a [known source detail type](#known-source-relationship-types) is RECOMMENDED.
value | the value of this attribute | `StringPredicate` | REQUIRED.





-----









<a name="predicate"/>
## 3.22 The "Predicate" Data Type and Subtypes

The `Predicate` data type defines the abstract concept of a function that accepts some inputs and rejects others.

### identifier

The identifier for the `Predicate` data type is:

`http://gedcomx.polygenea.org/v1/Predicate`

In the table of predicate subtypes, the "name" column defines a suffix to the `Predicate` identifier;
the full identifier for a subtype can be constructed as `http://gedcomx.polygenea.org/v1/Predicate/` concatenated with the value in the "name" column.

### subtypes

There are many subtypes of `Predicate`, collectively listed in the following table.

name | description | properties | constraints
`Top` | always true |  | 
`LitString` | true for a single value only | `x`, a string | 
MapHas | REQUIRED | set of string ↦ datum pairs | _x_, a set of string ↦ (datum predicate) pairs | for each (_a_ ↦ _b_) in _x_, there is a (_c_ ↦ _d_) pair in _v_ such that _a_ equals _c_ and _b_(_d_) is `true`
Same | RECOMMENDED | any | _i_, an integer<br/>_j_, an integer | _v_ equals the _j_th value of the _i_th tuple in _s_
Has | RECOMMENDED | set or list of _X_ | _f_, an _X_ predicate | _f_(_e_, _s_) is `true` for any _e_ in _v_
SetHas | RECOMMENDED | set of _X_ | _x_, a set of _X_ predicates | for each _f_ in _x_ there is a _e_ in _v_ such that _f_(_e_, _s_) is `true`
Cmp | OPTIONAL | either string or datum with a media type that has defined order | _∙_, an operator from the set {`<`, `≤`, `=`, `≠`, `≥`, `>`}<br/>_x_, a value of the same type as the predicate | for a datum: _v_ ∙ _x_ under the media type's defined ordering<br/>for a string: _v_ ∙ _x_ under a lexicographical ordering
ICmp | OPTIONAL | as `Cmp` | _∙_, an operator from the set {`<`, `≤`, `=`, `≠`, `≥`, `>`}<br/>_i_, an integer<br/>_j_, an integer | as `Cmp`, but using the _j_th value of the _i_th tuple in _s_ instead of _v_
Regex | OPTIONAL | string | _r_, a regex | _r_ matches _v_
Len | OPTIONAL | set or list | _∙_, an operator from the set {`<`, `≤`, `=`, `≠`, `≥`, `>`}<br/>_x_, an integer | ((the number of elements in _v_) ∙ _x_) is `true`
And | OPTIONAL | any (call it _X_) | _x_, a set of _X_ predicates | _f_(_v_, _s_) is `true` for all _f_ in _x_
Or | OPTIONAL | any (call it _X_) | _x_, a set of _X_ predicates | _f_(_v_, _s_) is `true` for at least one _f_ in _x_
Not | OPTIONAL | any (call it _X_) | _f_, an _X_ predicate | _f_(_v_, _s_) is `false`
Script | OPTIONAL | any | _x_, a datum defining a single function in some programming language | evaluating the function in _x_ with arguments _v_ and _s_ returns `true`


<a name="producer"/>
## 3.23 The "Producer" Data Type and Subtypes

The `Predicate` data type defines the abstract concept of a function that produces a value of a particular type.

### identifier

The identifier for the `Producers` data type is:

`http://gedcomx.polygenea.org/v1/Producer`

In the table of producer subtypes, the "name" column defines a suffix to the `Producer` identifier;
the full identifier for a subtype can be constructed as `http://gedcomx.polygenea.org/v1/Producer/` concatenated with the value in the "name" column.

### subtypes

There are many subtypes of `Producer`, collectively listed in the following table.

name | description | properties | constraints
Lit | REQUIRED | any | _x_, a value of the same type as the producer | _x_
Lookup | RECOMMENDED | any | _i_, an integer<br/>_j_, an integer | the _j_th value of the _i_th tuple in _s_ 
Match | OPTIONAL | string | _f_, a string producer<br/>_r_, a regex<br/>_i_, an integer | the contents of the _i_th matching group after matching _f_(_s_) with _r_, or the empty string if it does not match or the match has no such group
Slice | OPTIONAL | string or list | _f_, a string or list producer<br/>_i_, an integer<br/>_j_, an integer | the zero-indexed subsequence of _f_(_s_) from _i_ (inclusive) to _j_ (exclusive).<br/>Negative indices have the length of the sequence added to them before dereferencing;<br/>out-of-bounds indices are clamped to bounds; and<br/>negative-width subsequences return the empty sequence
Cat | OPTIONAL | string or list | _x_, a list of string or list producers | the sequence produced by concatenating the sequenced returned by each elements of _x_ in order
Union | OPTIONAL | set | _x_, a set of set producers | a set containing every value contained in any of the sets produced by each of the elements of _x_
Intersect | OPTIONAL | set | _x_, a set of set producers | a set containing those values that are in every set produced by each element of _x_
Script | OPTIONAL | any | _x_, a datum defining a single function in some programming language | the value returned when evaluating the function in _x_ with argument _s_






<a name="node-template"/>

## 3.33 The "NodeTemplate" Data Type

The `NodeTemplate` data type describes how to construct a node given some context.

### identifier

The identifier for the `NodeTemplate` data type is:

`http://gedcomx.polygenea.org/v1/NodeTemplate`

### extension

For every concrete node type named _X_ extending `Claim`, there is a concrete node type named _X_`Template` extending `NodeTemplate`.

### properties

None; in particular, `NodeTemplate` (and hence its extension nodes) does not have either an `attribution` nor a `source` property.

Nodes that extend `NodeTemplate` MUST use type `int` instead of type `URI` anywhere type `URI` is required for the corresponding type that extends `Claim`.  The integer values MUST be strictly smaller than the index of the `NodeTemplate` in its containing `Expectation` and MUST be greater than or equal to 0.

Nodes that extend `NodeTemplate` MAY use a function type as the value of a string or 
__FIXME__: _as will be described later_


| `NodeTemplate` subtype | Properties<br/>index. name : type | Constraints |
|---------------------|-----------------------------------|-------------|
| SubjectTemplate | 0. `slug` : string producer |
| PropertyQuery | 0. `key` : string producer <br/>1. `of` : int<br/>2. `value` : string producer |
| ConnectionQuery | 0. `key` : string producer <br/>1. `of` : int<br/>2. `value` : int |
| TagQuery | 0. `key` : string producers <br/>1. `of` : int |



<a name="node-query"/>

## 3.34 The "NodeQuery" Data Type

The `NodeQuery` data type describes an acceptable set of nodes.

### identifier

The identifier for the `NodeQuery` data type is:

`http://gedcomx.polygenea.org/v1/NodeQuery`

### extension

For every node type named _X_ extending `ResearchNode`, there is a concrete node type named _X_`Template` extending `NodeQuery`.

### properties

As per `ResearchNode` and its subtypes, except as noted below.

Nodes that extend `NodeQuery` MUST use type `int` instead of type `URI` anywhere type `URI` is required for the corresponding type that extends `Claim`.  The integer values MUST be strictly smaller than the index of the `NodeQuery` in its containing `Expectation` and must be greater than or equal to −1.

Nodes that extend `NodeQuery` MAY use a predicate type
__FIXME__: _as will be described later_


| `NodeQuery` subtype | Properties<br/>index. name : type | Constraints |
|---------------------|-----------------------------------|-------------|
| SubjectQuery | 0. `slug` : string predicate <br/>1. `source` : int | `slug`'s value MUST be _Top_ |
| PropertyQuery | 0. `key` : string predicate <br/>1. `of` : int<br/>2. `value` : string predicate<br/>3. `source` : int | |
| ConnectionQuery | 0. `key` : string predicate <br/>1. `of` : int<br/>2. `value` : int<br/>3. `source` : int | |
| TagQuery | 0. `key` : string predicate <br/>1. `of` : int<br/>2. `source` : int | |
| OutRefQuery | 0. `parents` : set of (string, int) pairs<br/>1. `details` : set of (string, string predicate) pairs | |
| DerivationQuery | 0. `support` : set of ints<br/>1. `reason` : string predicate | |
| ExpectationQuery | _exactly the same a Expectation_ | |
| InferenceQuery | 0. `support` : set of ints<br/>1. `reason` : int | |

