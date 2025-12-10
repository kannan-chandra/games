# Games Portal - Project Overview

## Project Description
Static webpage mobile-optimized party games portal hosted on GitHub Pages. No server component - all gameplay runs independently on each player's phone. Games requiring synchronization (Spyfall, Vellaiyappan) use a shared sync word as a random seed to generate pseudorandom assignments that match across all players.

## Architecture

### Tech Stack
- Pure HTML/CSS/JS (no frameworks or dependencies)
- Vanilla JavaScript
- Dark theme, mobile-first design
- System fonts (Apple/Blink)

### File Structure
```
/
├── index.html              # Landing page with game selection
├── charades.html           # Single-player timed word-guessing
├── spyfall.html           # Find the spy (location/role game)
├── vellaiyappan.html      # Find who has different word (Mr. White)
└── word-sets/
    ├── charades-N.txt     # Word lists for charades
    ├── white-N.txt        # Word pairs for vellaiyappan
    └── story-N.txt        # (appears unused currently)
```

## Synchronization System

### How It Works
Players synchronize without a server using deterministic pseudorandom generation:

1. All players enter the **same sync word** (agreed upon in person)
2. Each player selects **total player count** (2-6)
3. Each player picks their **unique player number** (1 to N)

### Key Functions
```javascript
// Simple hash function - converts string to deterministic number
function simpleHash(str) {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    const char = str.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    hash = hash & hash;
  }
  return Math.abs(hash);
}

// Seeded random - generates same "random" number for same seed
function seededRandom(seed, min, max) {
  const hash = simpleHash(seed);
  const normalized = (hash % (max - min + 1)) + min;
  return normalized;
}
```

### Seed Construction Pattern
Seeds combine sync word, round number, and purpose:
- `syncWord + roundNumber + '-spy'` → determines who is spy
- `syncWord + roundNumber + '-location'` → picks location
- `syncWord + roundNumber + '-odd'` → picks odd-one-out player
- `syncWord + roundNumber + '-pair'` → picks word pair
- `syncWord + roundNumber + '-role-' + playerNumber` → assigns role

## Game Details

### 1. Charades (charades.html)
- **Type**: Single-player timer game
- **Gameplay**: 60 seconds to guess as many words as possible
- **Controls**: Correct (score point) / Skip
- **Word Sets**: Auto-discovers `charades-1.txt`, `charades-2.txt`, etc.
- **Format**: First line = display name, remaining lines = words
- **Features**: Shuffles words, reshuffles when exhausted, shows summary with score

### 2. Spyfall (spyfall.html)
- **Type**: Multiplayer synchronized role assignment
- **Gameplay**: One player is spy (doesn't know location), others know location + role
- **Setup Flow**: Sync word → Player count → Player number → Reveal
- **Locations**: 31 hardcoded locations with 6 roles each (in JS array)
- **Special**: Sync word "danger" forces "Yishun" location
- **Assignments**:
  - Spy: Random player number
  - Location: Random from locations array
  - Roles: Seeded by player number

### 3. Vellaiyappan (vellaiyappan.html)
- **Type**: Multiplayer synchronized word assignment
- **Gameplay**: One player gets different word from pair, others get common word
- **Setup Flow**: Choose word set → Sync word → Player count → Player number → Reveal
- **Word Sets**: Auto-discovers `white-1.txt`, `white-2.txt`, etc.
- **Format**: First line = display name, remaining lines = comma-separated pairs (e.g., `Lime,Lemon`)
- **Assignments**:
  - Odd-one-out: Gets word[1] from pair
  - Everyone else: Gets word[0] from pair
  - Pair selection: Seeded by sync word + round

## Word Set Format

### Charades Word Sets (charades-N.txt)
```
Food
Idli
Dosai
Uttappam
```
Line 1 = Display name, rest = individual words

### Vellaiyappan Word Sets (white-N.txt)
```
Classic
Lime,Lemon
Dog,Cat
Coffee,Tea
```
Line 1 = Display name, rest = comma-separated word pairs

## Auto-Discovery System
Both charades and vellaiyappan use async discovery:
```javascript
async function discoverWordSets() {
  const sets = [];
  let i = 1;
  while (true) {
    try {
      const response = await fetch(`word-sets/prefix-${i}.txt`);
      if (response.ok) {
        // Parse first line for display name
        sets.push({ filename, displayName });
        i++;
      } else {
        break; // 404 = no more sets
      }
    } catch {
      break;
    }
  }
  return sets;
}
```

## UI Patterns

### Screen Management
All multi-screen games use:
```html
<section id="screen-name" class="screen active">...</section>
```
```javascript
function goToScreen(screenId) {
  document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
  document.getElementById(screenId).classList.add('active');
}
```

### CSS Variables
```css
--bg: #1e1e1e
--card: #2a2a2a
--text: #f5f5f5
--subtle: #cccccc
--primary: #4f8cff
--secondary: #444
--success: #4fbb6d
--danger: #ff4f4f
```

## Common Buttons
- Primary: Blue rounded pill buttons for main actions
- Secondary: Gray for back/cancel
- Number grid: 3-column grid for selecting numbers
- Word set buttons: Card-style selection with hover effects

## Deployment
Hosted on GitHub Pages - all files are static, no build process required.

## Notes
- No localStorage or state persistence
- Each round is independent
- Players must manually coordinate sync word in person
- Sync word capitalized in round title display
- Round counter increments for multiple rounds with same setup
