Modern Cloze Overlapper for Anki
================================

This is a rewrite of anki-simple-cloze-overlapper__ in modern JS. For an overview of features,
please take a look at a `usage example`__ in that repository.

__ https://github.com/michalrus/anki-simple-cloze-overlapper
__ https://github.com/michalrus/anki-simple-cloze-overlapper/blob/main/screen-recording.gif

In addition to ``anki-simple-cloze-overlapper`` features, support was added for:

- Nested clozes.
- Clozes in MathJax.
- Cloze Generator.

The code was tested on all Anki platforms: Desktop, AnkiDroid, AnkiWeb and AnkiMobile.

AnkiDroid requires the latest `Android System WebView`__ to be installed.

__ https://play.google.com/store/apps/details?id=com.google.android.webview

AnkiWeb lacks the hooks necessary to await the initialisation of MathJax,
which may lead to problems with equation rendering.

How to Use
----------

#. Place the `<_cloze-overlapper.js>`_ into Anki's `collection.media folder`__.

   __ https://docs.ankiweb.net/media.html#manually-adding-media

#. Create a new note type by cloning the built-in Cloze.
#. Add a rendering configuration field named ``Before|After|OnlyContext|RevealAll|InactiveHints``.
#. Put the content of `<front.template.anki>`_ into note's Front Template.
#. Put the content of `<back.template.anki>`_ into note's Back Template.
#. (Optional AnkiDroid compatibility) Add

   .. code:: css

     .android .card:not(.mathjax-rendered) {
         visibility: hidden;
     }

   to note's Styling to hide everything until ``onUpdateHook`` is executed.
   See AnkiDroid's `Advanced formatting`__ for details.

   __ https://github.com/ankidroid/Anki-Android/wiki/Advanced-formatting#hide-content-during-execution-of-onupdatehook

Rendering Configuration
-----------------------

Template's field ``Before|After|OnlyContext|RevealAll|InactiveHints`` controls the rendering
of clozes. Individual parameters are separated by either spaces, commas, pipes or dots.
Omitted rightmost parameters all take default values.

The parameters are as follows:

``Before`` (non-negative integer, defaults to 1)
  The number of clozes before the currently active ones to uncover.

``After`` (non-negative integer, defaults to 0)
  The number of clozes after the currently active ones to uncover.

``OnlyContext`` (Boolean ``true`` or ``false``, defaults to ``false``)
  Show clozes only within the context (before + current + after).
  Set to ``true`` for e.g. long lyrics/poems.

``RevealAll`` (Boolean ``true`` or ``false``, defaults to ``false``)
  Reveal all clozes on the back of the card. By default, only currently active clozes are revealed.
  (Context clozes are revealed even on cards' fronts.)

``InactiveHints`` (Boolean ``true`` or ``false``, defaults to ``false``)
  Use user-provided hints (i.e. ``{{c#::...::user provided hint}}``) for all clozes.
  By default, only the currently active clozes use provided hints, others use ``[...]``.

Context takes nesting of clozes into account: only clozes at the same level of nesting or above
can be considered before of after the current one. In the following example::

  {{c1::outer 1 {{c2::inner {{c3::deep 1}} {{c4::deep 2}} }} }} {{c5::outer 2}}

- ``c1``, ``c2`` and ``c3`` have no clozes before,
- ``c5`` has no clozes after,
- ``c3`` is before ``c4``, and similarly, ``c4`` is after ``c3``,
- ``c5`` is after ``c1``, ``c2`` and ``c4``, but only ``c1`` is before ``c5``.

Recall All Clozes Card
----------------------

If you need an extra card that asks you for all the clozes at once, add another cloze
with ``ask-all`` in its content, e.g. ``{{c99::ask-all}}``.

Styling of Clozes inside MathJax
--------------------------------

CSS ``.cloze`` class doesn't apply inside MathJax. The styling of MathJax clozes is relegated
to TeX macros: ``\AnkiClozeQ`` on the front of the card, and ``\AnkiClozeA`` on the back
of the card.

By default, ``\AnkiClozeA`` is identical to ``\AnkiClozeQ``. The style of ``\AnkiClozeQ`` is taken
from the ``.cloze`` class:

- If ``cloze { color: ... }`` is `parsable as RGB`__,
  ``\AnkiClozeQ`` will have ``\color[RGB]{RRR, GGG, BBB}``.

  __ https://www.w3.org/TR/css-color-4/#serializing-sRGB-values

- If ``cloze { font-style: ... }`` is either oblique or italic,
  ``\AnkiClozeQ`` will have ``\mathit``.

- If ``cloze { font-weight: ... }`` is bold or greater or equal to 700,
  ``\AnkiClozeQ`` will have ``\boldsymbol``.

You can always uncomment the following block in both ``front.template.anki``
and ``back.template.anki``, and redefine ``\AnkiClozeA`` and ``\AnkiClozeA`` as you see fit.

.. code:: html

  <!--
    Uncomment and adjust if MathJax style autodetection doesn't work for you.
    \[
      \renewcommand\AnkiClozeQ[1]{\boldsymbol{\color{blue} #1}}
      \renewcommand\AnkiClozeA[1]{\AnkiClozeQ{#1}}
    \]
  -->

Reloading ``_cloze-overlapper.js``
-----------------------------------

JavaScript modules are not reloaded from disk automatically. In order to reload
``_cloze-overlapper.js``, open DevTools on the Network tab, check “Disable cache”,
and press :kbd:`Ctrl + Shift + R`. It empties the card's page completely, but after navigating to
the next/previous card and back the module is reloaded.
