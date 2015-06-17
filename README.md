<p>This is an alpha version of a specification of a model for representing family history and genealogical research.</p>
<p>The central goal of this model is the representation of reasoning processes, as realised by the Inference, Derivation, and Expectation node types. A secondary goal is to ensure, at a data model level, that it is impossible to have two communicating parties think they understand one another when in fact they have different versions of some element of their shared universe of discourse. To reach these goals, the model uses a representation of beliefs about the past that somewhat resembles RDF, in that information about a given entity is stored outside of the entity. Unlike RDF, each of these Claim elements has a source, representing where it came from; the structure and presence of these sources is key to achieving the central goal of this model. Additionally, Claims and other elements are defined not to have identity (a.k.a, they are compare-by-value, bit-copiable, identity=value, immutable, etc.); by definition, changing the contents of any element actually creates a new element, leaving the old one present and unchanged. The notion that a different element is intended as an update of another element is itself a claim made during reasoning and represented as such in this model.</p>
<p>This document is an alpha release intended to present a self-consistent and fairly complete specification that can be used to understand how the various conceptual pieces of this approach could fit together and to form a starting point for discussions of this approach. Many times while writing this document the optimal choice was not evident; rather than list each of these decision points, this document simply makes decisions so that the presentation can hold together as a whole, optimal or not.</p>
<h1 id="representation-of-reasoning">Representation of Reasoning</h1>
<h2 id="basic-datatypes">Basic Datatypes</h2>
<dl>
<dt><strong>character</strong></dt>
<dd><p>an atomic unit of text as specified by <code>ISO/IEC 10646</code>.</p>
</dd>
<dt><strong>string</strong></dt>
<dd><p>a finite-length sequence of characters.</p>
</dd>
<dt><strong>octet</strong></dt>
<dd><p>a sequence of eight bits, each either 0 or 1.</p>
</dd>
<dt><strong>blob</strong></dt>
<dd><p>a finite-length sequence of octets.</p>
</dd>
<dt><strong>datum</strong></dt>
<dd><p>a (string, string, blob) tuple. Each string specifies part of the interpretation of the blob; they mean (in order)</p>
<ul>
<li><p>Media type, as specified by <a href="http://tools.ietf.org/html/rfc4288">RFC 4288</a>, such as &quot;application/javascript&quot;. If &quot;&quot;, assumed to be &quot;text/plain;charset=UTF-8&quot;. If media type alone does not specify an unambiguous interpretation of the bytes, clarifying parameters (as the charset of various text types) are REQUIRED.</p>
<p>Alternatively, this may be an absolute IRI describing a type, such as &quot;http://gedcomx.org/date/v1&quot;. Because absolute IRIs always contain a COLON <code>:</code> and media types never contain a COLON, it is unambiguous which form of media type is being specified.</p></li>
<li><p>Language, as specified by <a href="http://tools.ietf.org/html/bcp47">IETF BCP 47</a>; if &quot;&quot;, assumed to be locale-independent (i.e., may be processed using the default locale of the processor).</p></li>
</ul>
</dd>
<dt><strong>reference</strong></dt>
<dd><p>indicating exactly one node within the same dataset. Because they do not span datasets, references may have an implementation-defined format.</p>
<p>The phrase &quot;reference to a <em>X</em>&quot;, where <em>X</em> is Node or one of Node's subtypes, means the reference indicates an element of type <em>X</em> or one of <em>X</em>'s subtypes.</p>
</dd>
<dt><strong>integer</strong></dt>
<dd><p>a signed integer. Every implementations must be able to represent (at least) all integers in the interval [−1, 32767). During serialisation or deserialisation the maximum number of nodes an implementation can handle may be constrained by the largest magnitude integer that the implementation can represent.</p>
</dd>
<dt><strong>regex</strong></dt>
<dd><p>a string containing either (a) a <em>RegularExpressionLiteral</em> or (b) a <em>RegularExpressionBody</em>, both defined in <a href="http://www.ecma-international.org/ecma-262/5.1/#sec-7.8.5">ECMA 262 section 7.8.5</a>. Because <em>RegularExpressionLiteral</em>s always begin with a SOLIDUS <code>/</code> and <em>RegularExpressionBody</em>s never begin with a SOLIDUS, it is unambiguous which is present in a string. Unlike the ECMA 262 standard, empty regular expressions are allowed. If only a <em>RegularExpressionBody</em> is provided, it is treated as if it had no flags.</p>
<p>Values of type regexs are used in some predicates and producers and behave as documented in <a href="http://www.ecma-international.org/ecma-262/5.1/#sec-15.10">ECMA 262 section 15.10</a>.</p>
</dd>
</dl>
<h2 id="composite-datatypes">Composite Datatypes</h2>
<dl>
<dt><strong>set</strong> of <em>X</em></dt>
<dd><p>(where <em>X</em> is a datatype): an unordered collection of zero or more elements of type <em>X</em>, containing no duplicates.</p>
</dd>
<dt><strong>list</strong> of <em>X</em></dt>
<dd><p>(where <em>X</em> is a datatype): an ordered sequence of zero or more elements of type <em>X</em>, possibly containing duplicates. Given a list of <em>n</em> elements, the first element is identified by the index 0, and each subsequent element by the index (1 + index-of-previous-element).</p>
</dd>
<dt><strong>tuple</strong></dt>
<dd><p>A fixed, ordered group of values, each with a given type. Tuple values may be referenced by index (starting with 0, like a <em>list</em>) or, if the individual elements are given names in the specification, by name. Thus, e.g., the first elements of the tuple of a Property node may be referred to as either the <code>key</code> element or element <code>0</code>.</p>
</dd>
<dt><strong><em>X</em> ↦ <em>Y</em> pair</strong></dt>
<dd><p>A tuple of two elements. If <em>X</em> and <em>Y</em> are types, they specify the types of the elements. Otherwise, <em>X</em> and <em>Y</em> are the names of the elements and types are given elsewhere in the specification text.</p>
</dd>
</dl>
<h2 id="predicate-and-producer-datatypes">Predicate and Producer Datatypes</h2>
<h3 id="predicates">Predicates</h3>
<p>This specification uses the term &quot;<em>X</em> predicate&quot;, where <em>X</em> is a datatype, to mean a function that takes as its parameters</p>
<ul>
<li>a single value (called <em>v</em> in the table below) of type <em>X</em>, and</li>
<li>a list of node tuples (called <em>s</em> in the table below)</li>
</ul>
<p>and returns either <code>true</code> or <code>false</code>.</p>
<p>In this specification, predicates are only evaluated when matching a list of Node Queries against a list of node references; the value of <em>s</em> is created by taking the prefix of that list of node references, up to and including the node reference being matched against the Node Query containing the predicate, and dereferencing each.</p>
<p>Inside a predicate, neither <em>s</em> nor <em>v</em> may be modified and any node references inside tuples inside <em>s</em> are treated as opaque types (i.e., they may not be dereferenced).</p>
<table>
<thead>
<tr class="header">
<th align="left">name</th>
<th align="left">status</th>
<th align="left">types</th>
<th align="left">defined with</th>
<th align="left">returns <code>true</code> when</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">Top</td>
<td align="left">REQUIRED</td>
<td align="left">any</td>
<td align="left">(nothing)</td>
<td align="left">always</td>
</tr>
<tr class="even">
<td align="left">Lit</td>
<td align="left">REQUIRED</td>
<td align="left">any</td>
<td align="left"><em>x</em>, a value of the same type as the predicate</td>
<td align="left"><em>v</em> equals <em>x</em></td>
</tr>
<tr class="odd">
<td align="left">MapHas</td>
<td align="left">REQUIRED</td>
<td align="left">set of string ↦ datum pairs</td>
<td align="left"><em>x</em>, a set of string ↦ (datum predicate) pairs</td>
<td align="left">for each (<em>a</em> ↦ <em>b</em>) in <em>x</em>, there is a (<em>c</em> ↦ <em>d</em>) pair in <em>v</em> such that <em>a</em> equals <em>c</em> and <em>b</em>(<em>d</em>) is <code>true</code></td>
</tr>
<tr class="even">
<td align="left">Same</td>
<td align="left">RECOMMENDED</td>
<td align="left">any</td>
<td align="left"><em>i</em>, an integer<br/><em>j</em>, an integer</td>
<td align="left"><em>v</em> equals the <em>j</em>th value of the <em>i</em>th tuple in <em>s</em></td>
</tr>
<tr class="odd">
<td align="left">Has</td>
<td align="left">RECOMMENDED</td>
<td align="left">set or list of <em>X</em></td>
<td align="left"><em>f</em>, an <em>X</em> predicate</td>
<td align="left"><em>f</em>(<em>e</em>, <em>s</em>) is <code>true</code> for any <em>e</em> in <em>v</em></td>
</tr>
<tr class="even">
<td align="left">SetHas</td>
<td align="left">RECOMMENDED</td>
<td align="left">set of <em>X</em></td>
<td align="left"><em>x</em>, a set of <em>X</em> predicates</td>
<td align="left">for each <em>f</em> in <em>x</em> there is a <em>e</em> in <em>v</em> such that <em>f</em>(<em>e</em>, <em>s</em>) is <code>true</code></td>
</tr>
<tr class="odd">
<td align="left">Cmp</td>
<td align="left">OPTIONAL</td>
<td align="left">either string or datum with a media type that has defined order</td>
<td align="left"><em>∙</em>, an operator from the set {<code>&lt;</code>, <code>≤</code>, <code>=</code>, <code>≠</code>, <code>≥</code>, <code>&gt;</code>}<br/><em>x</em>, a value of the same type as the predicate</td>
<td align="left">for a datum: <em>v</em> ∙ <em>x</em> under the media type's defined ordering<br/>for a string: <em>v</em> ∙ <em>x</em> under a lexicographical ordering</td>
</tr>
<tr class="even">
<td align="left">ICmp</td>
<td align="left">OPTIONAL</td>
<td align="left">as <code>Cmp</code></td>
<td align="left"><em>∙</em>, an operator from the set {<code>&lt;</code>, <code>≤</code>, <code>=</code>, <code>≠</code>, <code>≥</code>, <code>&gt;</code>}<br/><em>i</em>, an integer<br/><em>j</em>, an integer</td>
<td align="left">as <code>Cmp</code>, but using the <em>j</em>th value of the <em>i</em>th tuple in <em>s</em> instead of <em>v</em></td>
</tr>
<tr class="odd">
<td align="left">Regex</td>
<td align="left">OPTIONAL</td>
<td align="left">string</td>
<td align="left"><em>r</em>, a regex</td>
<td align="left"><em>r</em> matches <em>v</em></td>
</tr>
<tr class="even">
<td align="left">Len</td>
<td align="left">OPTIONAL</td>
<td align="left">set or list</td>
<td align="left"><em>∙</em>, an operator from the set {<code>&lt;</code>, <code>≤</code>, <code>=</code>, <code>≠</code>, <code>≥</code>, <code>&gt;</code>}<br/><em>x</em>, an integer</td>
<td align="left">((the number of elements in <em>v</em>) ∙ <em>x</em>) is <code>true</code></td>
</tr>
<tr class="odd">
<td align="left">And</td>
<td align="left">OPTIONAL</td>
<td align="left">any (call it <em>X</em>)</td>
<td align="left"><em>x</em>, a set of <em>X</em> predicates</td>
<td align="left"><em>f</em>(<em>v</em>, <em>s</em>) is <code>true</code> for all <em>f</em> in <em>x</em></td>
</tr>
<tr class="even">
<td align="left">Or</td>
<td align="left">OPTIONAL</td>
<td align="left">any (call it <em>X</em>)</td>
<td align="left"><em>x</em>, a set of <em>X</em> predicates</td>
<td align="left"><em>f</em>(<em>v</em>, <em>s</em>) is <code>true</code> for at least one <em>f</em> in <em>x</em></td>
</tr>
<tr class="odd">
<td align="left">Not</td>
<td align="left">OPTIONAL</td>
<td align="left">any (call it <em>X</em>)</td>
<td align="left"><em>f</em>, an <em>X</em> predicate</td>
<td align="left"><em>f</em>(<em>v</em>, <em>s</em>) is <code>false</code></td>
</tr>
<tr class="even">
<td align="left">Script</td>
<td align="left">OPTIONAL</td>
<td align="left">any</td>
<td align="left"><em>x</em>, a datum defining a single function in some programming language</td>
<td align="left">evaluating the function in <em>x</em> with arguments <em>v</em> and <em>s</em> returns <code>true</code></td>
</tr>
</tbody>
</table>
<p>Implementations supporting the <code>Script</code> predicate type SHOULD ensure that all scripts are side-effect-free and return a Boolean value for every input.</p>
<h3 id="producers">Producers</h3>
<p>This specification uses the term &quot;<em>X</em> producer&quot;, where <em>X</em> is a datatype, to mean a function that takes as its parameter a list of node tuples (called <em>s</em> in the table below) and returns a value of type <em>X</em>.</p>
<p>In this specification, producers are only evaluated when evaluating a Node Template in the context of a list of node references; the value of <em>s</em> is created by dereferencing each element of that list.</p>
<p>Inside a producer, <em>s</em> may not be modified and any node references inside tuples inside <em>s</em> are treated as opaque types (i.e., they may not be dereferenced).</p>
<table>
<thead>
<tr class="header">
<th align="left">name</th>
<th align="left">status</th>
<th align="left">types</th>
<th align="left">defined with</th>
<th align="left">returns</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">Lit</td>
<td align="left">REQUIRED</td>
<td align="left">any</td>
<td align="left"><em>x</em>, a value of the same type as the producer</td>
<td align="left"><em>x</em></td>
</tr>
<tr class="even">
<td align="left">Lookup</td>
<td align="left">RECOMMENDED</td>
<td align="left">any</td>
<td align="left"><em>i</em>, an integer<br/><em>j</em>, an integer</td>
<td align="left">the <em>j</em>th value of the <em>i</em>th tuple in <em>s</em></td>
</tr>
<tr class="odd">
<td align="left">Match</td>
<td align="left">OPTIONAL</td>
<td align="left">string</td>
<td align="left"><em>f</em>, a string producer<br/><em>r</em>, a regex<br/><em>i</em>, an integer</td>
<td align="left">the contents of the <em>i</em>th matching group after matching <em>f</em>(<em>s</em>) with <em>r</em>, or the empty string if it does not match or the match has no such group</td>
</tr>
<tr class="even">
<td align="left">Slice</td>
<td align="left">OPTIONAL</td>
<td align="left">string or list</td>
<td align="left"><em>f</em>, a string or list producer<br/><em>i</em>, an integer<br/><em>j</em>, an integer</td>
<td align="left">the zero-indexed subsequence of <em>f</em>(<em>s</em>) from <em>i</em> (inclusive) to <em>j</em> (exclusive).<br/>Negative indices have the length of the sequence added to them before dereferencing;<br/>out-of-bounds indices are clamped to bounds; and<br/>negative-width subsequences return the empty sequence</td>
</tr>
<tr class="odd">
<td align="left">Cat</td>
<td align="left">OPTIONAL</td>
<td align="left">string or list</td>
<td align="left"><em>x</em>, a list of string or list producers</td>
<td align="left">the sequence produced by concatenating the sequenced returned by each elements of <em>x</em> in order</td>
</tr>
<tr class="even">
<td align="left">Union</td>
<td align="left">OPTIONAL</td>
<td align="left">set</td>
<td align="left"><em>x</em>, a set of set producers</td>
<td align="left">a set containing every value contained in any of the sets produced by each of the elements of <em>x</em></td>
</tr>
<tr class="odd">
<td align="left">Intersect</td>
<td align="left">OPTIONAL</td>
<td align="left">set</td>
<td align="left"><em>x</em>, a set of set producers</td>
<td align="left">a set containing those values that are in every set produced by each element of <em>x</em></td>
</tr>
<tr class="even">
<td align="left">Script</td>
<td align="left">OPTIONAL</td>
<td align="left">any</td>
<td align="left"><em>x</em>, a datum defining a single function in some programming language</td>
<td align="left">the value returned when evaluating the function in <em>x</em> with argument <em>s</em></td>
</tr>
</tbody>
</table>
<p>Implementations supporting the <code>Script</code> predicate type SHOULD ensure that all scripts are side-effect-free and return a value of the appropriate type for every input.</p>
<h2 id="the-eight-node-types">The Eight Node Types</h2>
<p>This specification defines several concrete types. These form a type hierarchy:</p>
<ul>
<li>Node
<ul>
<li>Claim
<ul>
<li>Subject: identifying a single (subject of discussion, source discussing it) pair</li>
<li>Property: a single piece of information about a node</li>
<li>Connection: a directed connection between two nodes</li>
</ul></li>
<li>Source
<ul>
<li>OutRef: identifying some information source external to this specification</li>
<li>Derivation: a free-text description of a step in the reasoning processes</li>
<li>Inference: a structured description of a step in the reasoning processes</li>
</ul></li>
<li>Aggregated Subject: a single subject inferred to be discussed by many source</li>
<li>Expectation: structured descriptions of trends, inference rules, and external knowledge</li>
</ul></li>
</ul>
<p>For each Node type there is a Node Query type, with the exception that Subject and Aggregated Subject nodes share a single Node Query type. Node Queries are used to structure the preconditions or antecedents of an Expectation and contain <a href="#predicates">predicates</a>.</p>
<p>For each Claim type there is a Node Template type. Node Templates are used to structure the postconditions or consequent of an Expectation and contain <a href="#producers">producers</a>.</p>
<p>Each type is described in terms of its constituent fields; each field is given both an index and a name. Indices are used to create tuples from nodes and streamline the presentation of <a href="#predicate-and-producer-datatypes">Predicate and Producer Datatypes</a>. Names are used in most other places in this specification because they are more suggestive of the purpose of the various fields.</p>
<p>By design, each field name corresponds to the same index everywhere it appears. To achieve that end, each Node Template has a dummy <code>source</code> field (index 0, always value <code>-1</code>).</p>
<h3 id="nodes">Nodes</h3>
<table>
<thead>
<tr class="header">
<th align="left">Node subtype</th>
<th align="left">is a</th>
<th align="left">Properties (index. name : type)</th>
<th align="left">Constraints</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">Aggregated Subject</td>
<td align="left"></td>
<td align="left">0. <code>parts</code> : reference to a Tag</td>
<td align="left"><code>parts</code>'s <code>key</code> MUST be &quot;same&quot; <br/> Each node referenced in <code>parts</code>'s <code>key</code>'s <code>of</code> MUST be either a Subject or and Aggregated Subject</td>
</tr>
<tr class="even">
<td align="left">Connection</td>
<td align="left">Claim</td>
<td align="left">0. <code>source</code> : reference to a Source<br/>1. <code>key</code> : string <br/>2. <code>of</code> : reference to a Node<br/>3. <code>value</code> : int</td>
<td align="left">some <code>key</code>s make constraints on <code>of</code> and/or <code>value</code></td>
</tr>
<tr class="odd">
<td align="left">Derivation</td>
<td align="left">Source</td>
<td align="left">0. <code>support</code> : set of references to Nodes<br/>1. <code>reason</code> : string</td>
<td align="left"></td>
</tr>
<tr class="even">
<td align="left">Expectation</td>
<td align="left"></td>
<td align="left">0. <code>antecedent</code> : list of Node Queries<br/>1. <code>consequent</code> : list of Node Templates</td>
<td align="left"><em>See below</em></td>
</tr>
<tr class="odd">
<td align="left">Inference</td>
<td align="left">Source</td>
<td align="left">0. <code>support</code> : list of references to Nodes<br/>1. <code>reason</code> : reference to an Expectation</td>
<td align="left"><code>reason</code>'s <code>antecedent</code> SHOULD <a href="#matching">match</a> <code>support</code><br/><em>See below</em></td>
</tr>
<tr class="even">
<td align="left">OutRef</td>
<td align="left">Source</td>
<td align="left">0. <code>parents</code> : set of (string ↦ reference to an OutRef) pairs<br/>1. <code>details</code> : set of (string ↦ datum) pairs</td>
<td align="left">some string values make constraints on their corresponding reference or datum values</td>
</tr>
<tr class="odd">
<td align="left">Property</td>
<td align="left">Claim</td>
<td align="left">0. <code>source</code> : reference to a Source<br/>1. <code>key</code> : string <br/>2. <code>of</code> : reference to a Node<br/>3. <code>value</code> : datum</td>
<td align="left">some <code>key</code>s make constraints on <code>of</code> and/or <code>value</code></td>
</tr>
<tr class="even">
<td align="left">Subject</td>
<td align="left">Claim</td>
<td align="left">0. <code>source</code> : reference to a Source<br/>1. <code>slug</code> : string</td>
<td align="left"><code>slug</code> SHOULD identify a single subject within the source</td>
</tr>
<tr class="odd">
<td align="left">Tag</td>
<td align="left">Claim</td>
<td align="left">0. <code>source</code> : reference to a Source<br/>1. <code>key</code> : string <br/>2. <code>of</code> : set of references to Nodes</td>
<td align="left">some <code>key</code>s make constraints on <code>of</code></td>
</tr>
</tbody>
</table>
<p>All references MUST be acyclic and MUST refer to another node in the same dataset. If some user insists some Claim has no source, it is RECOMMENDED that the <code>source</code> reference an OutRef with <code>details</code> including (&quot;type&quot; ↦ &quot;user assertion&quot;) and (&quot;user&quot; ↦ however the user is identified).</p>
<p>A Claim SHOULD NOT reference an Inference in its <code>source</code> field UNLESS it is an element of the <a href="#instantiating-nodes-from-inferences">instantiated node list</a> of that Inference.</p>
<p>Some constraints on Node Queries and Node Templates involve their containing Expectation, and can be seen as constraints on Expectations.</p>
<p>All nodes that could be <a href="#instantiating-nodes-from-inferences">instantiated from an Inference</a> MUST satisfy any constraints that apply to other nodes of that type. Implementations SHOULD enforce these constraints during Expectation and Node Template construction, but MAY delay enforcement until Inference construction instead.</p>
<h3 id="node-queries">Node Queries</h3>
<table>
<thead>
<tr class="header">
<th align="left">Node Query subtype</th>
<th align="left">Properties (index. name : type)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">Connection Query</td>
<td align="left">0. <code>source</code> : integer<br/>1. <code>key</code> : string predicate <br/>2. <code>of</code> : int<br/>3. <code>value</code> : int</td>
</tr>
<tr class="even">
<td align="left">Derivation Query</td>
<td align="left">0. <code>support</code> : set of ints<br/>1. <code>reason</code> : string predicate</td>
</tr>
<tr class="odd">
<td align="left">Expectation Query</td>
<td align="left"><em>exactly the same an Expectation</em></td>
</tr>
<tr class="even">
<td align="left">Inference Query</td>
<td align="left">0. <code>support</code> : list of ints<br/>1. <code>reason</code> : integer</td>
</tr>
<tr class="odd">
<td align="left">OutRef Query</td>
<td align="left">0. <code>parents</code> : set of (string ↦ int) pairs<br/>1. <code>details</code> : set of (string ↦ datum predicate) pairs</td>
</tr>
<tr class="even">
<td align="left">Property Query</td>
<td align="left">0. <code>source</code> : integer<br/>1. <code>key</code> : string predicate <br/>2. <code>of</code> : int<br/>3. <code>value</code> : datum predicate</td>
</tr>
<tr class="odd">
<td align="left">Subject Query</td>
<td align="left">0. <code>from</code> : integer</td>
</tr>
<tr class="even">
<td align="left">Tag Query</td>
<td align="left">0. <code>source</code> : integer<br/>1. <code>key</code> : string predicate <br/>2. <code>of</code> : set of ints</td>
</tr>
</tbody>
</table>
<p>The Subject Query matches both Subjects and Aggregated Subjects; there is not a separate Aggregated Subject Query.</p>
<p>Each integer value in a Node Query MUST either be <code>-1</code> or satisfy all of the following constraints:</p>
<ul>
<li>be inside a Node Query that is inside a list of Node Queries</li>
<li>be ≥ 0</li>
<li>be &lt; the index of the containing Node Query in its containing list</li>
</ul>
<p>If an int value treated as an index into the containing list does not index a node with a type that the same field in the corresponding Node could reference then the Node Query will never match any Node. Node Queries containing &quot;mistyped&quot; integers of like this probably indicate user error, but are not forbidden by this specification.</p>
<h3 id="node-templates">Node Templates</h3>
<table>
<thead>
<tr class="header">
<th align="left">Node Template subtype</th>
<th align="left">Properties (index. name : type or value)</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">Connection Template</td>
<td align="left">0. <code>source</code> : the integer value <code>-1</code><br/>1. <code>key</code> : string producer <br/>2. <code>of</code> : int<br/>3. <code>value</code> : int</td>
</tr>
<tr class="even">
<td align="left">Property Template</td>
<td align="left">0. <code>source</code> : the integer value <code>-1</code><br/>1. <code>key</code> : string producer <br/>2. <code>of</code> : int<br/>3. <code>value</code> : string producer</td>
</tr>
<tr class="odd">
<td align="left">Subject Template</td>
<td align="left">0. <code>source</code> : the integer value <code>-1</code><br/>1. <code>slug</code> : string producer</td>
</tr>
<tr class="even">
<td align="left">Tag Template</td>
<td align="left">0. <code>source</code> : the integer value <code>-1</code><br/>1. <code>key</code> : string producer <br/>2. <code>of</code> : (set of ints) producer</td>
</tr>
</tbody>
</table>
<p>Each integer value in a Node Template MUST either be <code>-1</code> or satisfy all of the following constraints:</p>
<ul>
<li>be inside a Node Template that is inside the <code>consequent</code> of an Expectation that has <em>n</em> elements in its <code>antecedent</code></li>
<li>be ≥ 0</li>
<li>be &lt; <em>n</em> + the the index of the containing Node Template the containing Expectation's <code>consequent</code></li>
</ul>
<h2 id="definitions-and-constraints">Definitions and Constraints</h2>
<p>A set of nodes is called a <strong>dataset</strong>. This specification only discusses complete datasets, where all node references are to other nodes within the dataset. Communication protocols that communicate via partial datasets are neither forbidden by this specification nor discussed by it.</p>
<h3 id="dependencies-and-cycles">Dependencies and Cycles</h3>
<p>The <strong>dependencies</strong> of a node is defined to be the set of all nodes that the node references unioned with all of those nodes' dependencies, recursively.</p>
<p>It is REQUIRED that all references be acyclic; in other words, no node may be included in its own dependencies.</p>
<h3 id="equality">Equality</h3>
<p>Node and value <strong>equality</strong> is defined as follows:</p>
<ul>
<li><p>Two nodes are equal if and only if both (1) they are the same type of node (or one is an Expectation Query and the other an Expectation), and (2) each field of one node is equal to the corresponding field of the other node.</p></li>
<li><p>Two references are equal if and only if they refer to nodes that are equal.</p></li>
<li><p>Two sets are equal if and only if each element in each set has an equal element in the other set.</p></li>
<li><p>Two lists are equal if and only if both (1) they have the same length and (2) for each valid index for the lists, the elements in both lists at that index are equal.</p></li>
<li><p>Two predicates are equal if they have the same type and are defined with equal values. They are not equal if there exists some inputs for which they give different results. If they are defined differently but give the same result for all inputs, their equality is not specified by this specification.</p></li>
<li><p>Two producers are equal if they have the same type and are defined with equal values. They are not equal if there exists some inputs for which they give non-equal results. If they are defined differently but give equal result for all inputs, their equality is not specified by this specification.</p></li>
<li><p>Two strings are equal if they contain the same characters in the same order.</p></li>
<li><p>Two data are equal if and only if either (1) there is an official notion of eqaulity for the given media type(s) and the blobs are equal under than definition or (2) the media types are equal and the blobs have the same bytes in the same order.</p></li>
</ul>
<h3 id="constituents">Constituents</h3>
<p>The <strong>constituents</strong> of any node that is not an Aggregated Subject node is defined to be the singleton set containing just that Node itself.</p>
<p>The <strong>constituents</strong> of an Aggregated Subject node is defined to be the set containing the Aggregated Subject node itself and the elements of the <em>constituents</em> of each of the nodes referenced in the <code>of</code> field of the node referenced by the Aggregated Subject's <code>parts</code> field.</p>
<h3 id="matching">Matching</h3>
<p>Node and value <strong>matching</strong> is defined as follows:</p>
<ul>
<li><p>A predicate and a value match if all of the following are true:</p>
<ol>
<li>the matching is occurring within the context of matching a list of Node Queries and a list of node references</li>
<li>the value has a type that allows the predicate to be evaluated using the value and the list of node tuples created from the list of node references, as described in the section <a href="#predicates">Predicates</a></li>
<li>evaluating the predicate with the value and the list of tuples yields the value <code>true</code></li>
</ol>
<p>Optionally, if the predicate definition does not utilise its list of tuples parameter (i.e., <em>s</em> is not referenced in the table in section <a href="#predicates">Predicates</a>) then the predicate MAY be said to match if condition 3 alone would be true for any list of tuples supplied.</p></li>
<li><p>The integer value <code>-1</code> matches every node reference.</p></li>
<li><p>A set of integers matches a set of node references if each integer in the set of integers matches some node reference in the set of node references. Note that it is <em>not</em> necessary for every node reference to match some integer.</p></li>
<li><p>A set of string ↦ integer pairs <em>q</em> matches a set of string ↦ node reference pairs <em>s</em> if for each (<em>k1</em> ↦ <em>i</em>) in <em>q</em>, there is a (<em>k2</em> ↦ <em>r</em>) pair in <em>s</em> such that <em>k1</em> equals <em>k2</em> and <em>i</em> matches <em>r</em>.</p></li>
<li><p>A non-negative integer <em>i</em> matches a node reference <em>r</em> if and only if all of the following are true:</p>
<ol>
<li>there is a list of node references that was used to initiate the matching process</li>
<li>there is an element of that list at index <em>i</em></li>
<li>the node referenced by the element at the index <em>i</em> is a constituent of <em>r</em>.</li>
</ol></li>
<li><p>A list of node references matches a list of Node Queries if and only if both (1) they have the same length and (2) for each valid list index for the lists, the elements in the list of node reference at that index matches the element in the list of Node Queries at that index.</p></li>
<li><p>A Subject Query matches a reference to a Subject if the Subject Query's <code>from</code> matches the Subject's <code>source</code>.</p></li>
<li><p>A Subject Query matches a reference to an Aggregated Subject if the Subject Query's <code>from</code> matches the <code>parts</code> or <code>source</code> of any element of the Aggregated Subject's constituents.</p></li>
<li><p>An Expectation Query matches a reference to an Expectation if the two are equal.</p>
<p>No predicates or additional flexibility with regard to Expectation Queries are included in this specification in part to keep the type hierarchy of bounded depth (i.e., to avoid <em>X</em> predicate predicate … predicate types).</p>
<p>It is possible to define semantics-preserving reordering and re-indexing operations on Expectation's fields. This specification does not currently consider same-semantics different-order Expectations and Expecation Queries to match.</p></li>
<li><p>A Node Query other than a Subject Query or Expectation Query matches a node reference if and only if both (1) the referenced node and the Node Query are of the same node type and (2) each field in the Node Query matches the corresponding field in the referenced node.</p></li>
</ul>
<h3 id="instantiating-nodes-from-inferences">Instantiating Nodes from Inferences</h3>
<p>The <strong>instantiated node list</strong> of an Inference is defined to be a list of Nodes having the same length as the <code>consequent</code> of the <code>reason</code> of the Inference such that the Node at index <em>i</em> of the instantiated node list is created from the Node Template at index <em>i</em> of the <code>consequent</code> by doing all of the following (in any order)</p>
<ul>
<li><p>replacing any producer values with the result of applying that function using a list of tuples created by dereferencing the elements of the <code>antecedent</code> of the Inference as the function's argument.</p></li>
<li><p>replacing the <code>source</code> value (which is <code>-1</code> in Node Templates) with a reference to the Inference.</p></li>
<li><p>replacing any non-negative integers with the node reference in the <code>support</code> list at the integer's index if the integer is smaller than the length of the <code>support</code> list; otherwise with a reference to the node at index <em>i</em> − <em>n</em> of the instantiated node list being constructed, where <em>n</em> is the length of the <code>support</code> list.</p></li>
</ul>
<p>A Claim SHOULD NOT reference an Inference in its <code>source</code> field UNLESS it is an element of the instantiated node list of that Inference.</p>
<h1 id="known-string-and-datum-values">Known string and datum values</h1>
<p>The bulk of this section and its subsections will be added at a later time. What is present here is just an initial draft of a few example specifications.</p>
<h2 id="known-tag-keys">Known Tag <code>key</code>s</h2>
<p>To convert a known Tag <code>key</code> in this section to a URI or IRI, prepend it with the URI or IRI of this specification concatenated with &quot;/Tag/key#&quot;</p>
<table>
<thead>
<tr class="header">
<th align="left"><code>key</code></th>
<th align="left">constraints</th>
<th align="left">meaning</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">distinct</td>
<td align="left">≥ 2 elements in <code>of</code><br/>all elements referenced in <code>of</code> have same Node subtype (except may mix Subject and Aggregated Subject)</td>
<td align="left">no two elements of <code>of</code> refers to the same real-world subject or contain equivalent information</td>
</tr>
<tr class="even">
<td align="left">same</td>
<td align="left">≥ 2 elements in <code>of</code><br/>all elements referenced in <code>of</code> have same Node subtype (except may mix Subject and Aggregated Subject)</td>
<td align="left">the elements of <code>of</code> are fully equivalent: the same real-world subject, alternative presentations of the same information, etc.</td>
</tr>
<tr class="odd">
<td align="left">unsupported</td>
<td align="left"><code>of</code> contains a single reference to a Claim, Inference, or Derivation</td>
<td align="left">the <code>source</code> does not make this Claim or the <code>support</code> does not justify this Inference or Derivation</td>
</tr>
<tr class="even">
<td align="left">wrong</td>
<td align="left"><code>of</code> cannot include both a &quot;wrong&quot; Tag and any element of that Tag's <code>of</code><br/><code>of</code> cannot include a Subject or Aggregated Subject</td>
<td align="left">the indicated node asserts a falsehood</td>
</tr>
</tbody>
</table>
<h2 id="known-property-keys">Known Property <code>key</code>s</h2>
<p>To convert a known Property <code>key</code> in this section to a URI or IRI, prepend it with the URI or IRI of this specification concatenated with &quot;/Property/key#&quot;</p>
<table>
<thead>
<tr class="header">
<th align="left"><code>key</code></th>
<th align="left">constraints</th>
<th align="left">meaning</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">type</td>
<td align="left"><code>value</code> is an enumerated type value<br/><code>of</code> is a Subject or Aggregated Subject</td>
<td align="left">what type of entity this subject refers to</td>
</tr>
</tbody>
</table>
<h3 id="known-enumerated-type-values">Known Enumerated Type Values</h3>
<p>To convert a value in this section to a URI or IRI, prepend it with the URI or IRI of this specification concatenated with &quot;/Subject/type/&quot;</p>
<table>
<thead>
<tr class="header">
<th align="left"><code>IRI</code></th>
<th align="left">supertypes</th>
<th align="left">meaning</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">person</td>
<td align="left"></td>
<td align="left">a human or putative human</td>
</tr>
<tr class="even">
<td align="left">event</td>
<td align="left"></td>
<td align="left">a distinct and recognisable (time period, location) pair or set thereof effecting or attempting to effect a common end</td>
</tr>
<tr class="odd">
<td align="left">birth</td>
<td align="left">event</td>
<td align="left">the coming forth of an infant from a womb, sometimes expanded to include related events such as conception, naming, legal registration, etc.</td>
</tr>
<tr class="even">
<td align="left">death</td>
<td align="left">event</td>
<td align="left">the transition from &quot;alive&quot; to &quot;dead&quot;</td>
</tr>
<tr class="odd">
<td align="left">place</td>
<td align="left"></td>
<td align="left">a location or region defined politically, geographically, socially, etc.</td>
</tr>
</tbody>
</table>
<p>The &quot;supertype&quot; column indicates an Expectation that derives supertype from subtype.</p>
<h2 id="known-connection-keys">Known Connection <code>key</code>s</h2>
<p>To convert a known Tag <code>key</code> in this section to a URI or IRI, prepend it with the URI or IRI of this specification concatenated with &quot;/Connection/key#&quot;</p>
<table>
<thead>
<tr class="header">
<th align="left"><code>key</code></th>
<th align="left">constraints</th>
<th align="left">meaning</th>
</tr>
</thead>
<tbody>
<tr class="odd">
<td align="left">update</td>
<td align="left"><code>of</code> and <code>value</code> have same node type</td>
<td align="left">an error or inaccuracy in <code>of</code> has been corrected in <code>value</code></td>
</tr>
</tbody>
</table>
<h1 id="partial-implementation-and-extension">Partial Implementation and Extension</h1>
<p>For various reasons, implementers should expect to interact with other software that has adopted only portions of this specification and that have extended this specification in various ways. This section discusses appropriate ways to interact with partial implementations and extensions.</p>
<h2 id="policies-and-suggestions">Policies and Suggestions</h2>
<p>The set of elements in each node type is fixed, and MUST NOT be extended or reduced.</p>
<p>The set of node types MAY be extended, though it is RECOMMENDED that extensions be handled with custom <code>key</code>s in Properties, Connections, and/or Tags instead. It is RECOMMENDED that software ignore new node types when receiving extended data.</p>
<h3 id="partial-implementations-and-data-reductions">Partial implementations and data reductions</h3>
<h4 id="removing-unwanted-or-private-information">Removing unwanted or private information</h4>
<p>Software MAY chose to ignore any node or set of nodes when receiving a dataset provided that all nodes that depend upon the ignored node(s) are also ignored.</p>
<p>Software MAY chose to omit any node or set of nodes while sending a dataset provided that all nodes that depend upon the omitted node(s) are also omitted.</p>
<h4 id="removing-expectations-and-inferences">Removing Expectations and Inferences</h4>
<p>Software sending/receiving a dataset MAY chose to omit/ignore all Inferences and Expectations by converting each Inference into a Derivation with the same <code>support</code> as the Inference and a textual representation of the Expectation as the Derivation's <code>reason</code>. If the software is unable to convert the Expectation to text, it SHOULD use the text &quot;(reason omitted)&quot; as the Derivation's <code>reason</code>.</p>
<h4 id="removing-all-reasoning">Removing all reasoning</h4>
<p>Software sending/receiving a dataset MAY chose to omit/ignore all reasoning, reducing the dataset to a &quot;belief snapshot&quot;. Belief snapshots are datasets that satisfy the following constraints:</p>
<ul>
<li>Only five node types are used: Subject, Property, Connection, Derivation, and OutRef</li>
<li>Each Claim's <code>source</code> references a Derivation</li>
<li>Each element of each Derivation's <code>support</code> references an OutRef</li>
</ul>
<p>General datasets may be reduced to a belief snapshot via the following process:</p>
<ol>
<li>Create a Subject for each group of Subjects and Aggregated Subjects linked by &quot;same&quot; Tags. The new Subject's <code>source</code> should be a Derivation whose <code>support</code> is the set of all OutRef nodes in any of the original nodes' dependencies.</li>
<li>Copy all Properties and Connections that pointed to any of the the original nodes to point to the new node. The new Properties' and Connections' <code>source</code>s should be Derivations whose <code>support</code>s are the sets of all OutRef nodes in any of the original nodes' dependencies.</li>
<li>Replacing any set of Properties or Connections that differ only in <code>source</code> with a single node having as its <code>source</code> a Derivation with the union of the original claims' <code>source</code>s' <code>support</code> as the new Derivation's <code>support</code>.</li>
</ol>
<h3 id="handling-unsupported-values">Handling Unsupported Values</h3>
<h4 id="optional-predicates-and-producers">Optional predicates and producers</h4>
<p>When software that supports Expectation nodes receives a dataset including Expectations that contain predicates that the receiving software cannot evaluate, it is RECOMMENDED that the recieving software keep the Expectation as received but treat the extra predicate as being equivalent to <em>Top</em> when error-checking received data and as equivalent to <em>Not</em>(<em>Top</em>) when deciding if the user can create new Inferences based on the Expectation.</p>
<p>When software that supports Expectation nodes receives a dataset including Expectations that contain producers that the receiving software cannot evaluate, it is RECOMMENDED that the receiving software keep the Expectation as received but treat them as unchecked (like a Derivation).</p>
<h4 id="constrained-vocabularies">Constrained vocabularies</h4>
<p>If software desires a particular set of keys in its data, it is RECOMMENDED that a set of term-normalising Expectations be used to automatically create new Inferences and normalised terms upon receipt of new datasets, keeping the pre-normalised terms as <code>support</code> for the Inferences.</p>
<p>For software not supporting Expectations, it is RECOMMENDED that Derivations be used, resulting in a dataset equivalent to creating the Expectations and Inferences and then applying the process outlined in <a href="#removing-expectations-and-inferences">Removing Expectations and Inferences</a>. For software not supporting any reasoning, it is RECOMMENDED that the resulting dataset be equivalent to creating the Expectations and Inferences and then applying the process outlined in <a href="#removing-all-reasoning">Removing all reasoning</a>.</p>
<p>If the software does not know how to normalise a particular term, or does but does not care to represent the resulting normalised term, it MAY omit the term and its containing node, as outlined in <a href="#removing-unwanted-or-private-information">Removing unwanted or private information</a>.</p>
<h4 id="constrained-representations-and-topics">Constrained representations and topics</h4>
<p>If software desires a particular structure in its data, for example desiring birthdate Properties instead of birth Subjects, it is RECOMMENDED that a set of Expectations be used to automatically create new Inferences and normalised structures upon receipt of new datasets.</p>
<p>In some cases, software may decide to discard additional nodes; for example, software that tracks human relationships might chose to omit not only a &quot;species&quot;:&quot;horse&quot; Property but also the Subject to which it is attached and all other claims <code>of</code> that Subject.</p>
<h1 id="discussion">Discussion</h1>
<h2 id="node-identity-and-redundancy">Node Identity and Redundancy</h2>
<p>Node identity is determined only <a href="#equality">equality</a>. There is no notion of durable URI, UUID, GUID, DOI, PURL, ARK, or any other form of unique, durable identifier for a node in this specification, nor should one be introduced unless that identifier can be uniquely determined from the contents of the node alone, as for example a hash-based UUID. Equality does require access to the dependencies of a node, but because dependencies are acyclic that is always a finite set.</p>
<p>Because node contents determine node identity, there is no intrinsic notion of versions of a node, nor of updating or editing a node's contents. The concept that one node is an update of another should be expressed using a Connection with <code>key</code> &quot;update&quot;.</p>
<p>Because node identity is determined only by content, omitting nodes does not impede collaboration; the non-omitted nodes can still be linked to new research and shared between clients.</p>
<p>Some of the practices for removing information outlined in <a href="#partial-implementations-and-data-reductions">Partial implementations and data reductions</a> can &quot;replace&quot; nodes or introduce new nodes containing information that is redundant with previous data. Replacement doesn't actually modify or destroy any existing nodes, instead creating new, similar nodes. When the recipient sends these new nodes back to the sender the sender will now have nodes expressing redundant information. Retaining these redundant nodes in the dataset does not impede research provided that the user interface prevents redundant information from distracting the user, as (for example) displaying only one copy of a set of Properties that differ only in <code>source</code>. New nodes containing no new information could also be omitted upon receipt, assuming that the recipient can identify those nodes as redundant.</p>
<h2 id="term-normalisation">Term Normalisation</h2>
<p>When extracting the contents of an external source as a set of Claim nodes citing the OutRef of the external source, there is a quality-of-use expectation that the <code>slug</code> of each Subject be a description (in any language) of some specific part of the source that references the Subject in question; and that the various Properties and Connections reflect, as closely as possible, the language of the source (e.g., using a Connection with key <code>Papa</code> if that is the wording used to describe the relationship in the source).</p>
<p>Term normalisation should then happen by introducing new, normalised-term properties and connections <code>source</code>d to Inference or Derivation nodes that use the non-normalised terms as (part of) their <code>support</code>. In many cases, the <code>support</code> should also include the contextual information used in normalisation such as the date, location, and/or language of the underlying document.</p>
<h2 id="inferences-vs.-derivations-and-the-creation-of-expectations">Inferences vs. Derivations, and the creation of Expectations</h2>
<p>Inferences and Derivations are both intended to fulfil the same objective: to represent the fact that some claims were constructed using reasoning supported by other pieces of information within the dataset. Either Source type can be used to model any reasoning that is based on a finite set of other nodes. They differ primarily in that Inferences are more complicated to implement but are more language-independent and machine-understandable than Derivations.</p>
<p>The Derivation node does not require the use of Expectations and their associated Node Queries, Node Templates, matches, predicates, and producers. However, they leave the <code>reason</code> in machine-opaque human-language text, meaning that they are not readily analysed by the computer, are language-dependent, may lack necessary detail to communicate the reasoning process, may contain text not in keeping with their support, etc.</p>
<p>The Inference node is a machine-understandable representation of reasoning. However, Inferences require the creation of Expectation nodes, with the corresponding complexities of Node Queries, Node Templates, matches, predicates, and producers.</p>
<p>It is anticipated that some users of tools supporting Inference nodes will not be comfortable creating their own Expectations manually and will not be willing to limit themselves to a set of pre-built Expectations. The following set of steps is provided as a suggested way of assisting users in creating nontrivial Expectation nodes without needing to understand their underlying structure.</p>
<ol>
<li><p>Have the user specify the nodes they intend to infer and select the nodes they recognise as the support for their inference.</p>
<p>(This step would also be present in creating Derivations, and would be followed by asking the user to type a free-text explanation.)</p></li>
<li><p>Build a set of candidate antecedents from the nodes identified as support, augmented with any nodes referenced by the nodes identified as being inferred that are not already in the antecedent or inferred collections.</p></li>
<li><p>For each field of each candidate antecedent (except the fields of an OutRef; see below), ask the user &quot;is this field important to your reasoning?&quot; If the answer is &quot;no&quot;, replace the field with <code>-1</code> (if a node reference) or predicate <em>Top</em> (otherwise). If the answer is &quot;yes&quot;, add the referenced node to the candidate antecedent set (if a node reference) or replace it with a predicate <em>Lit</em> (otherwise). Continue until you have asked about all nodes of all fields in the antecedent set.</p>
<p>For the <code>details</code> field of an OutRef, instead ask this question of each pair in the set and, if the answer is &quot;no&quot;, remove the pair completely to create a predicate <em>MapHas</em>.</p>
<p>For the <code>parents</code> field of an OutRef, instead ask this question of each pair in the set and, if the answer is &quot;no&quot;, remove the pair completely; if it the answer is &quot;yes&quot;, add the referenced OutRef to the candidate antecedent set.</p>
<p>You could optionally ask additional questions to build more advanced predicates; for example, to create a <em>Same</em> predicate you could ask something like &quot;we notice that these two nodes both have the same <code>value</code>; is that same-value characteristic important?&quot;</p></li>
<li><p>Order the candidate antecedents such that, if node <em>A</em> is in node <em>B</em>'s dependencies and both are antecedents then <em>A</em> appears before <em>B</em>. Use this ordering to populate the <code>antecedent</code> list and to replace node references with integers.</p>
<p>There may be many orderings that satisfy this requirement. Future versions of this specification might recommend a particular canonical ordering in order to reduce variability in Expection nodes.</p></li>
<li><p>Organise the inferred nodes such that, if node <em>A</em> is in node <em>B</em>'s dependencies and both are being inferred then <em>A</em> appears before <em>B</em>. Use this ordering to populate the <code>consequent</code> list and to replace node references with integers. Additionally, replace all inferred node <code>source</code> values with <code>-1</code>, as required by the definition of Node Template.</p>
<p>There may be many orderings that satisfy this requirement. Future versions of this specification might recommend a particular canonical ordering in order to reduce variability in Expection nodes.</p></li>
</ol>
