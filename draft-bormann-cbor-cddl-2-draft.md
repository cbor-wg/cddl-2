---
v: 3

title: >
  CDDL 2.0 — a draft plan
abbrev: CDDL 2.0
docname: draft-bormann-cbor-cddl-2-draft-latest
# date: 2022-10-19

keyword: Internet-Draft
cat: info
stream: IETF
wg: CBOR Working group

venue:
  mail: "cbor@ietf.org"
  github: cbor-wg/cddl-2

pi: [toc, sortrefs, symrefs, compact, comments]

author:
  -
    name: Carsten Bormann
    org: Universität Bremen TZI
    street: Postfach 330440
    city: Bremen
    code: D-28359
    country: Germany
    phone: +49-421-218-63921
    email: cabo@tzi.org


normative:
  RFC8610: cddl
  RFC9165: control1

informative:
  I-D.bormann-cbor-cddl-freezer: freezer
  I-D.bormann-cbor-edn-literals: edn-literals
  I-D.bormann-cbor-update-8610-grammar: grammar
  I-D.bormann-cbor-cddl-more-control: more-controls
  I-D.bormann-cbor-cddl-modules: modules
  I-D.bormann-cbor-rfc-cddl-models: models
  I-D.bormann-cbor-draft-numbers: numbers
  I-D.bormann-cbor-cddl-csv: cddl-csv
  useful:
    target: https://github.com/cbor-wg/cddl/wiki/Useful-CDDL
    title: Useful CDDL
  PSVI:
    target: https://www.w3.org/XML/2002/05/psvi-use-cases
    date: 2002-06-24
    title: Use Cases for XML Schema PSVI API


--- abstract

The Concise Data Definition Language (CDDL) today is defined by
RFC 8610 and RFC 9165.
The latter (as well as some more application specific specifications
such as RFC 9090) have used the extension point provided in RFC 8610,
the control operator.

As CDDL is used in larger projects, feature requirements become known
that cannot be easily mapped into this single extension point.
Hence, there is a need for evolution of the base CDDL specification
itself.

The present document provides a roadmap towards a "CDDL 2.0".
It is based on draft-bormann-cbor-cddl-freezer, but is more selective
in what potential features it takes up and more detailed in their
discussion.
It is intended to serve as a basis for prototypical implementations of
CDDL 2.0.
What specific documents spawn from the present one or whether this
document is evolved into a single CDDL 2.0 specification.

--- middle

Introduction        {#intro}
============

(Please see abstract.)

Note that the existing extension point can be exercised for new
features in parallel to the work described here.
One such draft, {{-more-controls}}, is planned to form the first set of
specifications going forward from the CDDL-2 project together with {{-grammar}}.

The rest of this introduction gives a rough overview over what could
be the development plan for CDDL 1.1, 2.0, 2.5.

## CDDL 1.1 + 2 plan (standards track) {#s11}

* Done before **IETF 117**: CDDL 1.1: {{-grammar}}, *Grammar* fixes:
  Empty files (enabling CDDL 2), non-literal tags, errata fixes (implemented)

* Done before **IETF 117**: Parallel to CDDL 1.1: More *control* operators
  {{-more-controls}}: Additional control operators, another iteration like RFC 9165 (implemented)

* Done before **IETF 118**: CDDL 2.0: {{-modules}} (`import`/`include`
  implemented; potentially further directives to be added)

* Done **2024**: CDDL 2.5: {{anno}} of the present document
  ("*annotations*", plus some functionality enabled by that).
  The requirements are clear, the specific form this takes needs to be
  worked out.
  Enables, e.g., {{Section 5 of -freezer}} (co-occurrence).

## Other documents {#s12}

Not on the main line of development, but important ancillary work:

* (Informational, Mid-2023): {{Section 5 of -freezer}}:
  CDDL-in-JSON format(s) for interchange of CDDL model information
  between tools.
* (Informational, with {{-modules}}): {{-models}}
  (builds standard collection of referenceable models).
* (BCP?): {{-numbers}}
  (BCP for handling assigned numbers during draft stage)

More explorative at this point:

* (Standards-Track?) The remaining {{syntax}} of this document:
  application-oriented literals in CDDL; develop with {{-edn-literals}}.
* (Informational or Standards-Track?): {{-cddl-csv}} (using CDDL to
  model CSV documents).

Mending syntax deficits {#syntax}
======================

The previous content of this section formed the basis for {{-grammar}},
except for {{tagolit-ref}}.

Tag-oriented Literals {#tagolit-ref}
---------------------

Incomplete, see {{tagolit}}.

Processing model: Beyond Validation {#anno}
================

{:compact}
*Proposal Status*:
: experiments with implementations ongoing

*Compatibility*:
: backwards compatible

The basic (implicit) processing model for CDDL 1.0 applies a CDDL data
model to a data item and returns a Boolean that indicates whether the
data item matches that model ("*validation*").

{{Section 4 of RFC9165}} extends this model with named "*features*".
A validation can indicate which features were used.
Validation could also be parameterized with information about what
features are allowed to be used, enabling variants (see {{Section 4 of
RFC9165}} and {{useful}} for examples).

The `cddl` tool ({{Appendix F of RFC8610}}) also supports experimental
forms of "annotating" a validated data item with information about
which rules were used to support validation, currently entirely based on the
information that is in a standard CDDL 1.0 data model.
This leads to a more general concept of "*annotation*", where the data
model specification supports "annotating" the validated instance by
optionally supplying information in the model.
(The annotated result is a special case of a "post-schema validation
instance" [PSVI], here one where the data item itself is only
augmented, not changed, by the process.)

Annotations could in turn provide input to further validation steps,
as is often done with Schematron validation in Relax-NG; with an
appropriate evaluation language this can be used for checking co-occurrence
constraints ({{Section 5 of -freezer}}).

Finally, annotations are a first step to *transformation*, i.e.,
describing how a validated data item should be interpreted as a
transformed data item by performing certain computations.
This generally requires even more support from an evaluation language,
simple transformations such as adding in default values may not need
much support though.

At this time, existing experimental implementations do not lead to a
clear choice for what processing model enhancements should be in
CDDL 2.0.
This document proposes to continue the experimentation and document
good approaches.

Module superstructure
=====================

The previous content of this section formed the basis for {{-modules}}.
Additional work might be started on the ideas outlined in the
subsections of this section.

Cross-universe references
-------------------------

See {{cross}}.
<!-- {Appendix A.2 of -cddl-2-draft}}. -->


ABNF is a lot like CDDL
------------------------

Many of the constructs defined here for CDDL also could be used with
ABNF specifications.  ABNF would definitely benefit from a standard
way to import snippets from existing RFCs.
Since CDDL contains ABNF support ({{Section 3 of RFC9165}}), it would be
natural to make some of the functionality discussed in this section
available for ABNF as well.

IANA Considerations
==================

(Insert new registry for application specific literals here, if adopted.)


Security considerations
=======================

The security considerations of {{-cddl}} apply.

--- back

Fridge
======

This appendix contains sections that may not make it to a 2.0, but
might be part of a followup.


Tag-oriented Literals {#tagolit}
---------------------

{:compact}
*Proposal Status*:
: rough idea, porting from EDN

*Compatibility*:
: backward (not forward)

Some CBOR tags often would be most natural to use in a CDDL spec with a literal
syntax that is tailored to their semantics instead of their
serialization in CBOR.  There is currently no way to add such syntaxes, no
defined extension point either.

The proposal
"Application-Oriented Literals in CBOR Extended Diagnostic Notation"
{{-edn-literals}} defines application-oriented
literals, e.g., of the form

> dt'2019-07-21T19:53Z'

for datetime items.  With additional considerations for unambiguous
syntax, a similar literal form could be included in CDDL.

This proposal opens a name space for the prefix that indicates an
application specific literal.
A registry could be provided to make this name space a genuine
extension point.
(This is currently the production `bsqual` in {{Appendix B of RFC8610}}.)

The syntax provided in {{-edn-literals}} does not
enable the use of CDDL types — it has the same flaw that is being
fixed for tag numbers in {{Section 3.2 of -grammar}}.


Cross-universe references {#cross}
-------------------------

Often, a CDDL specification needs to import from specifications in a
different language or platform.

### IANA references

In many cases, CDDL specifications make use of values that are
specified in IANA registries.  The `.iana` control operator can be
used to reference such a set of values.

The reference needs to be able to point to a draft, the registry of
which has not been established yet, as well as to an established IANA registry.

An example of such a usage might be:

~~~~ CDDL
cose-algorithm = int .iana ["cose", "algorithms", "value"]
~~~~

Unfortunately, the vocabulary employed in IANA registries has not been
designed for machine references.  In this case, the potential values
would come from applying the XPath expression

~~~~ xpath
//iana:registry[@id='algorithms']/iana:record/iana:value
~~~~

to `https://www.iana.org/assignments/cose/cose.xml`, plus some
filtering on the records returned that only leaves actual allocations.
Additional functionality may be needed for filtering with respect to other
columns of the registry record, e.g., `<capabilities>` in the case of
this example.


Acknowledgements
================
{: numbered="no"}

TBD
