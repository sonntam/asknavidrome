# AskNavidrome

[![Documentation Generator](https://github.com/rosskouk/asknavidrome/actions/workflows/build_sphinx_docs.yml/badge.svg?branch=main)](https://github.com/rosskouk/asknavidrome/actions/workflows/build_sphinx_docs.yml) [![Docker Container](https://github.com/rosskouk/asknavidrome/actions/workflows/build_image.yml/badge.svg)](https://github.com/rosskouk/asknavidrome/actions/workflows/build_image.yml)


**AskNavidrome** is an Alexa skill which allows you to play music hosted on a SubSonic API compatible media server, like Navidrome.

You can stream your own music collection to your Echo devices without the restrictions you would normally face with regular 
streaming services like Amazon Music or Spotify.  AskNavidrome allows you to:

- Skip backwards and forwards in your current queue or playlist without limitation.
- Avoid paying subscription costs.
- Avoid being forced to listen to adverts at regular intervals.
- Actually use the music collection you have already paid for!
- Run the service on a PC directly or inside a Docker container.

See the full documentation [here](https://rosskouk.github.io/asknavidrome)

## Translations

Speech output is translated per request locale using gettext message catalogs
in `skill/locales/`.  English is the source language, German (`de_DE`) ships as
a complete example.  Adding a language requires no code changes:

    pybabel extract --add-comments=TRANSLATOR -k build_speech:2 --project AskNavidrome -o skill/locales/asknavidrome.pot skill/app.py
    pybabel init -i skill/locales/asknavidrome.pot -d skill/locales -D asknavidrome -l fr_FR
    pybabel compile -d skill/locales -D asknavidrome

See [`skill/locales/README.md`](skill/locales/README.md) for the full workflow
and translator guidelines.  Note that the Alexa interaction model (sample
utterances) must also be translated per locale in the Alexa Developer Console,
the `alexa.json` file only contains the English model.