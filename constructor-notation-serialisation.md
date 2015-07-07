<style type="text/css">
body{font-family:sans-serif; counter-reset:h1;}
code{font-family:monospace;font-size:100%;background-color:#eee;border:1px solid #aaa; padding:1px; border-radius:4px;}
pre code{border:none;}
pre{display:table; border:1px solid #aaa; padding:4px; border-radius:6px;}
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

This document presents a simple UTF-8 serialisation of a dataset
as defined in the accompanying [conceptual model](overview.html).
The goal is to make a simple and compact textual representation, not to abide by any particular existing standard.

# Encoding

This document describes outputing characters.
This output stream SHALL be encoded in UTF-8 unless embedded in a data stream that unambiguously identified a different encoding.

Some tokens MAY be preceded and/or followed by any number of whitespace characters (i.e., `[ \t\n\r]*`):

If it says to output | may add preceding whitespace | may add following whitespace
-----------|------------------------------|-----------------------------
"a COMMA `,`" | Yes | Yes
"a COLON `:`" | Yes | Yes
"a LEFT PARENTHESES `(`" | No | Yes
"a RIGHT PARENTHESES `)`" | Yes | Yes
"a LEFT SQUARE BRACKET `[`" | No | Yes
"a RIGHT SQUARE BRACKET `]`" | Yes | No
"a LEFT CURLY BRACKET `{`" | No | Yes
"a RIGHT CURLY BRACKET `}`" | Yes | No
Anything else | No | No

Note if the specification does not explicitly name a character, whitespace padding is not allowed;
thus for example when [encoding a string](#encoding-a-string) says "Output _c_" no whitespace may be added even if _c_ happens to refer to a COMMA or other character in the table above.

A Dataset may be preceded and followed by arbitrary whitespace, regardless of its first and last characters.


## Encoding a Dataset

Given a dataset,

1.  Create an empty string:string mapping called _abbreviations_.
2.  Populate and encode abbreviations by repeating the following steps zero or more times:
    1.  Identify a string prefix to be shortened.  The prefix MUST NOT begin with a COLON ':' and MUST NOT already be either a key or a value in _abbreviations_.
    2.  Select a shortened form, using a valid URI `<scheme>` as onlined in [RFC 2396](http://tools.ietf.org/html/rfc2396):
        that is, something matching the regular expression `[a-zA-Z][a-zA-Z0-9\+\-\.]*`.
    3.  Output a LATIN CAPITAL LETTER N `N` and a LEFT PARENTHESES `(`.
    4.  Encode the prefix.
    5.  Output a COMMA `,`.
    6.  Encode the shortened form.
    7.  Output a RIGHT PARENTHESES `)`.
    8.  Add (prefix : shortened form) to _abbreviations_.
    
    Example: `N("http://polygenea.org/","pg")` means this encoding will abbreviate strings beginning with `http://polygenea.org/` with the shortened prefix `pg:`.

3.  Initialise _index_ as `0`.
4.  Create an empty (node reference : integer) mapping called _indexOf_.
5.  Repeat the following until every node is referenced as a key in _indexOf_:
    1.  Select a node to process such that both (1) a reference to that node is not a key in _indexOf_ and (2) all of the nodes referenced within the candidate node also referenced as a key in _indexOf_.  Call the selected node _n_.
    2.  Encode _n_.
    4.  Insert (reference to _n_ : _index_) into _indexOf_.
    5.  Increase _index_ by 1.

The letter `N` is selected because it is
not a letter that could start any other type
and suggests "namespace", the name of the similar abbreviation scheme used in XML


Because _abbreviations_ is populated one entry at a time while it is being output,
later abbreviations will be encoded using earlier ones.
Thus, the following representation of a two-entry _abbreviations_

    N("http://polygenea.org/Tag","pgt")
    N("http://polygenea.org/","pg")

could be represented more compactly if the abbreviations where selected in a different order:

    N("http://polygenea.org/","pg")
    N("pg:Tag","pgt")



## Encoding a Node, Node Template, or Node Query

Given an element _n_ to encode,

1.  Output an upper-case version of the initial letter of _n_'s type (which will match the regular expression `[ACDEIOPST]`).
2.  Output a LEFT PARENTHESES `(`.
3.  Encode the value at tuple-index 0 of _n_.
4.  If _n_ is of a type that has at least two elements in its tuple,
    1.  Output a COMMA `,`.
    2.  Encode the value at tuple-index 1 of _n_.
5.  If _n_ is of a type that has at least three elements in its tuple,
    1.  Output a COMMA `,`.
    2.  Encode the value at tuple-index 2 of _n_.
6.  If _n_ is of a type that has four elements in its tuple,
    1.  Output a COMMA `,`.
    2.  Encode the value at tuple-index 3 of _n_.
7.  Output a RIGHT PARENTHESES `)`.

## Encoding a Node Reference

Given 
a node reference _r_, and
read-access to a (node reference : integer) mapping called _indexOf_, and
read-access to an integer _index_,

1.  Let _x_ be the integer paired with _r_ in _indexOf_ (by construction, there will always be such an integer).
2.  Let _y_ be the result of computing _index_ − _x_ (by construction, this will always be a positive number).
3.  Let _dx_ be the result of computing &lceil;log<sub>10</sub>(_x_)&rceil;, which is the number of decimal digits needed to represent _x_.  If _x_ is 0, let _dx_ be 1.
4.  Let _dy_ be the result of computing 1 + &lceil;log<sub>10</sub>(_y_)&rceil;, which is one more than the number of decimal digits needed to represent _y_.
5.  If _dy_ < _dx_,
    1.  Output a HYPHEN-MINUS `-`
    2.  Encode _y_
6.  If _dx_ ≤ _dy_,
    1.  Encode _x_

## Encoding a string

Given
a string _s_ and
read-access to a (string : string) mapping called _abbreviations_,

1.  Output a QUOTATION MARK `"`
2.  If _s_ begins with a COLON `:` or a value mapped to by _abbreviations_,
    1.  Output a COLON `:`
    2.  For each character _c_ in _s_,
        2.  Output _c_
        1.  If _c_ is a QUOTATION MARK `"`, output _c_ again
3.  If _s_ begins with a key mapped from by _abbreviations_,
    1.  For each character _c_ in the value associated with that key, output _c_
    2.  Output a COLON `:`
    1.  For each character _c_ in _s_ after the prefix,
        2.  Output _c_
        1.  If _c_ is a QUOTATION MARK `"`, output _c_ again
4.  If _s_ does not begin with a COLON `:` nor key nor value in _abbreviations_,    
    1.  For each character _c_ in _s_,
        2.  Output _c_
        1.  If _c_ is a QUOTATION MARK `"`, output _c_ again
5.  Output a QUOTATION MARK `"`

The above performs minimal escaping:

-   a preceding COLON `:` is used to mean "no abbreviations here", and to escape itself in the first position of a string
-   QUOTATION MARK `"` is repeated twice to escape itself (similar to many CSV encodings)

There does not need to be any text after an abbreviated prefix.

If there are more than one possible prefixes that could be chosen in step 3,
there is a quality-of-implementation expectation that the longest such prefix be used.
Choosing a shorter prefix (or none at all, skipping to step 4) will still correctly represent the string, albeit not as concisely.

## Encoding an integer

Integer values (excepting &minus;1) are encoded using a standard base-10 representation with the ASCII decimal digits.
They always match the regular expression `-?(0|[1-9][0-9]*)`.

It is RECOMMENDED that the value −1 be encoded as the empty string, but implementations MAY use `-1` instead.

## Encoding a datum

Given a datum value _x_,

1.  Output a LEFT PARENTHESES `(`
2.  If the first tuple element (the media type string) is not "", encode it
3.  Output a COMMA `,`
4.  If the second tuple element (the language string) is not "", encode it
5.  Output a COMMA `,` and a REVERSE SOLIDUS `\`
7.  Output the Base64 encoding of the third tuple element, as described in [RFC 3548](http://tools.ietf.org/html/rfc3548), section 3
8.  Output a RIGHT PARENTHESES `)`

## Encoding a list

Given a list value _x_ containing _n_ items indexed 0 through _n_ − 1,

1.  Output a LEFT SQUARE BRACKET `[`
2.  If _n_ > 0, encode the element at index 0 of _x_
3.  If _n_ > 1,
    1.  For each integer _i_ from 1 up to _n_ — 1 in ascending order,
        1.  Output a COMMA `,`
        2.  Encode the element at index _i_ of _x_
4.  Output a RIGHT SQUARE BRACKET `]`

## Encoding a set

Given a set value _x_,

1.  Output a LEFT CURLY BRACKET `{`
2.  If there are any elements in _x_,
    1.  Encode some element _s_ from _x_
    2.  For each element _e_ of _x_ other than _s_ without repetition
        1.  Output a COMMA `,`
        2.  Encode _e_
4.  Output a RIGHT CURLY BRACKET `}`

If there is a comparison operator defined for the type of element stored in _x_,
It is RECOMMENDED that _s_ be the minimal element of _x_ and that the remaining elements be visited in ascending order.

## Encoding a key ↦ value pair

Given a pair _k_ ↦ _v_,

1.  Encode _k_
2.  Output a COLON `:`
1.  Encode _v_

## Encoding a predicate

Given a predicate of type _p_ constructed with tuple _t_ of _n_ elements,

1.  If _p_ is _Top_, there is nothing to do (encoded s zero characters)
2.  If _p_ is _Lit_, encode the single item in _t_
3.  If _p_ is not _Top_ or _Lit_,
    1.  Output a LATIN CAPITAL LETTER Q `Q`
    2.  Output the `name` of _p_, capitalised as given in [the specification](overview.html)
        (e.g., `ICmp` is LATIN CAPITAL LETTER I, LATIN CAPITAL LETTER C, LATIN SMALL LETTER M, LATIN SMALL LETTER P)
    3.  Output a LEFT PARENTHESES `(`
    4.  Encode the element at index 0 of _t_
    5.  If _n_ ≥ 2,
        1.  Output a COMMA `,`.
        2.  Encode the element at index 1 of _t_
    6.  If _n_ = 3,
        1.  Output a COMMA `,`.
        2.  Encode the element at index 2 of _t_
    7.  Output a RIGHT PARENTHESES `)`

The letter `Q` is selected because it is
not a letter that could start any other type
and suggests "query", the class of nodes that contain predicates.

## Encoding a producer

Given a producer of type _p_ constructed with tuple _t_ of _n_ elements,

1.  If _p_ is _Lit_, encode the single item in _t_
2.  If _p_ is not _Top_ or _Lit_,
    1.  Output a LATIN CAPITAL LETTER B `B`
    2.  Output the `name` of _p_, capitalised as given in [the specification](overview.html)
        (e.g., `Cat` is LATIN CAPITAL LETTER C, LATIN SMALL LETTER A, LATIN SMALL LETTER T)
    3.  Output a LEFT PARENTHESES `(`
    4.  Encode the element at index 0 of _t_
    5.  If _n_ ≥ 2,
        1.  Output a COMMA `,`.
        2.  Encode the element at index 1 of _t_
    6.  If _n_ = 3,
        1.  Output a COMMA `,`.
        2.  Encode the element at index 2 of _t_
    7.  Output a RIGHT PARENTHESES `)`

The letter `B` is selected because it is
not a letter that could start any other type
and suggests "builder", an approximate synonym for "producer"

## Example

The following encodes one possible form of an inference that someone named "John" is male.

    N("http://gedcomx.org/","gx")
    N("gx:v1/","gx1")
    O({},{"type":"imagination","author":"Luther Tychonievich"})
    S(2,"example person")
    P(2,"gx1:Name",3,"John Doe")
    E([S(,)
      ,P(,"gx1:Name",0,QRegex("Joh?n( |$)"))]
     ,[P(,"gx1:Gender",0,"gx:Male")])
    I([3,4],5)
    P(6,"gx1:Gender",3,"gx:Male")
