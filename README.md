# SeeClear FPL — Team Viewer

A native iOS application for browsing Premier League teams and their FPL players using the Fantasy Premier League API.

## Build & Run

### Requirements
- Xcode with iOS 17 SDK or later
- iOS 17+ Simulator or device
- No third-party dependencies are required

### Steps
1. Open `SeeClearFPL.xcodeproj` in Xcode.
2. Select an iPhone or iPad simulator/device.
3. Select **Product → Clean Build Folder** if required.
4. Build and run with **⌘R**.
5. Run unit tests with **⌘U**.

The app uses the FPL `bootstrap-static` endpoint:

`https://fantasy.premierleague.com/api/bootstrap-static/`

## Assumptions

- The FPL `bootstrap-static` response is the source of truth for teams and player information.
- A team's player count is derived from the players returned by the API for that team.
- Player price is displayed using the FPL `now_cost` value converted to millions.
- Players are grouped by Goalkeepers, Defenders, Midfielders and Forwards.
- Within each position, players are ordered by total FPL points descending, then by player name.
- Search is performed by player display name and updates as the user types.
- If the first network request fails but cached data exists, the cached data is considered usable and is displayed.
- If the app has never successfully loaded data and both network and cache are unavailable, an error state with Retry is shown.

## Architecture

The project uses **MVVM + Repository** with clear separation between presentation and data layers.

```text
SeeClearFPL
├── App
│   └── AppDelegate / SceneDelegate
├── Presentation
│   ├── Teams
│   ├── Squad
│   └── ViewModels
├── Data
│   ├── Models
│   ├── Network
│   ├── Repository
│   └── Cache
├── Core
│   └── UI
└── Resources
```

### Why these choices?

- **MVVM** — keeps view controllers focused on UI and user interaction while view models handle presentation state, filtering and transformation.
- **Repository pattern** — isolates the UI from the source of data and centralizes remote/cache behavior.
- **URLSession** — native networking solution with async/await and no unnecessary third-party dependency.
- **Codable** — provides type-safe decoding of the FPL JSON response.
- **Swift Concurrency** — async/await keeps network operations readable and avoids callback-heavy code.
- **Local JSON cache** — allows the last successful response to be shown when the network is unavailable.
- **UIKit** — follows the exercise requirement and keeps the UI lightweight using reusable programmatic cells/views, with storyboard used for navigation/scene setup.

## Error Handling & Refresh

The app handles the requested states:

- Initial loading
- Successful loading
- Initial failure with Retry
- Pull-to-refresh
- Refresh failure while keeping existing data visible
- Empty results
- Offline launch using previously cached data

Retry is **user initiated** through the Retry action rather than performing unlimited automatic retries.

## Testing

Unit tests cover:

- FPL player JSON decoding
- Player price transformation
- Player search and positional grouping
- Repository fallback to cached data when the network fails

Tests use mocked API/cache implementations, so they do not depend on a live network request.

## Improvements With More Time

- Add UI tests for navigation, search, refresh, retry and empty/error states.
- Add more repository tests covering refresh success/failure and cache-save failures.
- Introduce cache metadata/expiry so stale data can be identified.
- Improve error messages by distinguishing network, server and decoding failures.
- Add accessibility identifiers and further VoiceOver/accessibility validation.
- Add lightweight image/team badge support if required by the product design.
- Add more granular loading states and polished animations for state transitions.

## Known Limitations

- The application depends on the current structure of the FPL `bootstrap-static` API response.
- The cache stores the latest successful dataset; it does not currently implement an explicit expiry policy.
- The app does not perform background data synchronization.
- There is no player-detail screen because it is outside the requested exercise scope.
- Automated tests are unit tests; UI automation tests are not included.
