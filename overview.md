<style type="text/css">
body{font-family:sans-serif;}
code{font-family:monospace;font-size:100%;background-color:#eee;border:1px solid #aaa; padding:1px; border-radius:4px;}
table{border-collapse:collapse;}
tr:nth-child(2n){background-color:#ddd;}
td,th{padding:0.25ex 1ex;}
th{border-bottom:1px solid black;}
</style>


This is a work-in-progress overview of polygenea, a model for representing family history and genealogical research.

An important missing component of this specification is a discussion of standard values
for the various string and datum fields.
This draft does discuss "same", "wrong", "distinct", and "misinterpreted" Tag `key`s
but the discussion is incomplete and the names of those keys may change in future drafts.


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
Cmp | OPTIONAL | either datum with a media type that has defined order<br/>or string | _∙_, an operator from the set {`<`, `≤`, `=`, `≠`, `≥`, `>`}<br/>_x_, a value of the same type as the predicate | for a datum: _v_ ∙ _x_ under the media type's defined ordering<br/>for a string: _v_ ∙ _x_ under a lexicographical ordering
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
Lit | REQUIRED | any | _x_, a value | _x_
Lookup | RECOMMENDED | any | _i_, an integer<br/>_j_, an integer | the _j_th value of the _i_th tuple in _s_
Match | OPTIONAL | string | _f_, a string producer<br/>_r_, a regex<br/>_i_, an integer | the contents of the _i_th matching group after matching _f_(_s_) with _r_, or the empty string if it does not match or the match has no such group
Slice | OPTIONAL | string or list | _f_, a string or list producer<br/>_i_, an integer<br/>_j_, an integer | the zero-indexed subsequence of _f_(_s_) from _i_ (inclusive) to _j_ (exclusive).<br/>Negative indices have the length of the sequence added to them before dereferencing;<br/>out-of-bounds indices are clamped to bounds; and<br/>negative-width subsequences return the empty sequence
Cat | OPTIONAL | string or list | _x_, a list of string or list producers | the sequence produced by concatenating the sequenced returned by each elements of _x_ in order
Union | OPTIONAL | set | _x_, a set of set producers | a set containing every value contained in any of the sets produced by each of the elements of _x_
Intersect | OPTIONAL | set | _x_, a set of set producers | a set containing those values that are in every set produced by each element of _x_
Script | OPTIONAL | any | _x_, a datum defining a single function in some programming language | the value returned when evaluating the function in _x_ with argument _s_

Implementations supporting the `Script` predicate type SHOULD ensure that all scripts are side-effect-free and return a value of the appropriate type for every input.


The Nine Node Types
-------------------

Reasoning is a set of nodes.  Node types are expressed in the following hierarchy:

-   Node
    
    -   Claim
        
        -   Subject
            
            A tuple `(slug, source)`, where
            
            -   `slug` is a datum used to uniquely identify a single subject discussed by a source.
            -   `source` is a reference to a Source
        
        -   Property
        
            A tuple `(key, of, value, source)`, where
            
            -   `key` is a string
            -   `of` is a reference to a Node
            -   `value` is a datum
            -   `source` is a reference to a Source
        
        -   Connection

            A tuple `(key, of, value, source)`, where
            
            -   `key` is a string
            -   `of` is a reference to a Node
            -   `value` is a reference to a Node
            -   `source` is a reference to a Source
        
        -   Tag
        
            A tuple `(key, of, source)`, where
            
            -   `key` is a string
            -   `of` is a nonempty set of references to Nodes
            -   `source` is a reference to a Source
            
            Some `key`s impose restrictions on the `of` set:
            
            -   `key`s "same" and "distinct" require sets of cardinality ≥ 2.
            -   `key` "misinterpreted" requires a set of cardinality = 1 
                and that the node referenced in the set be a Claim.
            -   `key` "wrong" may neither reference a Subject nor another "wrong" Tag
                in the `of`,
                unless also including the `source` of that Claim.
            -   if `key` "wrong" references another "wrong" Tag,
                it may not also reference anything referenced by that referenced "wrong" 
                Tag.
            -   `key` "wrong" may not reference an Aggregated Subject or an OutRef.
    
    -   Aggregated Subject
        
        A singleton tuple `(parts)`, where
        
        -   `parts` is a references to a Tag node with `key` "same" 
            and only Subject and/or Aggregated Subject nodes referenced by its `of`.
    
    -   Source
        
        -   OutRef
            
            A tuple `(partOf, details)`, where
            
            -   `partOf` is a (possibly empty) set of references to OutRefs
            -   `details` is a set of string ↦ datum pairs
        
        -   Derivation
            
            A tuple `(support, reason)`, where
            
            -   `support` is a non-empty set of references to Claims
            -   `reason` is a string
        
        -   Inference
            
            A tuple `(support, reason)`, where
            
            -   `support` is an list of references to nodes
            -   `reason` is a reference to an Expectation
            
            It must be the case that the `support` matches the `antecedent` of 
            the `reason`.
    
    -   Expectation
        
        A tuple `(antecedent, consequent)`, where
        
        -   `antecedent` is an list of Node Queries
        -   `consequent` is an list of Node Templates
        
        A Node Query may be any Node except that
        
        -   In place of any value of non-reference type _X_,
            the Node Query has an _X_ predicate.
        
        -   In place of any value of reference type _X_, the Node Query must 
            have either the special value `-1` or a non-negative integer _i_ 
            such that the _i_th node in the `antecedent` is of a type that may 
            be referenced by _X_ and _i_ is strictly smaller than the index of 
            the Node Query that contains it.
        
        A Node Template may be any Claim except that
        
        -   It has the special value `-1` as its `source`
        
        -   In place of any value of non-reference type _X_, 
            the Node Template has an _X_ producer.

        -   In place of any value of reference type _X_,
            the Node Query must have a non-negative integer _i_ 
            such that both (1) the _i_th node in the list created by concatenating `antecedent` and `consequent` 
            is of a type that may be referenced by _X_
            and (2) _i_ is strictly smaller than the index of the Node Template that contains it in that concatenate list.



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
    both (1) they are the same type of node 
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
    2.  the value has a type that allows the predicate to be evaluated using the value and the list of node tuples created from the list of node references, as described in the section [Producers](#producers)
    3.  evaluating the predicate with the value and the list of tuples yields the value `true`
    
    Optionally, if the predicate definition does not utilise its list of tuples parameter (i.e., _s_ is not referenced in the table in section [Producers](#producers))
    then the predicate MAY be said to match if condition 3 alone would be true for any list of tuples supplied.
    
-   The integer value `-1` matches any node reference.

-   A set of integers matches a set of node references if
    each integer in the set of integers
    matches some node reference in the set of node references.
    Note that it is *not* necessary for every node reference to match some integer.

-   A non-negative integer _i_ matches a node reference _r_ if and only if all of the following are true:
    
    1.  there is a list of node references that was used to initiate the matching process 
    2.  there is an element of that list at index _i_
    3.  the node referenced by the element at the index _i_ is a constituent of _r_.

-   A list of node references matches a list of Node Queries if and only if 
    both (1) they have the same length
    and (2) for each valid list index for the lists, 
    the elements in the list of node reference at that index
    matches the element in the list of Node Queries at that index.

-   A Node Query matches a node reference if and only if
	both (1) the referenced node and the Node Query are of the same node type
	and (2) each field in the Node Query matches the corresponding field in the referenced node.

-   A string matches a regex if and only if the algorithm outlined
    in [ECMA 262 section 15.10.6.3](http://www.ecma-international.org/ecma-262/5.1/#sec-15.10.6.3)
    would return `true`.


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
    otherwise with a reference to the node at index _i_−_x_ 
    of the instantiated node list being constructed,
    where _x_ is the length of the `support` list.

A Claim SHOULD NOT reference an Inference in its `source` field 
UNLESS it is an element of the instantiated node list of that Inference.


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

The some of the practices for removing information
outlined in [Partial implementations and data reductions](#partial-implementations-and-data-reductions)
can "replace" nodes
or introduce new nodes containing information that is redundant with previous data.
Replacement doesn't actually modify or destroy any existing nodes,
instead creating new, similar nodes.
When the recipient sends these new nodes back to the sender
the sender will now have nodes expressing redundant information.
Keeping these redundant nodes around does not impede research
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
to represent the fact that some claims were constructed using other pieces of information.
Either can be used to model any reasoning that is based on a finite set of other nodes.
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

The Inference node is a purely machine-understandable representation of reasoning.
However, Inferences require the creation of Expectation nodes,
with the corresponding complexities of Node Queries, Node Templates, matches, predicates, and producers.

It is anticipated that some users of tools supporting Inference nodes
will not be comfortable creating their own Expectations manually
and will not be willing to limit themselves to a set of pre-built Expectations.
The following set of steps is provided as a suggested way of assisting users
in creating nontrivial Expectation nodes without needing to understand their underlying structure.

1.  Have the user specify the nodes they intend to infer
    and select the nodes they recognise as the support for their inference.
    
    If creating Derivations, ask the user to type up their `reason` and stop.
    The remaining steps are for constructing an Expectation and Inference.

2.  Build a set of candidate antecedents
    from the nodes they identified as support 
    augmented with any nodes referenced by the nodes the user identified as being inferred that are not already in the `antecedent` or inferred collections.

3.  For each field of each candidate antecedent (except `details` in an OutRef; see below),
    ask the user "is this field important to your reasoning?"
    If the answer is "no", replace the field with `-1` (if a node reference) or predicate _Top_ (otherwise).
    If the answer is "yes", add the referenced node to the candidate antecedent set (if a node reference) or replace it with a predicate _Lit_ (otherwise).
    Continue until you have asked about all nodes of all fields in the antecedent set.
    
    For the `details` field of an OutRef,
    instead ask this question of each pair in the set
    and, if the answer is "no", remove the pair completely
    to create a predicate _MapHas_.
    
    You could optionally ask additional questions to build more advanced predicates;
    for example, to create a _Same_ predicate you could ask something like 
    "we notice that these two nodes both have the same `value`;
    is that same-value characteristic important?"

4.  Order the candidate antecedents
    such that, if node _A_ is in node _B_'s dependencies and both are antecedents
    then _A_ appears before _B_.
    Use this ordering to populate the `antecedent` list and to replace node references with integers.

5.  Organise the inferred nodes
    such that, if node _A_ is in node _B_'s dependencies and both are being inferred
    then _A_ appears before _B_.
    Use this ordering to populate the `consequent` list and to replace node references with integers.
    Additionally, replace all inferred node `source` values with `-1`, as required by the definition of Node Template.


