Example 1: ad-hoc notation, working with a single source document
-----------------------------------------------------------------

This is a hasty example utilizing ad-hoc notation, including fairly arbitrary mixing of strings from various ontologies and strings of my own creation, displaying (type,language,value) triples as strings, and so on.  Its goal is simply to present one possible use for this model.

As suggested by Rob Hoare, the basis for this example is [https://familysearch.org/ark:/61903/1:1:MS79-1S8](https://familysearch.org/ark:/61903/1:1:MS79-1S8)

----

First, I'd make an OutRef for the source.  Let's make two, so we can see what the parents field is for, one for the 1900 census and one for the particular part of it in question:

    O1. OutRef(parents={},
          details={"type":"Census","country":"United States","year":"1900",...})
    O2. OutRef(parents={O2},
          details={"district":"1006","sheet":"9A",...}

Note that I expect this model to be replaced by the S&CEG's work, but I needed a filler to make a self-contained example specification.

Next, I'd document what that one source claims.  Part of that would be

    S1. Subject(slug="person of line 14")
    P1. Property(source=O2, key="family name", of=S1, value="Smith")
    P2. Property(source=O2, key="given name", of=S1, value="Magdoline")
    P3. Property(source=O2, key="DATE OF BIRTH", of=S1, value="Aug 1882")
    P4. Property(source=O2, key="Age at last birthday", of=S1, value="17")
    P5. Property(source=O2, key="Place of birth", of=S1, value="Illinois")

    S2. Subject(source=O2, slug="person of line 10")
    P6. Property(source=O2, key="family name", of=S2, value="Smith")
    P7. Property(source=O2, key="given name", of=S2, value="William")

    S3. Subject(source=O2, slug="household of lines 10-17")

    C1. Connection(source=O2, key="Head", of=S3, value=S2)
    C2. Connection(source=O2, key="member", of=S3, value=S1)
    C3. Connection(source=O2, key="Daughter", of=S2, value=S1)

    P8. Property(source=O2, key="enumerated date", of=O2, value="+1900-06-07")

So far this is essentially just RDF-like triples with identifiers and sources.  Now for some reasoning.  Let's start simple, just a term normalisation:

    E1. Expectation(antecedent=[
            SubjectQuery(from=-1),
            PropertyQuery(source=-1, key=Regex("/place of birth/i", of=0, value=Top)
        ], consequent=[
            PropertyTemplate(key="http://schema.org/birthPlace", of=0, value=Lookup(1,3))
        ])

A rough English translation of that expectation is

>   Given something with a "place of birth" property, you can infer a `http://schema.org/birthPlace` property with the same value.

That expressed the idea that any "place of birth" key, such as this census uses for its column header, can be converted into a particular standardised term.  I'd apply it as

    I1. Inference(support=[S1,P5], reason=E1)
    P9. Property(source=I1, key="http://schema.org/birthPlace", of=S1, value="Illinois")

I could also infer a birth date from the age; since my current list of producers does not include arithmetic, I'd either use a Script predicate or a Derivation such as

    D1. Derivation(antecedents={P4,P8}, reason="Birth date can be derived from age")
    S4. Subject(source=D1, slug="inferred birth")
    C4. Connection(source=D1, key="child", of=S4, value=S1)
    P10. Property(source=D1, key="type", of=S4, value="http://gedcomx.org/Birth")
    P11. Property(source=D1, key="date", of=S4, value="A+1882-06-07/1983-06-07")

Note here I've chosen to add a birth event instead of a birthdate; I could normalise that too:

    E2. Expectation(antecedent=[
            SubjectQuery(from=-1),
            PropertyQuery(source=-1, key=Lit("type"), of=0, value=Lit("http://gedcomx.org/Birth")),
            PropertyQuery(source=-1, key=Lit("date"), of=0, value=Top),
            SubjectQuery(from=-1),
            ConnectionQuery(source=-1, key=lit("child"), of=0, value=3)
        ], consequent=[
            PropertyTemplate(source=-1, key="birthdate", of=3, value=Lookup(2,3))
        ])
    I2. Inference(support=[S4,P10,P11,S1,C4], reason=E2)
    P12. Property(source=I2, key="birthdate", of=S1, value="A+1882-06-07/1883-06-07")

A rough English translation of expectation E2 is

>   Given a birth event with a date property and an entity participating in the birth as a child, you can infer the birthdate of the child to be the date of the event.

P12 and P3 agree, with P3 more precise than P12, but I'd leave them both in the data since they are both claims supported by O2 and let the user interface handle displaying just one of the two.



