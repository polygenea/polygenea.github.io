<style type="text/css">
body{font-family:sans-serif; counter-reset:h1;}
code{font-family:monospace;font-size:100%;background-color:#eee;border:1px solid #aaa; padding:1px; border-radius:4px;}
table{border-collapse:collapse;}
tr:nth-child(2n){background-color:#ddd;}
td,th{padding:0.25ex 1ex;}
th{border-bottom:1px solid black;}
h1:before { content: counter(h1) ". "; counter-increment: h1; }
h1 {counter-reset:h2;}
h2:before { content: counter(h1) "." counter(h2) ". "; counter-increment: h2; }
h2 {counter-reset: h3;}
h3:before { content: counter(h1) "." counter(h2) "." counter(h3) ". "; counter-increment: h3; }
h3 {counter-reset: h4;}
h4:before { content: counter(h1) "." counter(h2) "." counter(h3) "." counter(h4) ". "; counter-increment: h4; }
h4 {counter-reset: h5;}
</style>


This is an alpha version of a specification of a model for representing family history and genealogical research.

The central goal of this model is the representation of reasoning processes, as realised by the Inference, Derivation, and Expectation node types.
A secondary goal is to ensure, at a data model level, that it is impossible to have two communicating parties think they understand one another when in fact they have different versions of some element of their shared universe of discourse.
To reach these goals, the model uses a representation of beliefs about the past that somewhat resembles [RDF](http://www.w3.org/TR/rdf11-concepts/), in that information about a given entity is stored outside of the entity.
Unlike RDF, each of these Claim elements has a source, representing where it came from; the structure and presence of these sources is key to achieving the central goal of this model.
Additionally, Claims and other elements are defined not to have identity (a.k.a, they are compare-by-value, bit-copiable, identity=value, immutable, etc.); by definition, changing the contents of any element actually creates a new element, leaving the old one present and unchanged.
The notion that a different element is intended as an update of another element is itself a claim made during reasoning and represented as such in this model.

This document is an alpha release intended to present a self-consistent and fairly complete specification
that can be used to understand how the various conceptual pieces of this approach could fit together
and to form a starting point for discussions of this approach.
Many times while writing this document the optimal choice was not evident;
rather than list each of these decision points, this document simply makes decisions
so that the presentation can hold together as a whole, optimal or not.


Representation of Reasoning
===========================

Basic Datatypes
---------------

__character__
:   an atomic unit of text as specified by `ISO/IEC 10646`.

__string__
:   a finite-length sequence of characters.

__octet__
:   a sequence of eight bits, each either 0 or 1.

__blob__
:   a finite-length sequence of octets.

__datum__
:   a paired media type and payload.
    
    Media types may be specified by [RFC 4288](http://tools.ietf.org/html/rfc4288) (e.g., `text/javascript`) or represented by an IRI (e.g., `http://gedcomx.org/date/v1`).
    Because absolute IRIs always contain a COLON `:` and media types never contain a COLON, it is unambiguous which form of media type is being specified.

    Payload is typically a blob, but may be a string for some media types.
    
__reference__
:   indicating exactly one node within the same dataset.
    Because they do not span datasets,
    references may have an implementation-defined format.
    
    The phrase "reference to a _X_", where _X_ is Node or one of Node's subtypes,
    means the reference indicates an element of type _X_ or one of _X_'s subtypes.

__integer__
:   a signed integer.
    Every implementations must be able to represent (at least)
    all integers in the interval between −1 and 32766, inclusive.
    During serialisation or deserialisation the maximum number of nodes 
    an implementation can handle may be constrained by the largest magnitude integer
    that the implementation can represent.


Composite Datatypes
-------------------

__set__ of _X_
:	(where _X_ is a datatype):
	an unordered collection of zero or more elements of type _X_, containing no duplicates.

__list__ of _X_
:	(where _X_ is a datatype):
	an ordered sequence of zero or more elements of type _X_, possibly containing duplicates.
	Given a list of _n_ elements, the first element is identified by the index 0,
	and each subsequent element by the index (1 + index-of-previous-element).

__tuple__
:	A fixed, ordered group of values, each with a given type.
	Tuple values may be referenced by index (starting with 0, like a _list_)
	or, if the individual elements are given names in the specification, by name.
    Thus, e.g., the first elements of the tuple of a Property node 
    may be referred to as either the `key` element or element `0`.

**_X_ ↦ _Y_ pair**
:	A tuple of two elements.
	If _X_ and _Y_ are types, they specify the types of the elements.
	Otherwise, _X_ and _Y_ are the names of the elements
	and types are given elsewhere in the specification text.


The Eight Node Types
--------------------

This specification defines several concrete types.
These form a type hierarchy:

-   Node
    -   Claim
        -   Subject: identifying a single (subject of discussion, source discussing it) pair
        -   Property: a single piece of information about a node
        -   Connection: a directed connection between two nodes
    -   Source
        -   OutRef: identifying some information source external to this specification
        -   Derivation: a free-text description of a step in the reasoning processes 
        -   Inference: a structured description of a step in the reasoning processes 
    -   Aggregated Subject: a single subject inferred to be discussed by many source
    -   Expectation: structured descriptions of trends, inference rules, and external knowledge

For each Node type there is a Node Query type, with the exception that Subject and Aggregated Subject nodes share a single Node Query type.
Node Queries are used to structure the preconditions or antecedents of an Expectation.

For each Claim type there is a Node Template type.
Node Templates are used to structure the postconditions or consequent of an Expectation.

Each type is described in terms of its constituent fields;
each field is given both an index and a name.
Indices are used to create tuples from nodes and streamline the presentation of aspects of the specification that are not dependent on the particular node types;
names are when discussing particular nodes because they are more suggestive of the purpose of the various fields.

By design, each field name corresponds to the same index everywhere it appears.
To achieve that end, each Node Template has a dummy `source` field (index 0, always value `-1`).


### Nodes

| Node subtype | is a | Properties (index. name : type) | Constraints |
|--------------|------|-----------------------------------|-------------|
| Aggregated Subject | | 0. `parts` : reference to a Tag | `parts`'s `key` MUST be "same" |
| Connection | Claim | 0. `source` : reference to a Source<br/>1. `key` : string <br/>2. `of` : reference to a Node<br/>3. `value` : int | some `key`s make constraints on `of` and/or `value` |
| Derivation | Source | 0. `support` : set of references to Nodes<br/>1. `reason` : string | |
| Expectation | | 0. `antecedent` : list of Node Queries<br/>1. `consequent` : list of Node Templates | _See below_ |
| Inference | Source | 0. `support` : list of references to Nodes<br/>1. `reason` : reference to an Expectation | `reason`'s `antecedent` SHOULD [match](#matching) `support`<br/>_See below_ |
| OutRef | Source | 0. `parents` : set of (string ↦ reference to an OutRef) pairs<br/>1. `details` : set of (string ↦ datum) pairs | some string values make constraints on their corresponding reference or datum values |
| Property | Claim | 0. `source` : reference to a Source<br/>1. `key` : string <br/>2. `of` : reference to a Node<br/>3. `value` : datum | some `key`s make constraints on `of` and/or `value` |
| Subject | Claim | 0. `source` : reference to a Source<br/>1. `slug` : string | `slug` SHOULD identify a single subject within the source |
| Tag | Claim | 0. `source` : reference to a Source<br/>1. `key` : string <br/>2. `of` : set of references to Nodes | some `key`s make constraints on `of` |

All references MUST be acyclic and MUST refer to another node in the same dataset.
If some user insists some Claim has no source, it is RECOMMENDED that the `source` reference an OutRef
with `details` including ("type" ↦ "user assertion") and ("user" ↦ however the user is identified).

A Claim SHOULD NOT reference an Inference in its `source` field 
UNLESS it is an element of the [instantiated node list](#instantiating-nodes-from-inferences) of that Inference.

Some constraints on Node Queries and Node Templates involve their containing Expectation,
and can be seen as constraints on Expectations.

All nodes that could be [instantiated from an Inference](#instantiating-nodes-from-inferences)
MUST satisfy any constraints that apply to other nodes of that type.
Implementations SHOULD enforce these constraints during Expectation and Node Template construction,
but MAY delay enforcement until Inference construction instead.

### Node Queries

For each node type *X* (excepting Aggregated Subject),
there is a corresponding node query type *X* Query, which differs from the *X* node type as follows:

-   where *X* has a "reference to a *Y*", *X* Query has an integer
-   where *X* has a "string", *X* Query has a string predicate, which is one of
    -   __Top__, a special value meaning "matches anything"
    -   a string, matching only what it equals
    -   a datum with media type `application/javascript`; see [Matching](#matching) below for details
-   where *X* has a "datum", *X* Query has a datum predicate, which is one of
    -   __Top__, a special value meaning "matches anything"
    -   a datum, matching only what it equals
    -   a datum with media type `application/javascript`; see [Matching](#matching) below for details

### Node Templates

For each claim node type *X*
there is a corresponding node template type *X* Template, which differs from the *X* node type as follows:

-   field 0. `source` always has the value −1
-   where *X* has a "reference to a *Y*" (other than `source`), *X* Template has an integer
-   where *X* has a "string", *X* Template has a string producer, which is one of
    -   a string, producing only itself
    -   a datum with media type `application/javascript`; see [Instantiating Nodes from Inferences](#instantiating-nodes-from-inferences) below for details
-   where *X* has a "datum", *X* Template has a datum producer, which is one of
    -   a datum, producing only itself
    -   a datum with media type `application/javascript`; see [Instantiating Nodes from Inferences](#instantiating-nodes-from-inferences) below for details


Definitions and Constraints
---------------------------

A set of nodes is called a __dataset__.
This specification only discusses complete datasets, where all node references are to other nodes within the dataset.
Communication protocols that communicate via partial datasets
are neither forbidden by this specification
nor discussed by it.


### Dependencies and Cycles

The __dependencies__ of a node is defined to be the set of all nodes that the 
node references unioned with all of those nodes' dependencies, recursively.

It is REQUIRED that all references be acyclic; in other words, no node may be 
included in its own dependencies.

### Equality

Node and value __equality__ is defined as follows:

-   Two nodes are equal if and only if
    both (1) they are the same type of node (or one is an Expectation Query and the other an Expectation),
    and (2) each field of one node is equal to the corresponding field of the other node.

-   Two references are equal if and only if they refer to nodes that are equal.

-   Two sets are equal if and only if
    each element in each set has an equal element in the other set.

-   Two lists are equal if and only if
    both (1) they have the same length 
    and (2) for each valid index for the lists, the elements in both lists at that index are equal.

-   Two strings are equal if they contain the same characters in the same order.
    Looser versions of equality using unicode normalized forms are also permitted, though not required by this specification.

-   Two data are equal if the media types are equal and the payloads are equal.
    Looser versions of media-type-specific equality are also permitted, though not required by this specification.

### Constituents

The __constituents__ of any node that is not an Aggregated Subject node
is defined to be the singleton set containing just that Node itself.

The __constituents__ of an Aggregated Subject node 
is defined to be the set containing the Aggregated Subject node itself 
and the elements of the _constituents_ of each of the nodes referenced in the `of` field
of the node referenced by the Aggregated Subject's `parts` field.

### Matching

Node and value __matching__ is defined as follows:

-   The __Top__ predicate matches any value

-   A literal predicate and a value match if they are equal

-   A `application/javascript` predicate and a value match if the following process yields the value `true`:

    1.  create a javascript evaluation environment with two global variables
        -   `x` is the value to be matched against
        -   `s` is an array of objects
            -   if the matching is occurring within the context of matching a list of Node Queries to a list of node references, the *i*th object in the array is a javascript object representation of a dereference of the *i*th node reference
            -   otherwise, `s` is empty
    
    2.  execute the javascript code
    3.  yield the value produced by the last statement in the javascript code

-   The integer value `-1` matches every node reference.

-   A set of integers matches a set of node references if
    each integer in the set of integers
    matches some node reference in the set of node references.
    Note that it is *not* necessary for every node reference to match some integer.

-   A set of  string ↦ integer pairs _q_ matches a set of string ↦ node reference pairs _s_
    if for each (_k1_ ↦ _i_) in _q_, there is a (_k2_ ↦ _r_) pair in _s_ such that _k1_ equals _k2_ and _i_ matches _r_.

-   A non-negative integer _i_ matches a node reference _r_ if and only if all of the following are true:
    
    1.  there is a list of node references that was used to initiate the matching process 
    2.  there is an element of that list at index _i_
    3.  the node referenced by the element at the index _i_ is a constituent of _r_.

-   A list of node references matches a list of Node Queries if and only if 
    both (1) they have the same length
    and (2) for each valid list index for the lists, 
    the elements in the list of node reference at that index
    matches the element in the list of Node Queries at that index.

-   A Subject Query matches a reference to a Subject
    if the Subject Query's `from` matches the Subject's `source`.

-   A Subject Query matches a reference to an Aggregated Subject
    if the Subject Query's `from` matches the `parts` or `source` of any element of the Aggregated Subject's constituents.

-   An Expectation Query matches a reference to an Expectation
    if the two are equal.
    
    No predicates or additional flexibility with regard to Expectation Queries are included in this specification
    in part to keep the type hierarchy of bounded depth (i.e., to avoid _X_ predicate predicate … predicate types).
    
    It is possible to define semantics-preserving reordering and re-indexing operations on Expectation's fields.
    This specification does not currently consider same-semantics different-order Expectations and Expecation Queries to match.

-   A Node Query other than a Subject Query or Expectation Query matches a node reference if and only if
	both (1) the referenced node and the Node Query are of the same node type
	and (2) each field in the Node Query matches the corresponding field in the referenced node.


### Instantiating Nodes from Inferences

The __instantiated node list__ of an Inference is defined to be 
a list of Nodes
having the same length as the `consequent` of the `reason` of the Inference
such that the Node at index _i_ of the instantiated node list 
is created from the Node Template at index _i_ of the `consequent`
by doing all of the following (in any order)

-   replacing any `application/javascript` producer values with the value yielded by the following

    1.  create a javascript evaluation environment with one global variable `s`, an array of objects.
        The *i*th elements of `s` should be a javascript object representation of a dereference of the *i*th node reference in the `antecedents` if *i* is less than the number of antecedent elements;
        otherwise it should be a javascript object representation of the (*i* − *n*)th instantiated node, where *n* is the number of antecedent elements.
    
    2.  execute the javascript code
    3.  yield the value produced by the last statement in the javascript code


-   replacing the `source` value (which is `-1` in Node Templates)
    with a reference to the Inference.

-   replacing any non-negative integers
    with the node reference in the `support` list at the integer's index 
    if the integer is smaller than the length of the `support` list;
    otherwise with a reference to the node at index _i_ − _n_ 
    of the instantiated node list being constructed,
    where _n_ is the length of the `support` list.

A Claim SHOULD NOT reference an Inference in its `source` field 
UNLESS it is an element of the instantiated node list of that Inference.

Known string and datum values
=============================

The bulk of this section and its subsections will be added at a later time.
What is present here is just an initial draft of a few example specifications.

Known Tag `key`s
----------------

To convert a known Tag `key` in this section to a URI or IRI,
prepend it with the URI or IRI of this specification concatenated with "/Tag/key#"

| `key` | constraints | meaning |
|-------|-------------|---------|
| distinct | ≥ 2 elements in `of`<br/>each elements referenced in `of` either a Subject or and Aggregated Subject | no two elements of `of` refers to the same real-world subject |
| equivalent | ≥ 2 elements in `of` | the elements in `of` contain equivalent information (a hint that only one element needs to be displayed) |
| same | ≥ 2 elements in `of`<br/>each elements referenced in `of` either a Subject or and Aggregated Subject | all of the elements of `of` refer to the same real-world subject |
| unsupported | `of` contains a single reference to a Claim, Inference, or Derivation | the `source` does not make this Claim or the `support` does not justify this Inference or Derivation  | 
| wrong | `of` cannot include both a "wrong" Tag and any element of that Tag's `of`<br/>`of` cannot include a Subject or Aggregated Subject | the indicated node asserts a falsehood | 

Known Property `key`s
---------------------

To convert a known Property `key` in this section to a URI or IRI,
prepend it with the URI or IRI of this specification concatenated with "/Property/key#"

| `key` | constraints | meaning |
|-------|-------------|---------|
| type | `value` is an enumerated type value<br/>`of` is a Subject or Aggregated Subject | what type of entity this subject refers to |

### Known Enumerated Type Values

To convert a value in this section to a URI or IRI,
prepend it with the URI or IRI of this specification concatenated with "/Subject/type/"

| `IRI` | supertypes | meaning |
|-------|------------|---------|
| person | | a human or putative human |
| event | | a distinct and recognisable (time period, location) pair or set thereof effecting or attempting to effect a common end |
| birth | event | the coming forth of an infant from a womb, sometimes expanded to include related events such as conception, naming, legal registration, etc. |
| death | event | the transition from "alive" to "dead" |
| place | | a location or region defined politically, geographically, socially, etc. |

The "supertype" column indicates an Expectation that derives supertype from subtype.

Known Connection `key`s
-----------------------

To convert a known Tag `key` in this section to a URI or IRI,
prepend it with the URI or IRI of this specification concatenated with "/Connection/key#"

| `key` | constraints | meaning |
|-------|-------------|---------|
| update | `of` and `value` have same node type | an error or inaccuracy in `of` has been corrected in `value` |



Partial Implementation and Extension
====================================

For various reasons, implementers should expect to interact with other software
that has adopted only portions of this specification
and that have extended this specification in various ways.
This section discusses appropriate ways to interact with partial implementations and extensions.


Policies and Suggestions
------------------------

The set of elements in each node type is fixed, and MUST NOT be extended or reduced.

The set of node types MAY be extended, though it is RECOMMENDED that extensions be handled with custom `key`s in Properties, Connections, and/or Tags instead.
It is RECOMMENDED that software ignore new node types when receiving extended data.

### Partial implementations and data reductions

#### Removing unwanted or private information

Software MAY chose to ignore any node or set of nodes when receiving a dataset provided that all nodes that depend upon the ignored node(s) are also ignored.

Software MAY chose to omit any node or set of nodes while sending a dataset provided that all nodes that depend upon the omitted node(s) are also omitted.

#### Removing Expectations and Inferences

Software sending/receiving a dataset MAY chose to omit/ignore all Inferences and Expectations
by converting each Inference into a Derivation
with the same `support` as the Inference
and a textual representation of the Expectation as the Derivation's `reason`.
If the software is unable to convert the Expectation to text, it SHOULD use the text "(reason omitted)" as the Derivation's `reason`.

#### Removing all reasoning

Software sending/receiving a dataset MAY chose to omit/ignore all reasoning,
reducing the dataset to a "belief snapshot".
Belief snapshots are datasets that satisfy the following constraints:

-   Only five node types are used: Subject, Property, Connection, Derivation, and OutRef
-   Each Claim's `source` references a Derivation
-   Each element of each Derivation's `support` references an OutRef

General datasets may be reduced to a belief snapshot via the following process:

1.  Create a Subject for each group of Nodes linked by "same" Tags.
	The new Subject's `source` should be
	a Derivation whose `support` is the set of all OutRef nodes in any of the original nodes' dependencies.
2.  Copy all Properties and Connections that pointed to any of the the original nodes to point to the new node.
	The new Properties' and Connections' `source`s should be 
	Derivations whose `support`s are the sets of all OutRef nodes in any of the original nodes' dependencies.
3.  Replacing any set of Properties or Connections that differ only in `source` 
	with a single node having as its `source`
	a Derivation with the union of the original claims' `source`s' `support` as the new Derivation's `support`.

### Handling Unsupported Values

#### Optional predicates and producers

When software that supports Expectation nodes receives a dataset including Expectations that contain predicates that the receiving software cannot evaluate (i.e., their contained scripts use aspects of the language that the software's Javascript engine does not support),
it is RECOMMENDED that the recieving software keep the Expectation as received
but treat the unsupported predicates as being equivalent to _Top_ when error-checking received data
and as equivalent to `false` when deciding if the user can create new Inferences based on the Expectation.

When software that supports Expectation nodes receives a dataset including Expectations that contain producers that the receiving software cannot evaluate,
it is RECOMMENDED that the receiving software keep the Expectation as received
but treat them as unchecked (like a Derivation).

#### Constrained vocabularies

If software desires a particular set of keys in its data,
it is RECOMMENDED that a set of term-normalising Expectations be used
to automatically create new Inferences and normalised terms upon receipt of new datasets,
keeping the pre-normalised terms as `support` for the Inferences.

For software not supporting Expectations, it is RECOMMENDED that Derivations be used,
resulting in a dataset equivalent to creating the Expectations and Inferences
and then applying the process outlined in [Removing Expectations and Inferences](#removing-expectations-and-inferences).
For software not supporting any reasoning, it is RECOMMENDED that 
the resulting dataset be equivalent to creating the Expectations and Inferences
and then applying the process outlined in [Removing all reasoning](#removing-all-reasoning).

If the software does not know how to normalise a particular term,
or does but does not care to represent the resulting normalised term,
it MAY omit the term and its containing node, as outlined in [Removing unwanted or private information](#removing-unwanted-or-private-information).

#### Constrained representations and topics

If software desires a particular structure in its data,
for example desiring birthdate Properties instead of birth Subjects,
it is RECOMMENDED that a set of Expectations be used
to automatically create new Inferences and normalised structures upon receipt of new datasets.

In some cases, software may decide to discard additional nodes;
for example, software that tracks human relationships might chose to omit
not only a "species":"horse" Property
but also the Subject to which it is attached
and all other claims `of` that Subject.


Discussion
==========

Node Identity and Redundancy
----------------------------

Node identity is determined only [equality](#equality).
There is no notion of durable URI, UUID, GUID, DOI, PURL, ARK,
or any other form of unique, durable identifier for a node 
in this specification,
nor should one be introduced
unless that identifier can be uniquely determined from the contents of the node alone,
as for example a hash-based UUID.
Equality does require access to the dependencies of a node,
but because dependencies are acyclic that is always a finite set.

Because node contents determine node identity,
there is no intrinsic notion of versions of a node,
nor of updating or editing a node's contents.
The concept that one node is an update of another should be expressed using a Connection with `key` "update".

Because node identity is determined only by content,
omitting nodes does not impede collaboration;
the non-omitted nodes can still be linked to new research and shared between clients.

Some of the practices for removing information
outlined in [Partial implementations and data reductions](#partial-implementations-and-data-reductions)
can "replace" nodes
or introduce new nodes containing information that is redundant with previous data.
Replacement doesn't actually modify or destroy any existing nodes,
instead creating new, similar nodes.
When the recipient sends these new nodes back to the sender
the sender will now have nodes expressing redundant information.
Retaining these redundant nodes in the dataset does not impede research
provided that the user interface prevents redundant information from distracting the user,
as (for example) displaying only one copy of a set of Properties that differ only in `source`.
New nodes containing no new information could also be omitted upon receipt, assuming that the recipient can identify those nodes as redundant.



Term Normalisation
------------------

When extracting the contents of an external source as a set of Claim nodes citing the OutRef of the external source,
there is a quality-of-use expectation that the `slug` of each Subject
be a description (in any language) of some specific part of the source that references the Subject in question; 
and that the various Properties and Connections reflect, as closely as possible,
the language of the source 
(e.g., using a Connection with key `Papa` if that is the wording used to describe the relationship in the source).

Term normalisation should then happen by introducing new, normalised-term properties and connections 
`source`d to Inference or Derivation nodes that use the non-normalised terms as (part of) their `support`.
In many cases, the `support` should also include the contextual information used in normalisation
such as the date, location, and/or language of the underlying document.


Inferences vs. Derivations, and the creation of Expectations
------------------------------------------------------------

Inferences and Derivations are both intended to fulfil the same objective:
to represent the fact that some claims were constructed using reasoning supported by other pieces of information within the dataset.
Either Source type can be used to model any reasoning that is based on a finite set of other nodes.
They differ primarily in that Inferences are more complicated to implement
but are more language-independent and machine-understandable than Derivations.

The Derivation node does not require the use of Expectations
and their associated Node Queries, Node Templates, matches, predicates, and producers.
However, they leave the `reason` in machine-opaque human-language text,
meaning that they are not readily analysed by the computer,
are language-dependent,
may lack necessary detail to communicate the reasoning process,
may contain text not in keeping with their support,
etc.

The Inference node is a machine-understandable representation of reasoning.
However, Inferences require the creation of Expectation nodes,
with the corresponding complexities of Node Queries, Node Templates, matches, predicates, and producers.

It is anticipated that some users of tools supporting Inference nodes
will not be comfortable creating their own Expectations manually
and will not be willing to limit themselves to a set of pre-built Expectations.
The following set of steps is provided as a suggested way of assisting users
in creating nontrivial Expectation nodes without needing to understand their underlying structure.

1.  Have the user specify the nodes they intend to infer
    and select the nodes they recognise as the support for their inference.
    
    (This step would also be present in creating Derivations, and would be followed by asking the user to type a free-text explanation.)

2.  Build a set of candidate antecedents
    from the nodes identified as support, 
    augmented with any nodes referenced by the nodes identified as being inferred that are not already in the antecedent or inferred collections.

3.  For each field of each candidate antecedent (except the fields of an OutRef; see below),
    ask the user "is this field important to your reasoning?"
    If the answer is "no", replace the field with `-1` (if a node reference) or predicate _Top_ (otherwise).
    If the answer is "yes", add the referenced node to the candidate antecedent set (if a node reference) or replace it with a literal predicate (otherwise).
    Continue until you have asked about all nodes of all fields in the antecedent set.
    
    For the `details` field of an OutRef,
    instead ask this question of each pair in the set
    and, if the answer is "no", remove the pair completely.

    For the `parents` field of an OutRef,
    instead ask this question of each pair in the set
    and, if the answer is "no", remove the pair completely;
    if it the answer is "yes", add the referenced OutRef to the candidate antecedent set.
    
    You could optionally ask additional questions to build more advanced predicates;
    for example, to create a `x == s[i][j]` predicate you could ask something like 
    "we notice that these two nodes both have the same `value`;
    is that same-value characteristic important?"

4.  Order the candidate antecedents
    such that, if node _A_ is in node _B_'s dependencies and both are antecedents
    then _A_ appears before _B_.
    Use this ordering to populate the `antecedent` list and to replace node references with integers.
    
    There may be many orderings that satisfy this requirement.
    Future versions of this specification might recommend a particular canonical ordering in order to reduce variability in Expection nodes.

5.  Organise the inferred nodes
    such that, if node _A_ is in node _B_'s dependencies and both are being inferred
    then _A_ appears before _B_.
    Use this ordering to populate the `consequent` list and to replace node references with integers.
    Additionally, replace all inferred node `source` values with `-1`, as required by the definition of Node Template.

    There may be many orderings that satisfy this requirement.
    Future versions of this specification might recommend a particular canonical ordering in order to reduce variability in Expection nodes.

