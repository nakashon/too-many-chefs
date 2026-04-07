# Copilot Instructions — Games Repo

## Project Overview
A collection of kitchen-themed card games deployed via GitHub Pages.
- **Repo**: `nakashon/games` (renamed from too-many-chefs, originally too-many-cooks)
- **Local folder**: `/Users/nakashon/too-many-cooks` (not renamed locally)
- **Pages URL**: https://nakashon.github.io/games/
- **Tech**: Single self-contained HTML files with inline CSS + JS. No build tools.

## Games
1. **Chop Chop!** (`chop-chop.html`) — Push-your-luck card game. Clear your orders fastest.
2. **Too Many Chefs** (`index.html`) — Original all-chance dice/card game.

## Architecture
- Each game is one self-contained HTML file (~1400+ lines) with inline `<style>` and `<script>`
- Google Fonts: Fredoka (UI) + Patrick Hand (flavor text)
- Design: glassmorphism navy/charcoal palette with CSS variables
- i18n: `STRINGS.en`/`STRINGS.he` objects, `t(key, vars)` helper, RTL support
- Sound: Web Audio API square-wave oscillators, `sfxMuted` toggle
- No external dependencies, no build step

## Chop Chop! Game State
```javascript
let G = { players: [], currentIndex: 0, phase: 'title', turnNumber: 0,
  luck: { results: [], flipped: [], busted: false },
  stats: { turns: 0, totalServed: 0, totalDumped: 0, totalBusted: 0 } };
```

## Chop Chop! Card Types & Probabilities
- **COOKING** (weight 4, ~33%): Does nothing, returns to hand
- **READY** (weight 3, ~25%): Order served, removed from game
- **DUMP** (weight 2, ~17%): Moves order to next player (zero-sum)
- **BURNT** (weight varies by heat): Cancels entire turn
  - Low 🧊: 1, Med 🔥: 2, High 🌋: 3
- **SWAP** (weight = burnt/2): Everyone shuffles station order counts

## Chop Chop! Turn Flow
1. Cards dealt face-down (pop-in animation), count stays visible
2. Player taps cards one at a time to flip
3. BURNT → shake + fly-back animation → auto-advance
4. SWAP → resolve flipped cards FIRST (ready/dump/cooking), THEN shuffle all stations
5. Player can tap "Next Chef" to end turn voluntarily → resolve animations → auto-advance
6. AI turns: fully automatic, no buttons shown; `turnId` guards prevent stale timeouts

## Key Design Decisions (Learned the Hard Way)
- **Strategy via punishment**: Burnt cancels ALL gains. The loss of accumulated good cards IS the punishment. No extra penalty needed.
- **Push-your-luck works because**: every flip is a genuine risk/reward decision
- **3 failed strategy attempts** in original game: tap-to-flip (no downside), draw-picker (no reason to draw fewer), push-your-luck-bust (penalty too mild). These all failed because there was no meaningful downside.
- **Fixed player positions**: Players stay in fixed spots on screen. Swapping positions per turn was confusing.
- **Single button flow**: Merged STOP + NEXT into one "Next Chef" button. Less clicks = better UX.
- **Auto-advance**: After resolution animations complete, auto-move to next turn. No extra click needed.
- **Scaling orders by player count**: 2p=6, 3p=5, 4p=4, 5p=3. Prevents long 4-5 player games.
- **Cooking cards were too common**: Reduced from 50% to 33%, boosted Ready from 17% to 25%. Games are ~2x faster.
- **Swap commits turn first**: Flipped cards (ready/dump) resolve BEFORE the station shuffle. This makes swap strategically interesting — dump 3 cards then get swapped!

## Mobile
- Both `onclick` AND `ontouchend` with `e.preventDefault()` needed for mobile tapping
- `touch-action: manipulation` and `-webkit-tap-highlight-color: transparent`
- Spotlight row stays horizontal on mobile (not stacked)
- Min touch target: 44x44px

## User Preferences
- Owner speaks Hebrew — game has full EN/HE i18n support
- Hebrew text: punctuation/emojis go at END of phrase (logical order), not beginning
- `translations.md` exists for easy text editing
- Game was renamed from "The Lazy Chef" to "Chop Chop!" after daughter's feedback: clearing orders = being the BEST chef, not lazy
- User likes visible +N/-N float animations on order count changes
- User does NOT want count-to-zero animation during card dealing (misleading)
- AI names are food puns (Gordon Ramen, חומוס כהן, etc.)

## Files
- `chop-chop.html` — Main game (~1500 lines)
- `index.html` — Original "Too Many Chefs" game
- `about.html` — Game story and rules page
- `translations.md` — Editable EN/HE translation table
