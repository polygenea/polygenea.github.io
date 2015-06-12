Specification of Reasoning
==========================

The Nine Node Types
-------------------

Reasoning is a set of nodes.  Node types are expressed in the following 
hierarchy:

-   Node
    
    -   Claim
        
        -   Sourced Claim
        
            -   Subject
                
                A tuple `(slug, source)`, where
                
                -   `slug` is an arbitrary value to uniquely identify a single 
                    subject discussed by a source.
                -   `source` is a reference to a Source
            
            -   Property
            
                A tuple `(key, of, value, source)`, where
                
                -   `key` is a string
                -   `of` is a reference to a Node
                -   `value` is a string or a (MIME-type, bytestring) pair
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
                -   `key` "misinterpreted" requires a set of cardinality = 1 and 
                    that the node referenced in the set be a Sourced Claim.
                -   `key` "wrong" may not reference a Subject or another 
                    "wrong" Tag in the `of` unless also including the `source` of 
                    that Claim.
                -   if `key` "wrong" references another "wrong" Tag, it may not 
                    also reference anything referenced by that referenced "wrong" 
                    Tag.
                -   `key` "wrong" may not reference an Aggregated Subject.
        
        -   Aggregated Subject
            
            A singelton tuple `(parts)`, where
            
            -   `parts` is a references to a Tag node with `key` "same" and 
                only Subject and/or Aggregated Subject nodes referenced by its 
                `of`.
    
    -   Source
        
        -   OutRef
            
            A tuple `(partOf, details)`, where
            
            -   `partOf` is a (possibly empty) set of references to OutRefs
            -   `details` is a set of key↦value pairs, where each key is a 
                string and each value is either a string or a (MIME-type, 
                bytestring) pair
        
        -   Derivation
            
            A tuple `(support, reason)`, where
            
            -   `support` is a non-empty set of references to Claims
            -   `reason` is a string
        
        -   Inference
            
            A tuple `(support, reason)`, where
            
            -   `support` is an ordered list of references to nodes
            -   `reason` is a reference to an Expectation
            
            It must be the case that the `support` matches the `antecedent` of 
            the `reason`.
    
    -   Expectation
        
        A tuple `(antecedent, consequent)`, where
        
        -   `antecedent` is an ordered list of Node Queries
        -   `consequent` is an ordered list of Node Templates
        
        A Node Query may be any Node except that
        
        -   in place of any value of non-reference type _X_, the Node Query may 
            have a function of type _X_→Boolean.
        
        -   in place of any value of reference type _X_, the Node Query must 
            have either the special value `-1` or a non-negative integer _i_ 
            such that the _i_th node in the `antecedent` is of a type that may 
            be referneced by _X_ and _i_ is strictly smaller than the index of 
            the Node Query that contains it.
        
        A Node Template may be any Source Claim except that
        
        -   it has the special value `-1` as its `source`
        
        -   in place of any value of reference type _X_, the Node Query must 
            have a non-negative integer _i_ such that the _i_th node in the 
            list created by concatenating `antecedent` and `consequent` is of a 
            type that may be referneced by _X_ and _i_ is strictly smaller than 
            the index of the Node Template that contains it in that 
            concatentate list.
        
        -   in place of any value of non-reference type _X_, the Node Template 
            may have a function of type (list of node references)→_X_ that is 
            defined for any list of node references that matches `antecendent`.


Definitions and Constraints
---------------------------

The __dependencies__ of a node is defined to be the set of all nodes that the 
node references unioned with all of those nodes' dependencies, recursively.

It is required that all references be acyclic; in other words, no node may be 
included in its own dependencies.

Node and value __equality__ is defined as follows:

-   Two nodes are equal if and only if both (1) they are the same type of node 
    and (2) each field of one node is equal to the corresponding field of the 
    other node.

-   Two references are equal if and only if they refer to nodes that are equal.

-   Two sets are equal if and only if, for every element in each set there is 
    an equal element in the other set.

-   Two lists are equal if and only if both (1) they have the same length and 
    (2) for each valid list index for the lists, the elements in both lists at 
    that index are equal.

-   Two functions are equal if they are represented using the same bytes, and 
    they are not equal if there exists an input for which their results are 
    not equal.  If they are represented by different bytes but give the same 
    results for all inputs, their equality is not defined by this specification.

-   Two strings are equal if they contain the same conceptual characters in 
    the same order.

-   Two (MIME-type, bytestring) pairs are equal if they have the same 
    MIME-type and byte-for-byte equivalent bytestrings.

The __constituents__ of any node other than an Aggregated Subject node is 
defined to be the singleton set containing just that Node itself.

The __constituents__ of an Aggregated Subject node is defined to be the 
singleton set containing just the Aggregated Subject node itself unioned with 
the __constituents__ of each of the nodes referenced by its `parts`.

Node and value __matching__ is defined as follows:

-   Any pair that are equal also match

-   A function and a value match if the function can be evaluated with the 
    value as an argument and if doing so results in the Boolean value `true`.

-   The special value `-1` matches any node reference.

-   A non-negative integer _i_ matches a node reference _r_ if and only if both 
    (1) there is a list of node references that was used to initiate the
    matching process and (2) there is an element of that list at index _i_ and 
    (3) the element at the index _i_ is a constituent of _r_.

-   A list of node references matches a list of Node Queries if and only if 
    both (1) they have the same length and (2) for each valid list index for the 
    lists, the elements in the list of node refernece at that index matches the 
    element in the list of Node Queries at that index.

-   A node reference matches a Node Query if and only if both (1) the 
    referenced node and the Node Query are of the same node type, and (2) each 
    field in the referenced node matches the corresponding node in the Node 
    Query.

The __instantiated node list__ of an Inference is defined to be a list of 
Nodes having the same length as the `consequent` of the `reason` of the 
Inference such that the Node at index _i_ of the instantiated node list is 
created from the Node Template at index _i_ of the `consequent` by doing all of 
the following (in any order)

-   replacing any function values with the result of applying that function 
    with the `antecendent` of the Inference as the function's argument.

-   replacing the `source` value (which is `-1` in Node Templates) with a 
    reference to the Inference.

-   replacing any non-negative integers with the node reference in the 
    `antecedent` list at the integer's index if the integer is smaller than 
    the length of the `antecedent` list; otherwise with a reference to the node 
    at index _i_−_x_ of the instantiated node list being constructed, where _x_ 
    is the length of the `antecedent` list.

A Sourced Claim may only reference an Inference in its `source` field if it is 
an element of the instantiated node list of that Inference.


Discussion
==========

Node Identity
-------------

Node identity is determined only by the node's contents.  There is no notion 
of durable URI, UUID, GUID, or other unique, durable identifier for a node in 
this specification, nor should one be introduced unless that identifer can be 
uniquely determined from the contents of the node alone, as for example a 
hash-based UUID.

Because node contents determine node identity, there is no intrinsic notion of 
versions of a node, nor of updating or editing its contents.  The concept that 
one node is an update of another should be expressed using a Connection with 
`key` "update".


Term Normalisation
------------------

When extracting the contents of an external source as a set of Sourced Claim 
nodes citing the OutRef of the external source, there is a quality-of-use 
expectation that the `slug` of each Subject is a description (in any language) 
of some specific part of the source that references the Subject in question; 
and that the various Properties and Connections reflect, as closely as 
pssible, the language of the source (e.g., using a Connection with key `Papa` 
if that is the wording used to describe the relationship in the source).

Term normalisation should then happen by introducing new, normalised-term 
properties and connections `source`d to either Inference or Derivation nodes 
that use the non-normalised terms as (part of) their `support`.  In many cases, 
the `support` should also include some contextual information such as the 
date, location, and/or language of the underlying document.


Inferences vs. Derivations, and the creation of Expectations
------------------------------------------------------------

Inferences and Derivations are both intended to fulfill the same objective: to 
represent the fact that some claims were constructed using other pieces of 
information.

The Derivation node is flexible and does not require the relatively 
complicated support for matches and Node Queries and Node Templates and so on 
that Inferences do require.  However, they leave the `reason` in 
machine-opaque human-language text, which means they are not readily analysed 
by the computer and dependent on any users encountering the Derivation to be 
fluent in the same language and understand the way the creator of the 
Derivation chose to express their reasoning.

The Inference node, on the other hand, is purely machine-understandable 
representation of reasoning.  Like a Derivation, an Inference is flexible 
enough to be created for any finite-support inference.  However, Inferences 
require the creation of Expectation nodes.

Sophisticated Expecation nodes are possible, but many Expectations can be 
readily created using the following interactive process:

1.  Have the user specify the nodes they intend to infer and select the nodes 
    they recognize as the support for their inference.  This step would need 
    to be done for Derivation nodes as well...

2.  Build a set of candidate antecedents from the nodes they identified as support 
    augmented with any nodes referenced by the nodes they identified as being
    inferred that are not already in the `antecedent` or inferred collections.

3.  For each field of each candidate antecedents, ask the user "is this field 
    important to your reasoning?"  If the answer is "no", replace the field 
    with `-1` (if a node reference) or a predicate that always returns `true` 
    (otherwise).  If the answer is "yes", add the referenced node to the 
    candidate antecedent set (if a node reference) or leave the field as is 
    (otherwise).  Continue until you have asked about all nodes of all fields 
    in the antecedent set.
    
    You could optionally ask additional questions, such as "we notice that 
    these two nodes both have the same `value`; is that same-value 
    characteristic important?", to help build more advanced predicates, if you 
    so desire.

4.  Organize the candidate antecedents so that each node appears at a larger 
    index than any of its dependencies included that are also antecedents; then 
    replace all remaining node references with indices into the new 
    `antecedent` list.

5.  Organize the infered nodes so that each node appears at a larger index than 
    any of its dependencies included that are also infered; then replace all 
    remaining node references with indices into the new `antecedent` list 
    concatenated with the new `consequent` list, or with `-1` if the `source` 
    field.



Partial Implementation and Extension
====================================

Because some family history and genealogical software does not currently 
support formal inference at all, implementers should expect that it will take 
some time for full implementations of all features of this specification to be 
adopted.  It is also expected that many implementers will wish to add some 
extensions to the core terminology provided by this specification.  This 
section discusses appropriate ways to interact with partial implementations 
and extensions.


Removing Inferences and Expectations
------------------------------------

Software may convert all Inferences into Derivations with the same `support` 
and a textual representation of the Expectation as the `reason`.  If the 
software is unable or unwilling to convert the Expectation to text, it is 
suggested that the text "(reasoning lost during import)" be used instead.  
Once all Inferences have been replaced by Derivations, all Expectations can be 
removed.


Reducing to Conclusion + Source format
--------------------------------------

Software may remove all reasoning altogether by 

1.  Creating a Subject for each group of Subjects and Aggregated Subjects 
    linked by "same" Tags.
2.  Copying all Properties and Connections that pointed to the original nodes 
    to point to the new nodes instead.
3.  Replacing all `source`s with a pointer to a Derivation node with all of 
    the OutRef nodes in the original `source`'s dependencies as the 
    Derivation's `support` and "(reasoning lost during import)" as its 
    `reason`.
4.  Replacing any pair of Sourced Claims that differ only in `source` with a 
    single node having as its `source` a Derivation with the union of the two 
    original claim's `source`s' `support` as the new Derivation's `support`.

At the conclusion of this process, the data will be a set of Sourced Claims 
whose sources are all Derivations with only OutRefs as their `support`.  While 
this has discarded the vast majority of the work performed, it does closely 
match the structure of other family history and genealogy data formats.


Handling Unexpected `key`s and `value`s
---------------------------------------

Some software may not have the ability to process arbitrary `key` and/or 
`value` values wherever they appear.  A non-exhaustive scample of approaches 
for handling these cases follow:

-   Automatically apply a set of Expectations to convert between the provided 
    format and the desired format.  These could be as simple as term 
    normalisation (e.g., "father" implies "http://schema.org/parent") or more 
    complicated (e.g., a "birthdate" Property and some "parent" connections 
    implies a birth Subject with a "date" Property and various "parent" and 
    "child" connections between the birth and the participants, and possibly 
    also a family Subject with various Connections with its participants).

-   Simply discard the node in question; for example, if a tool does not care 
    about marriages it could simply ignore all marriage Connections and 
    Subjects and all nodes that depend on them.  If it was part of the 
    `support` for an Inference or Derivation, replace that source with a 
    Derivation using the dependencies of the discarded node as part of its 
    `support`.

-   In some cases it might be appropriate to discard the node and some of the 
    nodes on which it depends.  For example, software that is interested only 
    in human lineage would probably not have support for Property nodes 
    indicating that a Subject has "type":"pet", and if it found such would 
    probably discard not only that Property but also the described Subject and 
    everything that depends upon it.

Because node identity is determined only by content, discarding nodes does not 
impede collaboration; returning the kept subsets with any new nodes to the 
originator allows the originator to add the new nodes to their existing set, 
potentially without even realizing that some of the nodes sent were not 
returned.

A similar argument suggests that the various "replace with" approaches to 
handling unexpected or unwanted nodes.  Replacement doesn't actually modify or 
destroy any existing nodes went the replaced data is given back to the 
originator, it merely adds additional nodes which contain only information 
that could have been derived from the initially-provided information.  Keeping 
these redundent nodes around does not impede research as long as the user 
interface keeps the redundent information from distracting the user.  That 
said, redundant nodes of this kind could be discarded upon receipt without loss 
of communication ability as the next time the originator sends the data the 
same replacements will occur and the same nodes recreated.


Different sets of Node Types
----------------------------

It is hoped that most implementations based off of this specification will 
chose to implement all nine node types.  However, we anticipate that some will 
chose to implement the following subsets:

-   No Inference or Expectation (see [Removing Inferences and Expectations]() 
    above).

-   No Derivations (every Derivation can be replaced by an Inference + 
    Expectation pair by the [creation of Expectations]() discussed above; if no 
    user is involved, a minimal conversion answers "yes" for all non-reference 
    fields and "no" for all reference fields in that algorithm).
    
-   Only five node types: OutRef, Derivation, Subject, Property, and 
    Connection (see [Reducing to Conclusion + Source]() format above).

However, new node types are not anticipated and specification-compliant 
software does not need to accept input containing any node type beyond the nine 
in this standard.  If developers wish, they may make software that allows other 
node types to be present but ignores them; however, that practice is 
discouraged for security reasons: while no known vulnerability exists, 
implementations of some data standards in other fields have suffered from 
attacks based on embedding special code in the ignored part of the those 
standards.



Serialisation
=============

XML
---



JSON
----



Graphviz
--------



CBOR
----



Custom Small Strings
--------------------



