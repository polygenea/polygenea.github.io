This is a work-in-progress overview of polygenea, a model for representing family history and genealogical research.


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
    
    1.  Media type, as specified by [RFC 4288](http://tools.ietf.org/html/rfc4288); 
        if "", assumed to be "text/plain;charset=UTF-8".
        If media type alone does not specify an unambiguous interpetation of the bytes, clarifying parameters (as the charset of various "text" types)
        are REQUIRED. 
    3.  Language, as specified by [IETF BCP 47](http://tools.ietf.org/html/bcp47); 
        if "", assumed to be locale-independent (i.e., 
        may be processed using the default locale of the processor)

__reference__
:   indicating exactly one node within the same dataset.
    Because they do not span datasets,
    references may have an implementation-defined format.
    
    The phrase "reference to a _X_", where _X_ is Node or one of Node's subtypes,
    means the reference indicates an element of type _X_ or one of _X_'s subtypes.

__integer__
:   a signed integer.
    Every implementations must be able to represent (at least)
    all integers in the interval \[−1, 32766\].
    During serialisation or deserialisation the maximum number of nodes 
    an implemenation can handle may be constrained by the largest magnitude integer
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
	or, if the elements are given names in the specification, by name.

**_X_↦_Y_ pair**
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
-   a list of node tuples (called _m_ in the table below)

and returns either `true` or `false`.

Predicates are only ever evaluated when matching a list of Node Queries
against a list of node references;
the value of _m_ is created by taking the prefix of that list of node references
up to and including the node reference being matches against the Node Query containing the predicate
and derefrencing each.

Inside a predicate, neither _m_ nor _v_ may be modified
and any node references inside tuples inside _m_ are treated as opaque types 
(i.e., they may not be dereferenced).

name | status | types | defined with | returns `true` when
-----|--------|-------|--------------|--------------------
Top | REQUIRED | any | (nothing) | always
Lit | REQUIRED | any | a constant value _x_ | _v_ equals _x_
Same | RECOMMENDED | any | two indices _i_ and _j_ | _v_ equals the _j_th value of the _i_th tuple in _m_
Has | RECOMMENDED | set or list of _X_ | _X_ predicate _f_ | _f_(_e_) is `true` for any _e_ in _v_
MapHas | RECOMMENDED | set of string ↦ datum pairs | set _x_ of string ↦ datum predicate pairs | for each (_a_↦_b_) in _x_, there is a (_c_↦_d_) pair in _v_ such that _a_ = _c_ and _b_(_d_) is `true`
Cmp | RECOMMENDED | string | a constant value _x_ and an operator _∙_ from {`<`, `≤`, `=`, `≠`, `≥`, `>`} | _v_ ∙ _x_ under a lexicographical ordering
Cmp | OPTIONAL | datum with MIME type that has defined order | a constant value _x_ and an operator _∙_ from {`<`, `≤`, `=`, `≠`, `≥`, `>`} | _v_ ∙ _x_ under that ordering
ICmp | OPTIONAL | as `Cmp` | as `Cmp`, but two indices _i_ and _j_ instead of constant _x_ | as `Cmp`, but using the _j_th value of the _i_th tuple in _m_ instead of _v_
Regex | OPTIONAL | string | a constant PCRE _r_ | _r_ matches _v_
Len | OPTIONAL | set or list | a constant integer _x_ and an operator _∙_ from {`<`, `≤`, `=`, `≠`, `≥`, `>`} | (the number of elements in _v_) ∙ _x_
And | OPTIONAL | any (call it _X_) | two _X_ predicates | both predicates are _true_
Or | OPTIONAL | any (call it _X_) | two _X_ predicates | at least one predicates is _true_
Not | OPTIONAL | any (call it _X_) | another _X_ predicates | the predicate is `false`
Script | OPTIONAL | any | a datum with major type "script" defining a single function | evaluating the function in the script returns `true`

Implemenations supporting the `Script` predicate type SHOULD ensure that all scripts are side-effect-free and return a Boolean value for every input.

### Producers

This specification uses the term "_X_ producer", where _X_ is a datatype,
to mean a function that takes as its parameter 
a list of node tuples (called _m_ in the table below)
and returns a value of type _X_.

Producers are only ever evaluated when evaluating a Node Template
in the context of a list of node references;
the value of _m_ is created dereferencing each element of that list.

Inside a producer, _m_ may not be modified
and any node references inside tuples inside _m_ are treated as opaque types 
(i.e., they may not be dereferenced).

name | status | types | defined with | returns
-----|--------|-------|--------------|--------
Lit | REQUIRED | any | a constant value _x_ | _x_
Lookup | RECOMMENDED | any | two indices _i_ and _j_ | the _j_th value of the _i_th tuple in _m_
Match | OPTIONAL | string | a constant PCRE _r_<br/>and an integer _i_<br/>and a string producer _f_ | the contents of the _i_th matching group after matching _f_(_m_) with _r_, or the empty string if it does not match or the match has no such group
Slice | OPTIONAL | string or list | a string or list producer _f_ and two integers _i_ and _j_ | the zero-indexed subsequence of _f_(_m_) from _i_ (inclusive) to _j_ (exclusive)<br/>negative indices have the length of the sequence added to them before derefrencing<br/>out-of-bounds indices are clamped to bounds<br/>negative-width subsequences return the empty sequence
Cat | OPTIONAL | string or list | two string or list producers | the concatenation of the strings or lists produced by the two producers
Union | OPTIONAL | set | two set producers | the union of the sets produced by the two producers
Intersect | OPTIONAL | set | two set producers | the intersection of the sets produced by the two producers
Script | OPTIONAL | any | a datum with major type "script" defining a single function | the value returned when evaluating the function in the script

Implemenations supporting the `Script` predicate type SHOULD ensure that all scripts are side-effect-free and return a value of the appropriate type for every input.



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
                and that the node referenced in the set be a Sourced Claim.
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
            -   `details` is a set of key↦value pairs,
                where each key is a string and each value is a datum
        
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

The __dependencies__ of a node is defined to be the set of all nodes that the 
node references unioned with all of those nodes' dependencies, recursively.

It is REQUIRED that all references be acyclic; in other words, no node may be 
included in its own dependencies.

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
    If they are defined differently but give the same result for all inputs, their equality is not specified by this standard.

-   Two producers are equal if they have the same type and are defined with equal values.
    They are not equal if there exists some inputs for which they give non-equal results.
    If they are defined differently but give equal result for all inputs, their equality is not specified by this standard.

-   Two strings are equal if they contain the same conceptual characters in 
    the same order.

-   Two data are equal if and only if
    either (1) there is an official notion of eqaulity for the given MIME-type
    and the blobs are equal under than definition
    of (2) the strings are equal and the blobs have the same bytes in the same order.

The __constituents__ of any node *except* an Aggregated Subject node
is defined to be the singleton set containing just that Node itself.

The __constituents__ of an Aggregated Subject node 
is defined to be the singleton set containing the Aggregated Subject node itself 
and the elements of the __constituents__ of each of the nodes referenced in the `of` field
of the node referenced by the Aggregated Subject's `parts` field.

Node and value __matching__ is defined as follows:

-   A predicate and a value match if all of the following are true:
    
    1.  the matching is occurring within the context of matching a list of Node Queries and a list of node references
    2.  the value has a type that allows the predicate to be evaluated using the value and the list of node tuples created from the list of node references, as described in the section [Producers](#producers)
    3.  evaluating the predicate with the value and the list of tuples yields the value `true`
    
    Optionally, if the predicate does not utilize its list of tuples parameter
    then the predicate may be said to match if condition 3 alone would be true for any list of tuples supplied.
    
-   The special value `-1` matches any node reference.

-   A set of integers matches a set of node references if
    each integer in the set of integers
    matches some node reference in the set of node references.

-   A non-negative integer _i_ matches a node reference _r_ if and only if all of the following are true:
    
    1.  there is a list of node references that was used to initiate the matching process 
    2.  there is an element of that list at index _i_
    3.  the node referenced by the element at the index _i_ is a constituent of _r_.

-   A list of node references matches a list of Node Queries if and only if 
    both (1) they have the same length
    and (2) for each valid list index for the lists, 
    the elements in the list of node reference at that index
    matches the element in the list of Node Queries at that index.

-   A node reference matches a Node Query if and only if
	both (1) the referenced node and the Node Query are of the same node type
	and (2) each field in the referenced node matches the corresponding node in the Node Query.

The __instantiated node list__ of an Inference is defined to be 
a list of Nodes
having the same length as the `consequent` of the `reason` of the Inference
such that the Node at index _i_ of the instantiated node list 
is created from the Node Template at index _i_ of the `consequent`
by doing all of the following (in any order)

-   replacing any producer values 
    with the result of applying that function 
    using the `antecedent` of the Inference as the function's argument.

-   replacing the `source` value (which is `-1` in Node Templates)
    with a reference to the Inference.

-   replacing any non-negative integers
    with the node reference in the `support` list at the integer's index 
    if the integer is smaller than the length of the `support` list;
    otherwise with a reference to the node at index _i_−_x_ 
    of the instantiated node list being constructed,
    where _x_ is the length of the `support` list.

A Sourced Claim may only reference an Inference in its `source` field 
if it is an element of the instantiated node list of that Inference.


Discussion
==========

Node Identity
-------------

Node identity is determined only by the node's contents.
There is no notion of durable URI, UUID, GUID, or other unique, durable identifier for a node 
in this specification,
nor should one be introduced 
unless that identifier can be uniquely determined from the contents of the node alone,
as for example a hash-based UUID.

Because node contents determine node identity,
there is no intrinsic notion of versions of a node,
nor of updating or editing its contents.
The concept that one node is an update of another should be expressed using a Connection with `key` "update".


Term Normalisation
------------------

When extracting the contents of an external source as a set of Sourced Claim nodes citing the OutRef of the external source,
there is a quality-of-use expectation that the `slug` of each Subject
be a description (in any language) of some specific part of the source that references the Subject in question; 
and that the various Properties and Connections reflect, as closely as possible,
the language of the source 
(e.g., using a Connection with key `Papa` if that is the wording used to describe the relationship in the source).

Term normalisation should then happen by introducing new, normalised-term properties and connections 
`source`d to either Inference or Derivation nodes that use the non-normalised terms as (part of) their `support`.
In many cases, the `support` should also include some contextual information
such as the date, location, and/or language of the underlying document.


Inferences vs. Derivations, and the creation of Expectations
------------------------------------------------------------

Inferences and Derivations are both intended to fulfil the same objective:
to represent the fact that some claims were constructed using other pieces of information.

The Derivation node is flexible and does not require the relatively 
complicated support for matches and predicates and producers, etc.,
which Inferences do require.
However, they leave the `reason` in machine-opaque human-language text, which means that
they are not readily analysed by the computer,
are language-dependent,
may lack necessary detail or contain text not in keeping with their support,
etc.

The Inference node, on the other hand, is 
a purely machine-understandable representation of reasoning.
Like a Derivation, an Inference is flexible enough to be created for any finite-support inference.
However, Inferences require the creation of Expectation nodes.

Sophisticated Expectation nodes are possible, particularly with `Script`-type predicates and producers,
but many Expectations can be readily created using the following interactive process:

1.  Have the user specify the nodes they intend to infer
    and select the nodes they recognise as the support for their inference.
    This step would be done for Derivation nodes as well...

2.  Build a set of candidate antecedents
    from the nodes they identified as support 
    augmented with any nodes referenced by the nodes the user identified as being inferred that are not already in the `antecedent` or inferred collections.

3.  For each field of each candidate antecedent,
    ask the user "is this field important to your reasoning?"
    If the answer is "no", replace the field with `-1` (if a node reference) or _Top_ (otherwise).
    If the answer is "yes", add the referenced node to the candidate antecedent set (if a node reference) or replace it with a _Lit_ (otherwise).
    Continue until you have asked about all nodes of all fields in the antecedent set.
    
    You could optionally ask additional questions,
    such as "we notice that these two nodes both have the same `value`;
    is that same-value characteristic important?",
    to help build more advanced predicates, if you so desire.

4.  Organise the candidate antecedents
    such that, if node _A_ is in node _B_'s dependencies and both are antecedents
    then _A_ appears before _B_.
    Use this ordering to populate the `antecedent` list and to replace node references with integers.

5.  Organise the inferred nodes
    such that, if node _A_ is in node _B_'s dependencies and both are being inferred
    then _A_ appears before _B_.
    Use this ordering to populate the `consequent` list and to replace node references with integers.
    Additionally, replace all infered node `source` values with `-1`, as required by the definition of Node Template.



Partial Implementation and Extension
====================================

For various reasons, implementers should expect to interact with other software
that has adopted only portions of this standard
and that have extended this standard in various ways.
This section discusses appropriate ways to interact with partial implementations and extensions.


Policies and Suggestions
------------------------

The set of elements in each node type is fixed, and MUST NOT be extended or reduced.

The set of node types MAY be extended, though it is RECOMMENDED that extensions be handled with custom `key`s in Properties, Connections, and/or Tags instead.
It is RECOMMENDED that software ignore new node types when recieving extended data.

Software MAY chose to ignore any node upon receipt of a dataset provided that all nodes that depend upon it are also ignored.

Software MAY chose to omit any node while sending a dataset provided that all nodes that depend upon it are also omitted.

Software MAY chose to omit or ignore all Inferences and Expectations
by converting each Inference into a Derivation 
with the same `support` and a textual representation of the Expectation as the `reason`.
If the software is unable to convert the Expectation to text, it SHOULD use the text "(reasoning omitted)", or equivalent text in another language, as the `reason`.

Software MAY chose to omit or ignore all reasoning
and reduce the dataset to a set of Subjects, Properties, Connections, Derivations, and SourceReferences
where every Claim's `source` is a Derivation and every element of each Derivation's `support` is an OutRef.
And outline of a process for reducing data to this form is:

1.  Create a Subject for each group of Subjects and Aggregated Subjects linked by "same" Tags.
	The new Subject's `source` should be
	a Derivation whose `support` is the set of all OutRef nodes in any of the original nodes' dependencies.
2.  Copy all Properties and Connections that pointed to any of the the original nodes to point to the new node.
	The new Properties' and Connections' `source`s should be 
	Derivations whose `support`s are the sets of all OutRef nodes in any of the original nodes' dependencies.
3.  Replacing any set of Properties or Conenctions that differ only in `source` 
	with a single node having as its `source`
	a Derivation with the union of the original claims' `source`s' `support` as the new Derivation's `support`.

When software that supports Expectation nodes receives a dataset including Expectations that utilize predicates that the recieving software cannot evaluate,
it is RECOMMENDED that the recieving software keep the Expectation as recieved
but treat the extra predicate as being equivalent to _Top_ when error-checking received data
and as equivalent to _Not_(_Top_) when deciding if the user can create new Inferences based on the Expectation.

When software that supports Expectation nodes receives a dataset including Expectations that utilize producers that the recieving software cannot evaluate,
it is RECOMMENDED that the recieving software keep the Expectation as recieved
but treat them as unchecked (like a Derivation).

If software desires a particular set of keys in its data,
it is RECOMMENDED that a set of term-normalizing Expectations be used
to automatically create new Inferences and normalized terms upon receipt of new datasets.

If software desires a particular sructure in its data,
for example desiring birthdate Properties instead of birth Subjects,
it is RECOMMENDED that a set of Expectations be used
to automatically create new Inferences and normalized structures upon receipt of new datasets.

In some cases, software may decide to discard additional nodes;
for example, software that tracks human relationships might chose to omit
not only a "species":"horse" Property
but also the Subject to which it is attached
and all other claims `of` that Subject.


Identity, Redundency, and Node-Creating Changes
-----------------------------------------------

Because node identity is determined only by content,
discarding nodes does not impede collaboration;
returning the kept subsets with any new nodes to the originator
allows the originator to add the new nodes to their existing set, 
potentially without even realizing that some of the nodes sent were not returned.

Some of the suggestions in the previous section
permit or suggest the replacement of nodes
or the creation of new nodes
containing information that is redundent with previous data.
Replacement doesn't actually modify or destroy any existing nodes:
they are values and if the sender of data does not chose to discard them
they cannot be modified or destroyed by the recipient.
When the recipient sends back data, it may contain new deriviative nodes;
however, the information in these nodes, while redundent, is not new.
Keeping these redundant nodes around does not impede research
provided that the user interface prevents redundant information from distracting the user.
If nodes are clearly redundent, the originator's software could also omit them upon reciept of the redundent data.




