# OpenType Processing Model

This document describes the OpenType processing for MiaoUnicode. It covers
what attachment points and glyph variants need to be created and how
they interact with the lookups.

One of the design aims in this approach to processing is to minimise
the amount of absolute values that appear in the OpenType. The aim is
to keep as much of the absolute positioning in the font design realm
and out of the smart font code as we can. Thus we use attachment points
where we can and appropriate attachment. Otherwise we have the chunking
issue of grouping glyphs into classes based on their width, which is hard
to maintain and inaccurate. Although there are contexts in which we may
have to use such approaches.

## Tone Handling

The core of the processing of Miao revolves around dealing with the positional
tones. These tones correspond to the following positions:

Letter Name | Tone Position
------------|--------------
RIGHT		| Waist
TOP RIGHT	| Shoulder
ABOVE		| Head
BELOW		| Foot
None	    | Base

The first letter of each tone position is what we will use for AP naming
and the like.

There are two general approaches to attaching vowels to the base. The
first is used for shoulder, waist and base. Here, since we want the vowels
to increase the width of the cluster we use cursive attachment. This forces
the vowel to be a base glyph. For head and foot attachment we use normal
mark attachment, which requires the vowel to be a mark. The result of this
conflict is that we need two versions of each vowel, one is the mark and
the other a base. Since they are identical, one can be a simple reference
to the other.

Base position vowels have no tone letter. The base position is the default
position for the vowel as a base character, which is the default glyph
variant for a vowel. So there is no requirement to attach a base vowel
to the base, but we will probably do it anyway.

The basic processing model is that we walk the tone from its position
at the end of the syllable to just before the first vowel. As we do that
we also select which vowel glyph to use. Then we use the combination of
the tone and first vowel to attach the first vowel appropriately and then
we attach subsequent vowels in a chain to the previous vowel using the V
attachment point.

### OpenType Issues

An issue here is that since marks are considered non-spacing, it is possible
for a sequence of vowels in head or foot position to clash with following
syllables, or even the one before. While we can write code in the feax
processor to help with this, there are too many possible sequences to
handle every possible sequence automatically. Instead we will have to
be reactive and address problem sequences as they are identified.

## Handling Aspiration

The aspiration characters may occur between the base and vowels. They are
cursively attached to the base and vowels are attached to them rather than
the base if necessary.

In attaching vowels in the context of aspiration marks, we assume that
an upper aspiration does not interact with base positioned vowels and,
likewise that the lower aspiration mark (U+16F53) does not interact with
shoulder positioned vowels. Waist position vowels, though, intearct with
either. There are two ways of supporting this. One is to have a waist
base attachment point on the aspiration mark (either below or above it),
or we simply attach as if the vowel is in shoulder or base position and
then shift up or down accordingly.

One complexity is that Unicode have said that both aspiration marks may
occur. This causes confusion for the waist position vowels. Which should
they attach to? This becomes even more pertinent if you compare U+16F1A
and U+16F08. Care will need to be taken in those contexts to choose the
right aspiration to attach to. We will also need to decide which of the
two is cursively attached and which attached as a mark. This also means
we will need mark forms of the aspiration mark glyphs.
