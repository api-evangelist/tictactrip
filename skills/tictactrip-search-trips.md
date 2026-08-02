---
name: Search multimodal train and bus trips
description: Resolve origin/destination stop clusters and search Tictactrip for multimodal train + bus itineraries between them.
api: openapi/tictactrip-openapi-original.json
operations: [GetAutocomplete, GetAllStopClusters, GetStopCluster, GetResults]
---

# Search multimodal trips

Use this skill to find train/bus itineraries between two European locations.

## Auth
All calls require `Authorization: Bearer <JWT>` (partner token from sales@tictactrip.eu). Search requires the `API_SEARCH_PARTNER` role.

## Steps
1. **Resolve origin and destination.** Call `GetAutocomplete` (`GET /v2/autocomplete?q=<text>`) to turn a user's typed place into stop clusters, or browse `GetAllStopClusters` (`GET /v2/stopClusters`). Capture each side's `gpuid`. Use `GetStopCluster` (`GET /v2/stopClusters/{stopClusterGpuid}`) if you need full detail on one.
2. **Search itineraries.** Call `GetResults` (`POST /v2/results`) with the origin/destination gpuids, date and passengers. The response returns multimodal trips, each made of segments (see `data-model/tictactrip-data-model.yml`) with price and CO2 emissions.
3. **Pick a trip** to carry into the booking flow (see `tictactrip-book-trip.md`).

## Rules
- Errors use `{ errorKey, errorMessage }` JSON (not RFC 9457) — read `errorKey` to branch. `400` = missing/invalid params, `404` = unknown place, `500` = retry.
- There is no idempotency key; search is a safe read.
