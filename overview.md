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



Specification of Reasoning
==========================

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
:   a (string, string, blob) tuple.
    Each string specifies part of the interpretation of the blob;
    they mean (in order)
    
    -   Media type, as specified by [RFC 4288](http://tools.ietf.org/html/rfc4288),
        such as "application/javascript".
        If "", assumed to be "text/plain;charset=UTF-8".
        If media type alone does not specify an unambiguous interpretation of the bytes, clarifying parameters (as the charset of various text types)
        are REQUIRED.
        
        Alternatively, this may be an absolute IRI describing a type,
        such as "http://gedcomx.org/date/v1".
        Because absolute IRIs always contain a COLON `:`
        and media types never contain a COLON,
        it is unambiguous which form of media type is being specified.

    -   Language, as specified by [IETF BCP 47](http://tools.ietf.org/html/bcp47); 
        if "", assumed to be locale-independent (i.e., 
        may be processed using the default locale of the processor).

__reference__
:   indicating exactly one node within the same dataset.
    Because they do not span datasets,
    references may have an implementation-defined format.
    
    The phrase "reference to a _X_", where _X_ is Node or one of Node's subtypes,
    means the reference indicates an element of type _X_ or one of _X_'s subtypes.

__integer__
:   a signed integer.
    Every implementations must be able to represent (at least)
    all integers in the interval [−1, 32767).
    During serialisation or deserialisation the maximum number of nodes 
    an implementation can handle may be constrained by the largest magnitude integer
    that the implementation can represent.

__regex__
:   a string containing 
    either (a) a _RegularExpressionLiteral_ 
    or (b) a _RegularExpressionBody_, 
    both defined in [ECMA 262 section 7.8.5](http://www.ecma-international.org/ecma-262/5.1/#sec-7.8.5).
    Because _RegularExpressionLiteral_s always begin with a SOLIDUS `/` and _RegularExpressionBody_s never begin with a SOLIDUS, it is unambiguous which is present in a string.
    Unlike the ECMA 262 standard, empty regular expressions are allowed.
    If only a _RegularExpressionBody_ is provided, it is treated as if it had no flags.
    
    Values of type regexs are used in some predicates and producers and behave as documented in [ECMA 262 section 15.10](http://www.ecma-international.org/ecma-262/5.1/#sec-15.10).
    

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

Predicate and Producer Datatypes
--------------------------------

### Predicates

This specification uses the term "_X_ predicate", where _X_ is a datatype,
to mean a function that takes as its parameters

-   a single value (called _v_ in the table below) of type _X_, and
-   a list of node tuples (called _s_ in the table below)

and returns either `true` or `false`.

In this specification, predicates are only evaluated when matching a list of Node Queries
against a list of node references;
the value of _s_ is created by taking the prefix of that list of node references,
up to and including the node reference being matched against the Node Query containing the predicate,
and dereferencing each.

Inside a predicate, neither _s_ nor _v_ may be modified
and any node references inside tuples inside _s_ are treated as opaque types 
(i.e., they may not be dereferenced).

name | status | types | defined with | returns `true` when
-----|--------|-------|--------------|--------------------
Top | REQUIRED | any | (nothing) | always
Lit | REQUIRED | any | _x_, a value of the same type as the predicate | _v_ equals _x_
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

Implementations supporting the `Script` predicate type SHOULD ensure that all scripts are side-effect-free and return a Boolean value for every input.

### Producers

This specification uses the term "_X_ producer", where _X_ is a datatype,
to mean a function that takes as its parameter 
a list of node tuples (called _s_ in the table below)
and returns a value of type _X_.

In this specification, producers are only evaluated when evaluating a Node Template
in the context of a list of node references;
the value of _s_ is created by dereferencing each element of that list.

Inside a producer, _s_ may not be modified
and any node references inside tuples inside _s_ are treated as opaque types 
(i.e., they may not be dereferenced).

name | status | types | defined with | returns
-----|--------|-------|--------------|--------
Lit | REQUIRED | any | _x_, a value of the same type as the producer | _x_
Lookup | RECOMMENDED | any | _i_, an integer<br/>_j_, an integer | the _j_th value of the _i_th tuple in _s_
Match | OPTIONAL | string | _f_, a string producer<br/>_r_, a regex<br/>_i_, an integer | the contents of the _i_th matching group after matching _f_(_s_) with _r_, or the empty string if it does not match or the match has no such group
Slice | OPTIONAL | string or list | _f_, a string or list producer<br/>_i_, an integer<br/>_j_, an integer | the zero-indexed subsequence of _f_(_s_) from _i_ (inclusive) to _j_ (exclusive).<br/>Negative indices have the length of the sequence added to them before dereferencing;<br/>out-of-bounds indices are clamped to bounds; and<br/>negative-width subsequences return the empty sequence
Cat | OPTIONAL | string or list | _x_, a list of string or list producers | the sequence produced by concatenating the sequenced returned by each elements of _x_ in order
Union | OPTIONAL | set | _x_, a set of set producers | a set containing every value contained in any of the sets produced by each of the elements of _x_
Intersect | OPTIONAL | set | _x_, a set of set producers | a set containing those values that are in every set produced by each element of _x_
Script | OPTIONAL | any | _x_, a datum defining a single function in some programming language | the value returned when evaluating the function in _x_ with argument _s_

Implementations supporting the `Script` predicate type SHOULD ensure that all scripts are side-effect-free and return a value of the appropriate type for every input.


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
Node Queries are used to structure the preconditions or antecedents of an Expectation and contain [predicates](#predicates).

For each Claim type there is a Node Template type.
Node Templates are used to structure the postconditions or consequent of an Expectation and contain [producers](#producers).

Each type is described in terms of its constituent fields;
each field is given both an index and a name.
Indices are used to create tuples from nodes and streamline the presentation of [Predicate and Producer Datatypes](#predicate-and-producer-datatypes).
Names are used in most other places in this specification because they are more suggestive of the purpose of the various fields.

By design, each field name corresponds to the same index everywhere it appears.
To achieve that end, each Node Template has a dummy `source` field (index 0, always value `-1`).


### Nodes

| Node subtype | is a | Properties (index. name : type) | Constraints |
|--------------|------|-----------------------------------|-------------|
| Aggregated Subject | | 0. `parts` : reference to a Tag | `parts`'s `key` MUST be "same" <br/> Each node referenced in `parts`'s `key`'s `of` MUST be either a Subject or and Aggregated Subject |
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

| Node Query subtype | Properties (index. name : type) |
|--------------------|-----------------------------------|
| Connection Query | 0. `source` : integer<br/>1. `key` : string predicate <br/>2. `of` : int<br/>3. `value` : int |
| Derivation Query | 0. `support` : set of ints<br/>1. `reason` : string predicate |
| Expectation Query | _exactly the same an Expectation_ |
| Inference Query | 0. `support` : list of ints<br/>1. `reason` : integer |
| OutRef Query | 0. `parents` : set of (string ↦ int) pairs<br/>1. `details` : set of (string ↦ datum predicate) pairs |
| Property Query | 0. `source` : integer<br/>1. `key` : string predicate <br/>2. `of` : int<br/>3. `value` : datum predicate |
| Subject Query | 0. `from` : integer |
| Tag Query | 0. `source` : integer<br/>1. `key` : string predicate <br/>2. `of` : set of ints |

The Subject Query matches both Subjects and Aggregated Subjects; there is not a separate Aggregated Subject Query.

Each integer value in a Node Query MUST either be `-1` or satisfy all of the following constraints:

-   be inside a Node Query that is inside a list of Node Queries
-   be ≥ 0
-   be < the index of the containing Node Query in its containing list 

If an int value treated as an index into the containing list does not index a node with a type
that the same field in the corresponding Node could reference then the Node Query will never match any Node.
Node Queries containing "mistyped" integers of like this probably indicate user error, but are not forbidden by this specification.

### Node Templates

| Node Template subtype | Properties (index. name : type or value) |
|-----------------------|-----------------------------------|
| Connection Template | 0. `source` : the integer value `-1`<br/>1. `key` : string producer <br/>2. `of` : int<br/>3. `value` : int |
| Property Template | 0. `source` : the integer value `-1`<br/>1. `key` : string producer <br/>2. `of` : int<br/>3. `value` : string producer |
| Subject Template | 0. `source` : the integer value `-1`<br/>1. `slug` : string producer |
| Tag Template | 0. `source` : the integer value `-1`<br/>1. `key` : string producer <br/>2. `of` : (set of ints) producer |

Each integer value in a Node Template MUST either be `-1` or satisfy all of the following constraints:

-   be inside a Node Template that is inside the `consequent` of an Expectation that has _n_ elements in its `antecedent`
-   be ≥ 0
-   be < _n_ + the the index of the containing Node Template the containing Expectation's `consequent`



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

-   Two predicates are equal if they have the same type and are defined with equal values.
    They are not equal if there exists some inputs for which they give different results.
    If they are defined differently but give the same result for all inputs, their equality is not specified by this specification.

-   Two producers are equal if they have the same type and are defined with equal values.
    They are not equal if there exists some inputs for which they give non-equal results.
    If they are defined differently but give equal result for all inputs, their equality is not specified by this specification.

-   Two strings are equal if they contain the same characters in the same order.

-   Two data are equal if and only if
    either (1) there is an official notion of eqaulity for the given media type(s)
    and the blobs are equal under than definition
    or (2) the media types are equal and the blobs have the same bytes in the same order.

### Constituents

The __constituents__ of any node that is not an Aggregated Subject node
is defined to be the singleton set containing just that Node itself.

The __constituents__ of an Aggregated Subject node 
is defined to be the set containing the Aggregated Subject node itself 
and the elements of the _constituents_ of each of the nodes referenced in the `of` field
of the node referenced by the Aggregated Subject's `parts` field.

### Matching

Node and value __matching__ is defined as follows:

-   A predicate and a value match if all of the following are true:
    
    1.  the matching is occurring within the context of matching a list of Node Queries and a list of node references
    2.  the value has a type that allows the predicate to be evaluated using the value and the list of node tuples created from the list of node references, as described in the section [Predicates](#predicates)
    3.  evaluating the predicate with the value and the list of tuples yields the value `true`
    
    Optionally, if the predicate definition does not utilise its list of tuples parameter (i.e., _s_ is not referenced in the table in section [Predicates](#predicates))
    then the predicate MAY be said to match if condition 3 alone would be true for any list of tuples supplied.
    
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

-   replacing any producer values 
    with the result of applying that function 
    using a list of tuples created by dereferencing the elements of the `antecedent` of the Inference as the function's argument.

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
| distinct | ≥ 2 elements in `of`<br/>all elements referenced in `of` have same Node subtype (except may mix Subject and Aggregated Subject) | no two elements of `of` refers to the same real-world subject or contain equivalent information |
| same | ≥ 2 elements in `of`<br/>all elements referenced in `of` have same Node subtype (except may mix Subject and Aggregated Subject) | the elements of `of` are fully equivalent: the same real-world subject, alternative presentations of the same information, etc. |
| unsupported | `of` contains a single reference to a Claim, Inference, or Derivation | the `source` does not make this Claim or the `support` does not justify this Inference or Derivation  | 
| wrong | `of` cannot include both a "wrong" Tag and any element of that Tag's `of`<br/>`of` cannot include a Subject or Aggregated Subject | the indicated node asserts a falsehood | 

Known Property `key`s
---------------------

To convert a known Property `key` in this section to a URI or IRI,
prepend it with the URI or IRI of this specification concatenated with "/Property/key#"

| `key` | constraints | meaning |
|-------|-------------|---------|
| type | value is an enumerated type value<br/>`of` is a Subject or Aggregated Subject | what type of entity this subject refers to |

### Enumerated Type Values

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

1.  Create a Subject for each group of Subjects and Aggregated Subjects linked by "same" Tags.
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

When software that supports Expectation nodes receives a dataset including Expectations that contain predicates that the receiving software cannot evaluate,
it is RECOMMENDED that the recieving software keep the Expectation as received
but treat the extra predicate as being equivalent to _Top_ when error-checking received data
and as equivalent to _Not_(_Top_) when deciding if the user can create new Inferences based on the Expectation.

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
    If the answer is "yes", add the referenced node to the candidate antecedent set (if a node reference) or replace it with a predicate _Lit_ (otherwise).
    Continue until you have asked about all nodes of all fields in the antecedent set.
    
    For the `details` field of an OutRef,
    instead ask this question of each pair in the set
    and, if the answer is "no", remove the pair completely
    to create a predicate _MapHas_.

    For the `parents` field of an OutRef,
    instead ask this question of each pair in the set
    and, if the answer is "no", remove the pair completely;
    if it the answer is "yes", add the referenced OutRef to the candidate antecedent set.
    
    You could optionally ask additional questions to build more advanced predicates;
    for example, to create a _Same_ predicate you could ask something like 
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

