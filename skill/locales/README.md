# AskNavidrome message catalogs

Speech output of the skill is translated at runtime with Python's standard
library [`gettext`](https://docs.python.org/3/library/gettext.html).  The
message catalog for the locale of each incoming Alexa request is selected by
the `LocalizationInterceptor` in `skill/app.py`, so responses always match the
language the user speaks to their device.

## Layout

    locales/
      asknavidrome.pot                     # template with all English message ids
      de_DE/LC_MESSAGES/asknavidrome.po    # German translations (source)
      de_DE/LC_MESSAGES/asknavidrome.mo    # German catalog (compiled, committed)
      README.md                            # this file

The `locales` directory is resolved relative to `app.py`, so it works both in
the repository checkout and inside the Docker container.

## How strings are translated

User facing strings live in `skill/app.py` and are passed to `build_speech()`
as whole sentences with **named placeholders**:

    text = build_speech(handler_input,
                        "I couldn't find an album called {album} by {artist} in the collection.",
                        album=album.value, artist=artist.value)

Rules for message ids:

- **One full sentence per message id.**  Never split sentences into fragments
  and never concatenate translated pieces — word order differs per language.
- **Named placeholders only** (`{artist}`, `{album}`, ...), never positional
  ones.  Translators may reorder or drop placeholders to fit their language,
  but must keep the placeholder names exactly.
- **No f-strings or string concatenation** for user facing text, format the
  translated template instead.
- Interpolated values (slot values like artist names) are sanitised for SSML
  before they are substituted in, the translated template itself is left
  untouched so language specific punctuation survives.
- Log messages are **not** translated, they stay English on purpose.

## Extracting messages

Message ids are extracted from the source code with
[pybabel](http://babel.pocoo.org/en/latest/cmdline.html) (part of Babel, a dev
dependency).  Run from the repository root:

    pybabel extract --add-comments=TRANSLATOR -k build_speech:2 --project AskNavidrome -o skill/locales/asknavidrome.pot skill/app.py

`-k build_speech:2` teaches the extractor that the message id is the second
argument of `build_speech()`.  Run this whenever messages are added or changed,
then update the existing catalogs:

    pybabel update -i skill/locales/asknavidrome.pot -d skill/locales -D asknavidrome

## Adding a new language

1. Create the catalog (use the Alexa locale code, e.g. `fr_FR`, `es_ES`, `it_IT`):

       pybabel init -i skill/locales/asknavidrome.pot -d skill/locales -D asknavidrome -l fr_FR

2. Translate the empty `msgstr` entries in
   `locales/fr_FR/LC_MESSAGES/asknavidrome.po` (any PO editor such as Poedit
   works).  Keep the `{placeholder}` names intact.  When done, remove the
   `#, fuzzy` marker from the header — fuzzy entries are not compiled.

3. Compile the catalog and commit both the `.po` and the generated `.mo`
   (the runtime only reads `.mo`, committing it keeps the Docker build simple):

       pybabel compile -d skill/locales -D asknavidrome

No code changes are required.  Unknown locales and English (`en-US`)
transparently fall back to the English message ids.

## Alexa side of localization

The backend translation above only covers the skill's *responses*.  To actually
speak to the skill in another language the Alexa skill must also publish that
locale:

- Translate the sample utterances in the interaction model.  The `alexa.json`
  file in the repository root only contains the English model; other locales
  are maintained in the Alexa Developer Console (or exported as additional
  model files).
- Choose an invocation name for the locale.
- Build and publish the locale in the skill's distribution settings.

Built-in intents (`AMAZON.StopIntent`, ...) and slot types (`AMAZON.Artist`, ...)
are localized by Amazon automatically.
