# Implicit Contracts

This document captures **unwritten assumptions** discovered by auditing the frontend codebase.
These are data shapes and behaviors the UI depends on that may not be explicitly documented.

---

## 1. Game Detail Response

The frontend `CatchupAdapter` expects the game detail response to have this structure:

```typescript
type ApiGameResponse = {
  game?: {
    id?: number;
    home_team?: string;
    away_team?: string;
    game_date?: string;
    game_end_time?: string;       // Used for post-game grouping
    final_whistle_time?: string;  // Alternative for game end
    venue?: string;
    home_score?: number;
    away_score?: number;
  };
  team_stats?: TeamStat[];
  player_stats?: PlayerStat[];
  plays?: PlayByPlayEvent[];
  social_posts?: SocialPost[];
};
```

### Key Assumptions

1. **`game.game_date`** - ISO 8601 format expected
2. **`game.home_team` / `game.away_team`** - Full team names (not abbreviations)
3. **`game.game_end_time` or `game.final_whistle_time`** - Used to determine post-game posts
4. **`plays` array** - Expected to be sorted by `play_index`
5. **`social_posts` array** - Expected to be sorted by `posted_at`

---

## 2. Play-by-Play Events

The frontend `PbpEventRow` component uses these fields:

```typescript
interface PbpEvent {
  id: string;
  gameId: string;
  period: number;           // Quarter number (1-4, 5+ for overtime)
  gameClock: string;        // Format: "MM:SS" or "MM:SS.s"
  elapsedSeconds: number;   // Calculated from period + gameClock
  eventType: string;        // play_type from backend
  description: string;
  team?: string;            // Team abbreviation
  playerName?: string;
  homeScore?: number;
  awayScore?: number;
}
```

### Event Type Assumptions

The UI maps these event types to display labels:

| Backend Value | UI Label |
|---------------|----------|
| `shot` | Shot |
| `made_shot` | Made |
| `missed_shot` | Miss |
| `rebound` | Reb |
| `assist` | Ast |
| `turnover` | TO |
| `steal` | Stl |
| `block` | Blk |
| `foul` | Foul |
| `free_throw` | FT |
| `timeout` | Timeout |
| `substitution` | Sub |
| `jump_ball` | Jump |
| `period_start` | Start |
| `period_end` | End |
| `game_end` | Final |
| `highlight` | ⭐ |
| `play` / `event` / empty | (hidden) |

---

## 3. Social Posts

The frontend `XHighlight` component expects:

```typescript
interface TimelinePost {
  id: string;
  gameId: string;
  team: string;              // Team abbreviation
  postUrl: string;           // Full X/Twitter URL
  tweetId: string;           // Extracted from URL
  postedAt: string;          // ISO 8601 timestamp
  hasVideo: boolean;
  mediaType: 'video' | 'image' | 'none';
  mediaTypeRaw?: string;     // Raw value from backend
  videoUrl: string;          // Direct video URL
  imageUrl: string;          // Thumbnail/poster image
  sourceHandle: string;      // X handle (without @)
  tweetText: string;         // Tweet content
}
```

### Media Type Assumptions

1. **`media_type` MUST be one of**: `'video'`, `'image'`, `'none'`
2. If `media_type` is missing, UI infers from `video_url`/`image_url`
3. If `has_video=true` but no `video_url`, UI embeds the tweet via Twitter API

### Post Grouping Logic

The frontend groups posts into phases:
- **Pre-game**: First ~20% of chronologically sorted posts
- **In-game**: Distributed across PBP timeline
- **Post-game**: Posts after `game_end_time` or last ~10%

---

## 4. Player Stats

The `FinalStats` component parses raw stats with these key lookups:

```typescript
// Minutes field variations
const minutesKeys = ['minutes', 'min', 'minutes_played', 'mins', 'mp'];

// Priority stat ordering
const priority = [
  'minutes', 'min', 'mp',
  'pts', 'points',
  'reb', 'rebounds',
  'ast', 'assists',
  'stl', 'steals',
  'blk', 'blocks',
  'tov', 'to', 'turnovers',
  'fg', 'fga', 'fg_pct',
  'fg3', '3p', 'fg3a', '3pa', 'fg3_pct', '3p_pct',
  'ft', 'fta', 'ft_pct',
  'orb', 'drb', 'trb', 'pf'
];
```

### Filter Logic

- Only players with **≥ 1 minute played** are displayed
- Minutes can be number or "MM:SS" string format

---

## 5. Team Stats

The UI excludes these fields from team stat display:

```typescript
const TEAM_STAT_EXCLUDES = [
  'points', 'pts', 'score',     // Shown separately as final score
  'source',                      // Metadata
  'updated_at',                  // Metadata
  'game_score',                  // Derived metric
  'plus_minus'                   // Excluded
];
```

---

## 6. Period/Quarter Display

The frontend formats periods as:

| Period Value | Display |
|--------------|---------|
| 1 | 1st Quarter |
| 2 | 2nd Quarter |
| 3 | 3rd Quarter |
| 4 | 4th Quarter |
| 5 | Overtime |
| 6+ | Overtime 2, etc. |

---

## 7. Elapsed Time Calculation

The UI calculates elapsed seconds assuming **12-minute NBA quarters**:

```typescript
const calculateElapsedSeconds = (quarter: number, gameClock: string) => {
  const quarterMinutes = 12;
  const completedQuarters = Math.max(0, quarter - 1);
  const completedSeconds = completedQuarters * quarterMinutes * 60;
  
  // Parse "MM:SS" or "MM:SS.s"
  const match = gameClock.match(/^(\d+):(\d+)/);
  if (match) {
    const minutes = parseInt(match[1], 10);
    const seconds = parseInt(match[2], 10);
    const remainingSeconds = minutes * 60 + seconds;
    const elapsedInQuarter = quarterMinutes * 60 - remainingSeconds;
    return completedSeconds + elapsedInQuarter;
  }
  return completedSeconds;
};
```

**Note**: This assumes NBA games. Other sports may need different period lengths.

---

## 8. Missing Endpoints

The UI expects endpoints that may not be implemented:

### `GET /api/pbp/game/{gameId}`

Expected by `PbpApiAdapter`:

```typescript
type ApiPbpResponse = {
  events?: Array<{
    id?: string | number;
    game_id?: string | number;
    period?: number;
    game_clock?: string;
    elapsed_seconds?: number;
    event_type?: string;
    description?: string;
    team?: string;
    team_id?: string;
    player_name?: string;
    player_id?: string | number;
    home_score?: number;
    away_score?: number;
  }>;
};
```

**Status**: UI currently gets PBP from `/api/admin/sports/games/{id}` response

---

## 9. Date/Time Format Expectations

All timestamps should be **ISO 8601** format:
- `2026-01-01T19:30:00Z`
- `2026-01-01T19:30:00-05:00`

The UI uses `Date` constructor directly:
```typescript
new Date(game.game_date)
new Date(post.posted_at)
```

---

## 10. ID Type Handling

The backend uses `int` IDs, but the UI converts everything to `string`:

```typescript
id: String(game.id),
gameId: String(post.game_id || ''),
```

Both string and number IDs should work in responses.

---

## Action Items

These implicit contracts should be:
1. ✅ Added to OpenAPI spec as documentation
2. ⚠️ Enforced via backend validation where possible
3. 🔄 Reviewed when making breaking changes

