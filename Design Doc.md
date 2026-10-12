# The Public Records Collection — Design Doc

## 1. The big idea

**The Public Records Collection** is a party pack in the shape of a Jackbox
Party Pack. Each game in the collection is built out of **public-domain
material** — books old enough to be free, museum paintings, historical
photographs, silent films — and presented with a **scrapbook / collage**
aesthetic: things look taped, thumbtacked, and pasted together.

The shape of play is always the same:

- **One host** runs the game on a big screen. They own the game; nobody else
  needs to buy anything.
- **Everyone else joins in a browser** — phone or desktop — with a short
  **room code**, and plays entirely in that browser tab.
- The **host picks a game** from a shared collection menu and can tweak
  **settings** (timers, a family-friendly switch) before starting.

## 2. Product principles

- **Host owns, players just show up.** Only the host installs/runs anything;
  players only need a browser and the room code.
- **Public domain only.** Every asset is either public domain or made by us.
  No licensing surprises, no takedowns.
- **Hand-made look.** The scrapbook/collage style is the collection's unifying
  visual identity; each game re-skins it with its own palette, fonts, and
  imagery.
- **Big-screen readable, small-screen playable.** The host display is designed
  for a TV across the room; the player controller is designed for a thumb.
- **Filterable.** Content is tagged so a family-friendly switch can hide
  anything spicy.

## 3. Architecture

### 3.1 Shape

- **Server = Namazu Elements Element (authoritative).** All game logic, room
  state, scoring, timers, and validation live server-side in a custom Element
  written in Java. The Element is the single source of truth. It exposes:
  - **REST** endpoints for create/join/rejoin and fallback fetches.
  - A **WebSocket** endpoint that pushes state changes to the host and players
    in real time.
- **Host client = native Godot 4.4 app.** Connects with `HTTPRequest` +
  `WebSocketPeer`. Renders the collection menu, settings, lobby, and each game's
  big-screen display.
- **Player client = browser app** (Vite/React), served as static content by the
  Element. Connects with `fetch` + `WebSocket`. Renders the join screen, lobby,
  and each game's controller.
- **Content = JSON packs + optimized assets**, bundled with and served by the
  Element.

### 3.2 Why server-authoritative (not peer-to-peer)

A party game is turn-based and forgiving on latency, but it is not forgiving of
a host's flaky home connection or players dropping off Wi-Fi. Putting authority
in the Element means:

- Players can drop and rejoin without breaking the game.
- No NAT/WebRTC negotiation, no "one person is the server" fragility.
- The host's machine is just a display, not the authority.

The platform's Crossfire is peer-to-peer and aimed at action games; it also has
no Godot or browser client today. It is not used here.

### 3.3 The two-view rule

Every game must define **two views** of its state:

- a **public view** — what the host's big screen shows everyone, and
- a **private view per player** — what one player is allowed to see (their hand,
  their answer, their secret), and nothing more.

The server produces both; the clients never share game state directly.

### 3.4 Reconnection

Rooms persist server-side. A player who reloads or drops rejoins by room code +
their player identity token and resumes their private view. The game does not
pause or reset.

## 4. Shared platform (all games)

Built once, reused by every game.

- **Rooms & codes** — create a room, get a short code, join/leave/rejoin.
- **Lobby** — the host sees who's joined; players mark ready.
- **Settings** — per-game options plus shared ones (timers, family-friendly).
- **Realtime protocol** — one WebSocket carrying `state`, `playerJoined`,
  `playerLeft`, `phase`, `timer`, `results`, and per-game action messages.
- **Content pipeline** — a JSON schema for prompts/assets, a loader, and the
  family-friendly filter.
- **Theme system** — each game declares its palette, fonts, and backgrounds;
  the menu previews them so each game feels distinct before you start.
- **Guest players** — v1 needs no accounts: a player is a name + a room.
  (Host ownership/licensing via store SKUs is deferred.)

## 5. Content filtering

Content is tagged with a single boolean: **`is_spicy`**. The host's
**family-friendly** toggle, when on, filters out spicy prompts, books, and
assets. (Tags currently recorded in source content include `is_spicy` already;
`trigger_warnings` exist for reference but are not a filter dimension yet.)

## 6. The games

### 6.1 Get Lit (Special Edition)

**Pitch.** Cards Against Humanity, but instead of a deck of answer cards you get
a **library of public-domain works**, and you answer by **highlighting a
passage**.

**How it plays.**

1. **Setup.** Each player picks **one work** from the library at the start of
   the game and keeps it for the whole game. (You can replay with a different
   work later.)
2. **Prompt.** Each round, the host shows a prompt — e.g. _"Words in a magic
   spell."_
3. **Answer.** Every player **selects a passage** from their own work that fits
   the prompt. For example, from *Macbeth*:
   _"Double, double toil and trouble; Fire burn and caldron bubble."_
4. **Reveal.** All answers are shown anonymously on the big screen.
5. **Vote.** Players spend **hearts** on their favorites.
   - A normal heart = **1 point**.
   - Each player gets **one golden heart per round** = **3 points**.
   - You may spend as many normal hearts as you like.
6. **Score.** Most points at the end wins.

**Content needed.** A library of public-domain books (text), a prompt list, and
parody titles (flavor). *Ready:* ~88 books, ~256 prompts, ~114 parody titles.

### 6.2 Sticker Doodle

**Pitch.** Pictionary, but every drawing must prominently include a **sticker** —
usually a person cut out of a public-domain painting.

**How it plays.**

1. **One rotating drawer** each round; everyone else guesses.
2. The drawer gets a **prompt** (a celebrity, a movie/TV title, or a
   type-of-person/occupation) and a **required sticker** (a public-domain
   portrait).
3. The drawer must work the sticker into their drawing, then draw around it.
4. Guessers type guesses; **correct guesses score**. The **timer runs faster as
   more people guess correctly**, so the round stays snappy.
5. Scoring rewards both **guessing correctly** and **drawing well** (exact
   "good drawing" scoring to be pinned during build).

**Content needed.** Prompt packs + sticker images. *Ready:* 395 prompts across
celebrity / movie_tv / person_type, and ~100 public-domain sticker images.

### 6.3 Present Time!

**Pitch.** Gifts are judged on vibes, and you're judged on your taste — both
giving and receiving.

**How it plays.**

1. **Store.** The host picks a **Store** (a themed set of gifts).
2. **Draw.** **9 random gifts** are revealed.
3. **Rate.** Every player privately rates each gift **0–5 stars**.
4. **Shop.** Simultaneously and secretly, every player **assigns one gift to
   each other player** (simultaneous secret assignment). Gifts are unique; if two
   players try to give the same gift, the gift's **average star rating** breaks
   the tie.
5. **Score.** Points come from **both giving and receiving**: 1 star = 1 point.
   **First to 30 points wins**, across multiple rounds/stores.

**Content needed.** Gift lists per Store, and **gift images** (to be created).

### 6.4 Average Joe

**Pitch.** Guesspionage-style: answer as close to **the average person** as you
can.

**Answer types.**

- Yes/No.
- A 1–5 scale.
- Multi-select from a list.
- **Pin a point** on a diagram or map.
- A trivia question where you guess **what percentage** of people got it right.

**How it plays.** Players answer; scoring rewards proximity to the real
aggregate. Data comes from **surveys**, seeded with a starter dataset and grown
by a **lobby mini-activity**: while waiting for the game to start, players can
answer survey questions, and those answers are added to the database.

**Content needed.** Survey questions + aggregate data. *Ready:* a few questions.

### 6.5 Cin-A-Mouse

**Pitch.** The mice have taken over an old movie studio; help them make a
masterpiece out of a dusty old film.

**How it plays (draft).** Players remix an existing public-domain film by
choosing **sound effects** and **existing dialog**, picking **music**,
**recording their own dialog**, and **writing subtitles**.

**Content needed.** Public-domain film clips + audio. *Ready:* nothing yet —
designed from scratch, built last.

## 7. Visual identity

The collection shares a **scrapbook/collage** base — taped edges, thumbtacks,
faded paper, pre-1930s palette — and each game layers its **own** identity on
top:

- **Get Lit:** library / reading-room warmth; book plates.
- **Sticker Doodle:** sticky, playful, sticker-sheet energy.
- **Present Time!:** gift-wrap and ribbon; department-store catalog.
- **Average Joe:** survey forms, clipboards, chart paper.
- **Cin-A-Mouse:** film strip, sprocket holes, studio marquee.

## 8. Roadmap

Built in phases, each verified on a temporary preview instance before it is
considered done.

- **Phase 1 — Shared platform + vertical slice.** Rooms/codes, WebSocket, host
  and player shells, settings, content pipeline, themes. A throwaway
  connect-and-ready game proves host + browsers round-trip.
- **Phase 2 — Get Lit.** First full game, end to end.
- **Phase 2b — Sticker Doodle.**
- **Phase 3 — Present Time!** (needs gift art).
- **Phase 4 — Average Joe** (needs survey subsystem + seed data).
- **Phase 5 — Cin-A-Mouse** (media/voice; designed then).
- **Phase 6 — polish, packaging, family-friendly pass, release.**

## 9. Open questions

- Sticker Doodle: exact scoring for "drawing well."
- Present Time!: how many Stores ship at launch; gift art source/style.
- Average Joe: which questions ship as seed data.
- Cin-A-Mouse: full mechanic design.
- Distribution: native host builds for Windows/macOS/Linux; ownership/licensing
  model (SKUs/receipts) deferred past v1.
