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
  RFC5234: abnf
  RFC7405: abnf-case
  RFC9090: oid
  useful:
    target: https://github.com/cbor-wg/cddl/wiki/Useful-CDDL
    title: Useful CDDL
  I-D.draft-bormann-cbor-cddl-freezer: freezer
  cddlc:
    title: CDDL conversion utilities
    target: https://github.com/cabo/cddlc
  Err6526:
    target: "https://www.rfc-editor.org/errata/eid6526"
    title: Errata Report 6526
    seriesinfo:
      RFC: 8610
    date: false
  Err6527:
    target: "https://www.rfc-editor.org/errata/eid6527"
    title: Errata Report 6527
    seriesinfo:
      RFC: 8610
    date: false
  Err6543:
    target: "https://www.rfc-editor.org/errata/eid6543"
    title: Errata Report 6543
    seriesinfo:
      RFC: 8610
    date: false
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

Mending syntax deficits
======================

Non-literal Tag Numbers {#tagnum}
-----------------------

{:compact}
*Proposal Status*:
: complete

*Compatibility*:
: backward (not forward)

The CDDL 1.0 syntax for expressing tags in CDDL is (ABNF as in {{-abnf}}):

~~~ abnf
type2 /= "#" "6" ["." uint] "(" S type S ")"
~~~

This means tag numbers can only be given as literal numbers (uints).
Some specifications operate on ranges of tag numbers, e.g., {{?RFC9277}}
has a range of tag numbers 1668546817 (0x63740101) to 1668612095
(0x6374FFFF) to tag specific content formats.
This can currently not be expressed in CDDL.

CDDL 2.0 extends this to

~~~ abnf
type2 /= "#" "6" ["." tag-number] "(" S type S ")"
tag-number = uint / ("<" type ">")
~~~

So the above range can be expressed in a CDDL fragment such as:

~~~ cddl
ct-tag<content> = #6.<ct-tag-number>(content)
ct-tag-number = 1668546817..1668612095
; or use 0x63740101..0x6374FFFF
~~~

Note that reuses the angle bracket syntax for generics; this reuse is
innocuous as a generic parameter/argument only ever occurs after a
rule name (`id`), while it occurs after `.` here.
(Whether there is potential for human confusion can be debated; the
above example deliberately uses generics as well.)


Tag-oriented Literals
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
{{?I-D.bormann-cbor-edn-literals}} defines application-oriented
literals, e.g., of the form

> dt'2019-07-21T19:53Z'

for datetime items.  With additional considerations for unambiguous
syntax, a similar literal form could be included in CDDL.

This proposal opens a name space for the prefix that indicates an
application specific literal.
A registry could be provided to make this name space a genuine
extension point.
(This is currently the production `bsqual` in {{Appendix B of RFC8610}}.)

The syntax provided in {{?I-D.bormann-cbor-edn-literals}} does not
enable the use of CDDL types — it has the same flaw that is being
fixed for tag numbers in {{tagnum}}.

Clarifications
--------------


{:compact}
*Proposal Status*:
: complete

*Compatibility*:
: errata fix (targets 1.0 and 2.0)

A number of errata reports have been made around some details of text
string and byte string literal syntax: {{Err6527}} and {{Err6543}}.
These need to be addressed by re-examining the details of these
literal syntaxes.
Also, {{Err6526}} needs to be applied (missing backslashes in text
explaining backslash escaping).

### Err6527

The ABNF used in {{RFC8610}} for the content of text string literals
is rather permissive:

~~~ abnf
text = %x22 *SCHAR %x22
SCHAR = %x20-21 / %x23-5B / %x5D-7E / %x80-10FFFD / SESC
SESC = "\" (%x20-7E / %x80-10FFFD)
~~~

This allows almost any non-C0 character to be escaped by a backslash,
but critically misses out on the `\uXXXX` and `\uHHHH\uLLLL` forms
that JSON allows to specify characters in hex.  Both can be solved by
updating the SESC production to:

~~~ abnf
SESC = "\" ( %x22 / "/" / "\" /                 ; \" \/ \\
             %x62 / %x66 / %x6E / %x72 / %x74 / ; \b \f \n \r \t
             (%x75 hexchar) )                   ; \u
hexchar = non-surrogate / (high-surrogate "\" %x75 low-surrogate)
non-surrogate = ((DIGIT / "A"/"B"/"C" / "E"/"F") 3HEXDIG) /
               ("D" %x30-37 2HEXDIG )
high-surrogate = "D" ("8"/"9"/"A"/"B") 2HEXDIG
low-surrogate = "D" ("C"/"D"/"E"/"F") 2HEXDIG
~~~

Now that SESC is more restrictively formulated, this also requires an
update to the BCHAR production used in the ABNF syntax for byte string
literals:

~~~ abnf
bytes = [bsqual] %x27 *BCHAR %x27
BCHAR = %x20-26 / %x28-5B / %x5D-10FFFD / SESC / CRLF
bsqual = "h" / "b64"
~~~

The updated version explicit allows `\'`, which is no longer allowed
in the updated SESC:

~~~ abnf
BCHAR = %x20-26 / %x28-5B / %x5D-10FFFD / SESC / "\'" / CRLF
~~~

### Err6543

The ABNF used in {{RFC8610}} for the content of byte string literals
lumps together byte strings notated as text with byte strings notated
in base16 (hex) or base64 (but see also updated BCHAR production above):

~~~ abnf
bytes = [bsqual] %x27 *BCHAR %x27
BCHAR = %x20-26 / %x28-5B / %x5D-10FFFD / SESC / CRLF
~~~

Errata report 6543 proposes to handle the two cases in separate
productions (where, with an updated SESC, BCHAR obviously needs to be
updated as above):

~~~ abnf
bytes = %x27 *BCHAR %x27
      / bsqual %x27 *QCHAR %x27
BCHAR = %x20-26 / %x28-5B / %x5D-10FFFD / SESC / CRLF
QCHAR = DIGIT / ALPHA / "+" / "/" / "-" / "_" / "=" / WS
~~~~

This potentially causes a subtle change, which is hidden in the WS production:

~~~ abnf
WS = SP / NL
SP = %x20
NL = COMMENT / CRLF
COMMENT = ";" *PCHAR CRLF
PCHAR = %x20-7E / %x80-10FFFD
CRLF = %x0A / %x0D.0A
~~~

This allows any non-C0 character in a comment, so this fragment
becomes possible:

~~~ cddl
foo = h'
   43424F52 ; 'CBOR'
   0A       ; LF, but don't use CR!
'
~~~

The current text is not unambiguously saying whether the three apostrophes
need to be escaped with a `\` or not, as in:

~~~ cddl
foo = h'
   43424F52 ; \'CBOR\'
   0A       ; LF, but don\'t use CR!
'
~~~

... which would be supported by the existing ABNF in {{-cddl}}.

Processing model
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

{:compact}
*Proposal Status*:
: collection of rough ideas with examples

*Compatibility*:
: bidirectional (both backward and forward)

Originally, CDDL was used for small data models that could be
expressed in a few lines.  As the size of data models that need to be
expressed in CDDL has increased, the need to modularize and re-use
components is increasing.

CDDL 1.0 has been designed with a crude form of composition:
Concatenating a number of CDDL snippets creates a valid CDDL data
model unless there is a name collision (identical redefinition is
allowed to facilitate this approach).
With larger models, managing the name space to avoid collisions
becomes more pressing.

The knowledge which CDDL snippets need to be concatenated in order to
obtain the desired data model lives entirely outside the CDDL snippets
in CDDL 1.0.
In CDDL 2.0, rules could be packaged as modules and referenced from other
modules.

There could be some control of namespace pollution, as well as
unambiguous referencing into evolving specifications ("versioning")
and selection of alternatives (as was emulated with snippets in
{{Section 11 of ?RFC8428}}, although an alternative approach for expressing
variants is demonstrated in {{useful}} based on {{Section 4 of RFC9165}}).

Compatibility
-------------

One approach to achieve the module structure that is friendly to
existing environments that operate with CDDL 1.0 snippets and CDDL 1.0
implementations is to add a super-syntax (similar to the way pragmas
are often added to a language), e.g., by carrying them in what is
parsed as comments in CDDL 1.0.

each module to be valid CDDL (if
missing some rule definitions to be imported).

Namespacing
-----------

A convention for mapping CDDL-internal names to external ones could be
developed, possibly steered by some pragma-like constructs.  External
names would likely be URI-based, with some conventions as they are
used in RDF or Curies.  Internal names might look similar to XML
QNames.  Note that the identifier character set for CDDL deliberately
includes $ and @, which could be used in such a convention.

Cross-universe references
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

Potential examples
------------------

This section shows some examples that illustrate potential syntaxes
and semantics to be examined.

One of the potential objectives here is to keep documents that make
use of this extension generally valid as CDDL 1.0 documents, albeit
possibly with a need to add further CDDL 1.0 rules to obtain a
complete specification.

### How name spaces might look like

Implicit namespacing might be provided by using a document reference
as a namespace tag:

~~~
RFC8610.int ⬌ int
RFC9090.oid ⬌ oid
~~~

Note that this example establishes a namespace for the prelude
(`RFC8610.int`); maybe it is worth to do that more explicitly.

### Explicitly interacting with namespaces

New syntax for explicitly interacting with namespaces might be but
into RFC 8610 comments, with a specific prefix (and possibly starting
left-aligned).  Prefixes proposed include `;;<` and `;#`; the below
will use `;#` even though that probably could pose too many conflicts;
it also might be too inconspicuous.


~~~
;# export oid, roid, pen as RFC9090
oid = #6.111(bstr)
roid = #6.110(bstr)
pen = #6.112(bstr)

~~~

Besides an implicit import such as

~~~
; unadorned, just import?
a = [RFC9090.oid]
~~~

there also could be an explicit import syntax:

~~~
;# import oid from RFC9090
a = [oid]
~~~

Such an explicit syntax might also be able to provide additional
parameters such as in the IANA examples above.

### Document references

A convention for establishing RFC references might be easy to
establish, but at least Internet-Draft references and IANA registry
references should also supported.
It is probably worth to add some indirection here, as names of
Internet-Drafts might change (including by becoming RFCs).

~~~
;# include draft-ietf-cbor-time-tag-02.txt as time-tag
event-start = time-tag.etime
~~~


### Add retroactive exporting to RFCs

Existing RFCs with CDDL in them could presume an `export ...all... as RFCnnnn`
(Possibly also per-section exports as in `RFC8610.D` for the prelude?)

Namespace tags for those exports need to be reserved so they cannot be
occupied by explicit exporting.

New specifications (including RFCs) can "include"/"import" from these
namespaces, and maybe "export" their own rules in a more considered way.

### Operations

* "export":
  1. prefix: add a namespace name to "local" rulenames:

     ~~~
     `oid` ➔ `RFC9090.oid`
     ~~~
  2. make that namespace available to other specs

* "import": include (prefixed) definitions from a source
    1. use as is: `RFC9090.oid`
    2. unprefix: `oid`

  Example: prelude processing — include+unprefix from Appendix D of RFC8610.

* "include": find files, turn into namespaces to import from

### To be discussed

How to find the document that exports a namespace
(IANA? Use by other SDOs?  Internal use in an org?
How to transition between these states?)

Multiple documents exporting into one namespace
(*Immutable* RFC9090 namespace vs. "OID"-namespace?
Who manages *mutable* namespaces?)

Updates, revisions, versions, semver:

~~~
;# insert OID ~> 2.2   ; twiddle-wakka: this version or higher
~~~

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

(Insert new registry for application specific literals here.)


Security considerations
=======================

The security considerations of {{-cddl}} apply.

--- back

Acknowledgements
================
{: numbered="no"}

TBD
