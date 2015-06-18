<style type="text/css">
body{font-family:sans-serif; counter-reset:h1;}
code{font-family:monospace;font-size:100%;background-color:#eee;border:1px solid #aaa; padding:1px; border-radius:4px;}
table{border-collapse:collapse;}
tr:nth-child(2n){background-color:#ddd;}
td,th{padding:0.25ex 1ex;}
th{border-bottom:1px solid black;}
</style>


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
source | The attribution of this subject. | URI | REQUIRED. MUST resolve to an instance of one of the following types: [`http://gedcomx.polygenea.org/v1/OutRef`](#outref), [`http://gedcomx.polygenea.org/v1/Inference`](#inference), or [`http://gedcomx.polygenea.org/v1/Derivation`](#derivation).


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
key | Enumerated value identifying the type of property | Enumerated Value | REQUIRED. If an Enumerated Value, use of a [known property type](#known-property-types) is RECOMMENDED.
of | A reference to the `ReasoningNode` to which this property applies | URI | REQUIRED. MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/ReasoningNode`](#reasoning-node). Some `key`s may impose additional constraints.
value | The value of the property. | string | REQUIRED. Some `key`s may impose additional constraints.


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
key | Enumerated value identifying the type of connection | string | REQUIRED. If an Enumerated Value, use of a [known connection type](#known-connection-types) is RECOMMENDED.
of | A reference to the `ReasoningNode` to which this connection applies | URI | REQUIRED. MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/ReasoningNode`](#reasoning-node). Some `key`s may impose additional constraints.
value | The value of the the connection | URI | REQUIRED. MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/ReasoningNode`](#reasoning-node). Some `key`s may impose additional constraints.


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
key | Enumerated value identifying the type of connection | Enumerated Value | REQUIRED. MUST identify a tag type, and use of a [known tag type](#known-tag-types) is RECOMMENDED.
of | A list of references to jointly-tagged `ReasoningNode`s | List of URI.  Order is not preserved | REQUIRED. Each URI MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/ReasoningNode`](#reasoning-node).  Some `key`s may impose additional constraints.


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
basis | A references to `Tag` with `key` "same" | URI | REQUIRED. MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/Tag`](#tag) that has `key` `http://gedcomx.polygenea.org/v1/Tag/same`.



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
support | A set of references to the antecedents of this derivation | List of URI. Order is not preserved. | REQUIRED. Each URI MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/ResearchNode`](#research-node).
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
support | A list of references to the antecedents of this derivation | List of URI. Order is preserved. | REQUIRED. Each URI MUST resolve to an instance of [`http://gedcomx.polygenea.org/v1/ResearchNode`](#research-node).
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
key | the topic of this attribute | Enumerated Value | REQUIRED. MUST identify a source detail type, and use of a [known source detail type](#known-source-detail-types) is RECOMMENDED.
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

The `KeyPreducate` data type stores a generic key:value pair where the value is a `Predicate`.
They are used in place of `KeyVal` values in `OutRefQuery` elements.

### identifier

The identifier for the `KeyPredicate` data type is:

`http://gedcomx.polygenea.org/v1/KeyPredicate`

### properties

name  | description | data type | constraints
------|-------------|-----------|------------
key | the topic of this attribute | Enumerated Value | REQUIRED. MUST identify a source detail type, and use of a [known source detail type](#known-source-relationship-types) is RECOMMENDED.
value | the value of this attribute | `Predicate` | REQUIRED.



<a name="predicate"/>
## 3.38 The "Predicate" Data Type and Subtypes

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
`Lit` | true for a single value only | `x`, a string | 
`MapHas` | peeks inside an `OutRef`'s `details` | `x`, a set of `KeyPredicate`s | 
`Same` | two nodes have the same contents | `i`, an integer<br/>`j`, a string | `i` is ≥ 0 and less than the index of the containing `NodeQuery`<br/>`j` is the name of a property of the `NodeQuery` at index `i`<br/>the type of the `j` property is either string or Enumerated Value
`Cmp` | compares a value with a constant | `op`, which is one of "<", "<=", "==", "!=", ">=", ">"<br/>`x`, a string | 
`ICmp` | compares two values | | `op`, which is one of "<", "<=", "==", "!=", ">=", ">"<br/>`i`, an integer<br/>`j`, an integer | `i` and `j` have the same constraints as for `Same` in this table
`Regex` | regular expression matching | `r`, a string | the contents of `r` must be either a _RegularExpressionLiteral_ or a _RegularExpressionBody_ as defined in [ECMA 262 section 7.8.5](http://www.ecma-international.org/ecma-262/5.1/#sec-7.8.5).
`And` | combines two or more predicates | `parts`, a set of `Predicate`s | The set of predicates is not empty
`Or` | combines two or more predicates | `parts`, a set of `Predicate`s | The set of predicates is not empty
`Not` | inverts a predicate | `x`, a Predicate |
`Script` | arbitrary code | `src`, a string<br/>`type`, a string | `lang` identifies a scripting language<br/>`src` is a program in that language<br/>`src` contains a single function accepting two parameters: a string and a list of `ResearchNode`s; and returning a Boolean value


<a name="string-producer"/>
## 3.39 The "StringProducer" Data Type and Subtypes

The `StringProducer` data type defines the abstract concept of a function that produces a string value.

### identifier

The identifier for the `StringProducer` data type is:

`http://gedcomx.polygenea.org/v1/StringProducer`

In the table of producer subtypes, the "name" column defines a suffix to the `StringProducer` identifier;
the full identifier for a subtype can be constructed as `http://gedcomx.polygenea.org/v1/StringProducer/` concatenated with the value in the "name" column.

### subtypes

There are many subtypes of `StringProducer`, collectively listed in the following table.

name | description | properties | constraints
`Lit` | a constant value | `x`, a string | 
`Lookup` | a value copied from another node | `i`, an integer<br/>`j`, a string | `i` is ≥ 0 and less than the index of the containing `NodeTemplate`<br/>`j` is the name of a property of the `NodeQuery` or `NodeTemplate` at index `i`<br/>the type of the `j` property is either a string, an Enumerated Value, or a `StringProducer`
`Match` | a regular expression match | `f`, a `StringProducer`<br/>`r`, a string<br/>`i`, an integer | the contents of `r` must be either a _RegularExpressionLiteral_ or a _RegularExpressionBody_ as defined in [ECMA 262 section 7.8.5](http://www.ecma-international.org/ecma-262/5.1/#sec-7.8.5)
`Slice` | a substring | `f`, a `StringProducer`<br/>`i`, an integer<br/>`j`, an integer | 
`Cat` | string concatentation | `parts`, a list of `StringProducer`s | 
`Script` | arbitrary code | `src`, a string<br/>`type`, a string | `lang` identifies a scripting language<br/>`src` is a program in that language<br/>`src` contains a single function accepting one parameter: a list of `ResearchNode`s; and returning a string value



<a name="set-producer"/>
## 3.40 The "SetProducer" Data Type and Subtypes

The `SetProducer` data type defines the abstract concept of a function that produces a string value.

### identifier

The identifier for the `SetProducer` data type is:

`http://gedcomx.polygenea.org/v1/SetProducer`

In the table of producer subtypes, the "name" column defines a suffix to the `SetProducer` identifier;
the full identifier for a subtype can be constructed as `http://gedcomx.polygenea.org/v1/SetProducer/` concatenated with the value in the "name" column.

### subtypes

There are many subtypes of `SetProducer`, collectively listed in the following table.

name | description | properties | constraints
`Lit` | a constant value | `x`, a set of integer values | every element of `x` must be ≥ 0<br/>every element of `x` must be < the index of the containing `NodeTemplate`
`Lookup` | a value copied from another node | `i`, an integer<br/>`j`, a string | `i` is ≥ 0 and less than the index of the containing `NodeTemplate`<br/>`j` is the name of a property of the `NodeQuery` or `NodeTemplate` at index `i`<br/>the type of the `j` property is either a set of integers or a `SetProducer`
`Union` | the union of other sets | `parts`, a set of `SetProducer`s |
`Intersect` | the common elements of other sets | `parts`, a set of `SetProducer`s |


<a name="node-template"/>

## 3.41 The "NodeTemplate" Data Type and Subtypes

The `NodeTemplate` data type describes how to construct a node given some context.
It has a subtype corresponding to each subtype of `Claim`

### identifier

The identifier for the `NodeTemplate` data type is:

`http://gedcomx.polygenea.org/v1/NodeTemplate`

### extension

For every concrete node type named _X_ extending `Claim`, there is a concrete node type named _X_`Template` extending `NodeTemplate`.

| `NodeTemplate` subtype | Properties | Constraints |
|------------------------|------------|-------------|
| SubjectTemplate | `slug`, a `StringProducer` |
| PropertyTemplate | `key`, a `StringProducer`<br/>`of`, an integer<br/>`value`, a `StringProducer` | `of` must be ≥ `-1` and < the index of this `PropertyTemplate`
| ConnectionTemplate |`key`, a `StringProducer`<br/>`of`, an integer<br/>`value`, an integer | both `of` and `value` must be ≥ `-1` and < the index of this `ConnectionTemplate`
| TagTemplate | `key`, a `StringProducer` <br/>`of`, a `SetProducer` |



<a name="node-query"/>

## 3.34 The "NodeQuery" Data Type

The `NodeQuery` data type describes an acceptable set of nodes.
It has a subtype corresponding to each subtype of `ResearchNode`

### identifier

The identifier for the `NodeQuery` data type is:

`http://gedcomx.polygenea.org/v1/NodeQuery`

### extension

For every node type named _X_ extending `ResearchNode`, there is a concrete node type named _X_`Query` extending `NodeQuery`.

| `NodeQuery` subtype | Properties | Constraints |
|---------------------|-----------------------------------|-------------|
| SubjectQuery | `from`, an integer | all integer values must be ≥ `-1` and < the index of this `NodeQuery` |
| PropertyQuery | `key`, a `StringPredicate` <br/>`of`, an integer<br/>`value`, a `StringPredicate`<br/>`source`, an integer | all integer values must be ≥ `-1` and < the index of this `NodeQuery` |
| ConnectionQuery | `key`, a `StringPredicate` <br/>`of`, an integer<br/>`value`, an integer<br/>`source`, an integer | all integer values must be ≥ `-1` and < the index of this `NodeQuery` |
| TagQuery | `key`, a `StringPredicate` <br/>`of`, a set of integers<br/>`source`, an integer | all integer values must be ≥ `-1` and < the index of this `NodeQuery` |
| OutRefQuery | `parents`, a set of `KeyInt`s<br/>`details`, a set of `KeyPredicate`s | |
| DerivationQuery | `support`, a set of integers<br/>`reason`, a `StringPredicate` | all integer values must be ≥ `-1` and < the index of this `NodeQuery` |
| ExpectationQuery | _exactly the same a Expectation_ | |
| InferenceQuery | `support`, a list of integers<br/>`reason`, an integer | all integer values must be ≥ `-1` and < the index of this `NodeQuery` |


-----

__add the following sections__

# 6. Know Enumerated Types for Reasoning

_this section and its subsections are far from complete_

<a name="known-connection-types"/>

## 6.1 Known Connection Types

| URI | constraints | description |
|-----|-------------|-------------|
| `http://gedcomx.polygenea.org/v1/Connection/update` | `of` and `value` have same node type | an error or inaccuracy in `of` has been corrected in `value` |

Any [Known Role Types](https://github.com/FamilySearch/gedcomx/blob/master/specifications/conceptual-model-specification.md#known-roles)
is also considered to be a Known Connection Type.

<a name="known-property-types"/>

## 6.2 Known Property Types

| URI | constraints | description |
|-----|-------------|-------------|
| `http://gedcomx.polygenea.org/v1/Property/type` | `value` is an Enumerated Value; use of a [known subject type](#known-subject-types) is RECOMMENDED<br/>`of` is a Subject or Aggregated Subject | what type of entity this subject refers to |
| `http://gedcomx.polygenea.org/v1/Property/date` | | the date associated with the described node |
| `http://gedcomx.polygenea.org/v1/Property/date#formal` | `value` is a [`http://gedcomx.org/date/v1`](https://github.com/FamilySearch/gedcomx/blob/master/specifications/date-format-specification.md) | the date associated with the described node |
| `http://gedcomx.polygenea.org/v1/Property/date#original` | | the date associated with the described node |
| `http://gedcomx.polygenea.org/v1/Property/name` | | a name
| `http://gedcomx.polygenea.org/v1/Property/gender` | using a [Known Gender Type](https://github.com/FamilySearch/gedcomx/blob/master/specifications/conceptual-model-specification.md#known-gender-types) for `value` is RECOMMENDED | the biological or social gender identity

Any [Known Name Types](https://github.com/FamilySearch/gedcomx/blob/master/specifications/conceptual-model-specification.md#known-name-types)
or [Known Fact Qualifiers](https://github.com/FamilySearch/gedcomx/blob/master/specifications/conceptual-model-specification.md#known-fact-qualifier)
is also considered to be a Known Property Type.


<a name="known-subject-types"/>

## 6.2.1 Known Subject Types

| URI | subtype of | description |
|-----|------------|-------------|
| `http://gedcomx.polygenea.org/v1/Subject/person` | a human or putative human |
| `http://gedcomx.polygenea.org/v1/Subject/event` | a distinct and recognisable (time period, location) pair or set thereof effecting or attempting to effect a common end |
| `http://gedcomx.polygenea.org/v1/Subject/place` | a location or region defined politically, geographically, socially, etc. |

Any [Known Event Type](https://github.com/FamilySearch/gedcomx/blob/master/specifications/conceptual-model-specification.md#known-event-types)
or [additional Known Event Type](https://github.com/FamilySearch/gedcomx/blob/master/specifications/event-types-specification.md)
or [Known Fact Type](https://github.com/FamilySearch/gedcomx/blob/master/specifications/conceptual-model-specification.md#known-fact-types)
is also considered to be a Known Subject Type.
Many [additional Known Fact Type](https://github.com/FamilySearch/gedcomx/blob/master/specifications/fact-types-specification.md) values also describe event types.


<a name="known-source-detail-types"/>

## 6.3 Known Source Detail Types

| URI | constraints | description |
|-----|-------------|-------------|
| `http://gedcomx.polygenea.org/v1/OutRef/accessed` | a [`http://gedcomx.org/date/v1`](https://github.com/FamilySearch/gedcomx/blob/master/specifications/date-format-specification.md), optionally followed by a SEMICOLON `;` and an identifier of who performed the access | when the external resource described by this `OutRef` was accessed by the identified access, or by the creator of this `OutRef` if no identifier is included | 


<a name="known-source-relationship-types"/>

## 6.4 Known Source Relationship Types

| URI | constraints | description |
|-----|-------------|-------------|
| `http://gedcomx.polygenea.org/v1/OutRef/extends` | | this `OutRef` is a subpart of the indicated `OutRef` | 

<a name="known-tag-types"/>

## 6.5 Known Tag Types

| URI | constraints | description |
|-----|-------------|-------------|
| `http://gedcomx.polygenea.org/v1/Tag/distinct` | ≥ 2 elements in `of`<br/>each elements referenced in `of` either a Subject or and Aggregated Subject | no two elements of `of` refers to the same real-world subject |
| `http://gedcomx.polygenea.org/v1/Tag/equivalent` | ≥ 2 elements in `of` | the elements in `of` contain equivalent information (a hint that only one element needs to be displayed) |
| `http://gedcomx.polygenea.org/v1/Tag/same` | ≥ 2 elements in `of`<br/>each elements referenced in `of` either a Subject or and Aggregated Subject | all of the elements of `of` refer to the same real-world subject |
| `http://gedcomx.polygenea.org/v1/Tag/unsupported` | `of` contains a single reference to a Claim, Inference, or Derivation | the `source` does not make this Claim or the `support` does not justify this Inference or Derivation  | 
| `http://gedcomx.polygenea.org/v1/Tag/wrong` | `of` cannot include both a "wrong" Tag and any element of that Tag's `of`<br/>`of` cannot include a Subject or Aggregated Subject | the indicated node asserts a falsehood | 



