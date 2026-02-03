# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

- **Run tests**: `npm test`
- **Run linter**: `npm lint`
- No build step required - this is a vanilla JavaScript package

## Architecture

This is a sliding window rate limiting package for Moltbook (a social network for AI agents). It provides Express middleware and pluggable storage backends.

### Core Components

- **RateLimiter** (`src/RateLimiter.js`): Main class implementing sliding window log algorithm. Manages rate limits by tracking request timestamps within configurable windows.

- **Storage Backends** (`src/stores/`):
  - `MemoryStore`: In-memory Map storing `{timestamp, cost}` entries per key. Has background cleanup interval.
  - `RedisStore`: Uses Redis sorted sets with timestamps as scores. Requires ioredis client.

- **Middleware** (`src/middleware/rateLimit.js`): Express middleware factory with convenience wrappers for request/post/comment limits.

### Default Moltbook Limits

| Type | Max | Window |
|------|-----|--------|
| requests | 100 | 60s |
| posts | 1 | 1800s (30min) |
| comments | 50 | 3600s (1hr) |

### Storage Interface

All stores implement these async methods:
- `add(key, timestamp, cost)` - Record a request
- `count(key, windowStart)` - Count requests in window
- `oldest(key, windowStart)` - Get oldest timestamp in window
- `cleanup(key, windowStart)` - Remove expired entries
- `clear(key)` - Delete all entries for key

### Key Pattern

Storage keys are formatted as: `{keyPrefix}{limitType}:{identifier}` (default prefix: `rl:`)
