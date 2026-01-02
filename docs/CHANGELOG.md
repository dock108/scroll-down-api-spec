# Changelog

All notable changes to the API specification will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [1.0.0] - 2026-01-01

### Added

#### Core Game Endpoints
- `GET /api/admin/sports/games` - List games with filtering
- `GET /api/admin/sports/games/{gameId}` - Get full game details
- `POST /api/admin/sports/games/{gameId}/rescrape` - Trigger game rescrape
- `POST /api/admin/sports/games/{gameId}/resync-odds` - Resync odds data

#### Social Posts Endpoints
- `GET /api/social/posts` - List social posts with filters
- `GET /api/social/posts/game/{gameId}` - Get posts for a game
- `POST /api/social/posts` - Create social post
- `POST /api/social/posts/bulk` - Bulk create posts
- `DELETE /api/social/posts/{postId}` - Delete post

#### Teams Endpoints
- `GET /api/admin/sports/teams` - List teams
- `GET /api/admin/sports/teams/{teamId}` - Get team details
- `GET /api/admin/sports/teams/{teamId}/social` - Get team social info

#### Scraper Management Endpoints
- `POST /api/admin/sports/scraper/runs` - Create scrape run
- `GET /api/admin/sports/scraper/runs` - List scrape runs
- `GET /api/admin/sports/scraper/runs/{runId}` - Get run details
- `POST /api/admin/sports/scraper/runs/{runId}/cancel` - Cancel run

#### Play-by-Play Endpoints
- `GET /api/pbp/game/{gameId}` - Get PBP events (planned)

### Core Models
- `Game` - Game summary and details
- `Team` - Team information
- `TeamStat` - Team boxscore stats
- `PlayerStat` - Player boxscore stats
- `SocialPost` - Social media post (X/Twitter)
- `PlayByPlayEvent` - PBP event
- `OddsEntry` - Betting odds
- `ScrapeRun` - Scraper job tracking

### Notes
- Initial extraction from existing sports-data-admin and scroll-down-sports-ui
- Documents current state as of January 2026
- Some endpoints (like `/api/pbp/game/{gameId}`) are expected by UI but not yet implemented



