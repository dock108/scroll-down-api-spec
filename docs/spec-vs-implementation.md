# Spec vs Implementation Validation Report

This document compares the OpenAPI specification against the actual backend and frontend implementations.

**Last Updated:** January 1, 2026

---

## Summary

| Category | Status |
|----------|--------|
| Games Endpoints | ✅ Aligned |
| Teams Endpoints | ✅ Aligned |
| Social Posts Endpoints | ✅ Aligned |
| Play-by-Play Endpoints | ⚠️ Deviation (see below) |
| Scraper Endpoints | ✅ Aligned |
| Model Schemas | ✅ Aligned |

---

## Detailed Validation

### ✅ Games Endpoints

#### `GET /api/admin/sports/games`

| Aspect | Spec | Backend | UI Usage | Status |
|--------|------|---------|----------|--------|
| Path | `/api/admin/sports/games` | `/api/admin/sports/games` | `SportsApiAdapter.ts` | ✅ Match |
| Method | GET | GET | - | ✅ Match |
| Query: league | array of strings | `list[str]` | filtered client-side | ✅ Match |
| Query: startDate | date | date | used | ✅ Match |
| Query: endDate | date | date | used | ✅ Match |
| Query: limit | int (50 default) | int (50 default) | 100 | ✅ Compatible |
| Response: games | GameSummary[] | GameSummary[] | mapped | ✅ Match |

#### `GET /api/admin/sports/games/{gameId}`

| Aspect | Spec | Backend | UI Usage | Status |
|--------|------|---------|----------|--------|
| Path | `/api/admin/sports/games/{gameId}` | `/api/admin/sports/games/{game_id}` | `CatchupAdapter.ts` | ✅ Match |
| Response: game | GameMeta | GameMeta | full usage | ✅ Match |
| Response: team_stats | TeamStat[] | TeamStat[] | `FinalStats.tsx` | ✅ Match |
| Response: player_stats | PlayerStat[] | PlayerStat[] | `FinalStats.tsx` | ✅ Match |
| Response: plays | PlayEntry[] | PlayEntry[] | mapped to timeline | ✅ Match |
| Response: social_posts | SocialPostEntry[] | SocialPostEntry[] | mapped to posts | ✅ Match |

---

### ✅ Teams Endpoints

#### `GET /api/admin/sports/teams`

| Aspect | Spec | Backend | Status |
|--------|------|---------|--------|
| Path | `/api/admin/sports/teams` | `/api/admin/sports/teams` | ✅ Match |
| Response: teams | TeamSummary[] | TeamSummary[] | ✅ Match |
| Response: total | int | int | ✅ Match |

#### `GET /api/admin/sports/teams/{teamId}`

| Aspect | Spec | Backend | Status |
|--------|------|---------|--------|
| Path | `/api/admin/sports/teams/{teamId}` | `/api/admin/sports/teams/{team_id}` | ✅ Match |
| Response: recentGames | TeamGameSummary[] | TeamGameSummary[] | ✅ Match |

---

### ✅ Social Posts Endpoints

#### `GET /api/social/posts`

| Aspect | Spec | Backend | UI Usage | Status |
|--------|------|---------|----------|--------|
| Path | `/api/social/posts` | `/api/social/posts` | documented | ✅ Match |
| Query: game_id | int | int | used | ✅ Match |
| Query: team_id | string | string (abbrev) | used | ✅ Match |

#### `GET /api/social/posts/game/{gameId}`

| Aspect | Spec | Backend | UI Usage | Status |
|--------|------|---------|----------|--------|
| Path | `/api/social/posts/game/{gameId}` | `/api/social/posts/game/{game_id}` | `SocialPostApiAdapter.ts` | ✅ Match |
| Response: posts | SocialPostResponse[] | SocialPostResponse[] | mapped | ✅ Match |

---

### ⚠️ Play-by-Play Endpoints

#### `GET /api/pbp/game/{gameId}`

| Aspect | Spec | Backend | UI Usage | Status |
|--------|------|---------|----------|--------|
| Path | `/api/pbp/game/{gameId}` | **NOT IMPLEMENTED** | `PbpApiAdapter.ts` | ⚠️ **DEVIATION** |
| Fallback | - | PBP in `/games/{id}` response | `CatchupAdapter.ts` uses game detail | ✅ Workaround |

**Decision:** 
- 🛠 The UI has a `PbpApiAdapter` that expects `/api/pbp/game/{gameId}` but this endpoint doesn't exist
- ✅ The `CatchupAdapter` correctly gets PBP from the game detail response
- 📝 Spec documents the expected endpoint for future implementation
- 🔄 Backend should implement this endpoint OR UI should remove `PbpApiAdapter`

**Recommendation:** Keep spec as-is. Backend can implement when needed, or spec can be updated to mark as "not implemented".

---

### ✅ Scraper Endpoints

All scraper management endpoints match:

| Endpoint | Spec | Backend | Status |
|----------|------|---------|--------|
| `POST /api/admin/sports/scraper/runs` | ✅ | ✅ | Match |
| `GET /api/admin/sports/scraper/runs` | ✅ | ✅ | Match |
| `GET /api/admin/sports/scraper/runs/{runId}` | ✅ | ✅ | Match |
| `POST /api/admin/sports/scraper/runs/{runId}/cancel` | ✅ | ✅ | Match |

---

## Model Schema Validation

### ✅ GameSummary

| Field | Spec Type | Backend Type | UI Expectation | Status |
|-------|-----------|--------------|----------------|--------|
| id | integer | int | String(id) | ✅ Compatible |
| league_code | string enum | str | filtered | ✅ Match |
| game_date | date-time | datetime | Date() | ✅ Match |
| home_team | string | str | string | ✅ Match |
| away_team | string | str | string | ✅ Match |
| home_score | int nullable | int | number | ✅ Match |
| away_score | int nullable | int | number | ✅ Match |

### ✅ PlayerStat

| Field | Spec Type | Backend Type | UI Expectation | Status |
|-------|-----------|--------------|----------------|--------|
| team | string | str | string | ✅ Match |
| player_name | string | str | string | ✅ Match |
| minutes | number nullable | float | parsed | ✅ Match |
| points | int nullable | int | number | ✅ Match |
| raw_stats | object | dict | Record<string, any> | ✅ Match |

### ✅ SocialPostEntry

| Field | Spec Type | Backend Type | UI Expectation | Status |
|-------|-----------|--------------|----------------|--------|
| id | integer | int | String(id) | ✅ Compatible |
| post_url | uri | str | string (URL) | ✅ Match |
| posted_at | date-time | datetime | ISO string | ✅ Match |
| media_type | enum | str | 'video'/'image'/'none' | ✅ Match |
| video_url | uri nullable | str | string | ✅ Match |
| tweet_text | string nullable | str | string | ✅ Match |

### ✅ PlayEntry

| Field | Spec Type | Backend Type | UI Expectation | Status |
|-------|-----------|--------------|----------------|--------|
| play_index | integer | int | number | ✅ Match |
| quarter | int nullable | int | period | ✅ Match |
| game_clock | string nullable | str | "MM:SS" | ✅ Match |
| play_type | string enum | str | eventType | ✅ Match |
| team_abbreviation | string nullable | str | team | ✅ Match |

---

## Field Naming Conventions

### Backend vs Spec Alignment

The backend uses **snake_case** for all response fields, which the spec preserves:
- `game_date`, `home_team`, `away_team`
- `team_stats`, `player_stats`
- `post_url`, `posted_at`

The frontend adapters correctly transform these to camelCase where needed:
- `game_date` → `date`
- `home_team` → `homeTeam`
- `post_url` → `postUrl`

**Status:** ✅ Consistent

---

## Nullable vs Optional Fields

The spec uses `nullable: true` for fields that can be null in responses:
- `home_score`, `away_score` - null before game completion
- `video_url`, `image_url` - null if no media
- `player_name` - null for non-player events

**Status:** ✅ Correctly specified

---

## Action Items

| Item | Priority | Owner | Status |
|------|----------|-------|--------|
| Implement `/api/pbp/game/{gameId}` endpoint | Low | Backend | 📋 Planned |
| Remove unused `PbpApiAdapter` from UI | Low | Frontend | 📋 Optional |
| Add auth headers to spec when implemented | Medium | Backend | 📋 Future |
| Set up automated spec validation in CI | Low | DevOps | 📋 Future |

---

## Appendix: Files Reviewed

### Backend (sports-data-admin)
- `api/app/routers/sports.py` - Games, Teams, Scraper endpoints
- `api/app/routers/social.py` - Social posts endpoints
- `api/app/db_models.py` - Database models

### Frontend (scroll-down-sports-ui)
- `src/adapters/SportsApiAdapter.ts` - Games API calls
- `src/adapters/CatchupAdapter.ts` - Game detail + timeline
- `src/adapters/SocialPostAdapter.ts` - Social posts
- `src/adapters/PbpAdapter.ts` - PBP (unused)
- `src/adapters/GameAdapter.ts` - Interface definitions
- `src/adapters/PostAdapter.ts` - Post interface
- `src/components/scores/FinalStats.tsx` - Stats display
- `src/components/embeds/XHighlight.tsx` - Post display
- `src/components/timeline/PbpEventRow.tsx` - PBP event display
- `src/pages/GameCatchup.tsx` - Main game page

