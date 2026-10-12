# The Public Records Collection — Game Design Document

Design documentation for **The Public Records Collection**, a Jackbox-style
party pack built in **Godot** with a **Namazu Elements** backend.

The full design lives in [Design Doc.md](./Design%20Doc.md).

## Elevator pitch

A local-multiplayer party pack where one host puts the game on the big screen and
everyone else joins from their phone's browser. Every game is built out of
**public-domain works** — old books, paintings, photographs, films — with a
hand-made, taped-together **scrapbook** look. Five games ship in the collection,
each with its own visual identity, chosen from one shared host menu.

## The five games

| Game | In one line | Status |
|---|---|---|
| **Get Lit** | Cards Against Humanity, but you hunt down a passage from a public-domain book and highlight it to answer the prompt. | Most prepared |
| **Sticker Doodle** | Drawing + guessing, but every drawing must incorporate a required public-domain portrait sticker. | Equally prepared |
| **Present Time!** | Give and receive "gifts": rate a pool of gifts, then shop for each other. First to 30 points. | Gifts listed, no art yet |
| **Average Joe** | Guess how the average person would answer — across yes/no, scales, multi-select, map-pinning, and percentages. | A few questions ready |
| **Cin-A-Mouse** | The mice have taken over a movie studio: remix an old film with SFX, music, your own voice, and subtitles. | Nothing yet; built last |

## Status

Early. Platform + Get Lit are being built first (see the roadmap in the design doc).
Server is a Namazu Elements Element (Java, Elements 3.9); host client is Godot 4.4;
player client is a browser app.
