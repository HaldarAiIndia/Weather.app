# Weather

A weather app built with Jetpack Compose and Material 3, with live forecasts from Open-Meteo. Search for any
city, swipe between the places you follow, and watch the sky, the icon and the charts animate with the
weather. No API key, no account.

## What it shows

- **Clean architecture in one module.** `domain/` holds the models, repository interfaces and use cases and
  knows nothing about Android or HTTP. `data/` implements them over Open-Meteo and `SharedPreferences`.
  `ui/` renders state from view models. `di/AppContainer.kt` wires it all by hand, so there is one place to
  swap a piece.
- **Material You.** `WeatherTheme` takes its colours from the wallpaper on Android 12+ and falls back to a
  sky-blue Material 3 palette below. Every surface, chip and button is a Material 3 component; the sky
  gradient is tinted toward the theme's primary so the palette shows through.
- **Animations that mean something.** The weather icons are drawn on a `Canvas` and driven by one looping
  phase (sun rays turn, rain falls, the bolt flashes). The sky cross-fades between conditions. The hourly
  temperature curve draws itself in with `PathMeasure`. The temperature flips with `AnimatedContent`, the
  daily range bars grow in, sections enter one after another, and the sun arc eases to its position.
- **Live data with an honest offline story.** The last successful forecast per place is cached as JSON; the
  app opens with it, refreshes if it is older than 15 minutes, and shows a banner with a retry when the
  network fails instead of an empty screen. Pull down to refresh.
- **State handling.** A `StateFlow` of a single UI state per screen; a sealed `ForecastUiState` for
  loading, ready and failed-with-cache; failures typed as `ForecastException` so the UI can explain them.
- **Previews.** Every screen has a stateless content composable with `@Preview`s backed by
  `ui/preview/PreviewData.kt`, so you can iterate on the look in the editor without a device.

## Added: nearby location, widgets, more motion

- **My location.** A target button in the app bar (and a "Use my current location" row in Search, and a button on the
  empty screen) asks for location permission, gets a coarse fix from Play services, names it with `Geocoder`, and saves
  it as the single "my location" place (`Place.CURRENT_LOCATION_ID`). The pager glides to it. Failures (permission,
  location switch off, no fix) show a snackbar with a Settings shortcut. See `data/location/`, `LocateAndSaveUseCase`.
- **Widgets.** Two home-screen widgets in `widget/`: 2x2 (temperature, condition, high/low) and 4x2 (plus the next
  hours). They read the same cache as the app, repaint when the app closes, refresh every 30 min, and the refresh
  glyph fetches a new forecast. Tap anywhere else to open the app.
- **Motion.** Count-up temperature, pager pages that scale and fade as they slide, a rippling locate icon while
  searching, press-scale on rows and buttons, a floating hero icon on the empty screen (`ui/components/Motion.kt`).

## Files

| Path | What it is |
| --- | --- |
| `domain/model/` | `Place`, `Forecast`, `WeatherCondition` (WMO code mapping), `UnitSystem`, `ForecastException` |
| `domain/repository/Repositories.kt` | The interfaces the UI depends on |
| `domain/usecase/` | `GetForecastUseCase` (cache-then-network), place and settings use cases |
| `data/remote/` | `HttpClient` over `HttpURLConnection`, `OpenMeteoApi`, the `@Serializable` DTOs |
| `data/local/LocalStore.kt` | Saved places, unit preference and cached forecasts in one prefs file |
| `data/repository/` | Repository implementations, DTO to domain mappers, error translation |
| `ui/home/` | The pager of forecasts: hero, hourly timeline, 7-day list, metrics grid, sun arc |
| `ui/search/` | City search with debounce, saved places with remove |
| `ui/components/WeatherIcon.kt` | The animated Canvas weather glyphs |
| `ui/components/SkyBackground.kt` | The condition-driven animated sky |
| `ui/theme/` | Colour schemes, type scale, shapes, `WeatherTheme` |

## Where to change things

- **Another weather service:** implement `data/remote/WeatherApi.kt` and change one line in
  `AppContainer`. The DTOs and mappers are the only Open-Meteo-specific code.
- **More detail cards:** add a `MetricCard` in `ui/home/MetricsGrid.kt`; the value already exists on
  `CurrentConditions` or is one field away in `OpenMeteoApi.CURRENT_FIELDS` and `CurrentDto`.
- **A new condition look:** `WeatherIcon.kt` composes glyphs from `drawSun`, `drawCloud`, `drawRain` and
  friends; `skyPalette()` in `SkyBackground.kt` picks the gradient.
- **Persistence:** `LocalStore` is the only class that touches `SharedPreferences`. Replace it with Room or
  DataStore behind the same three repositories.

## Running

Press **Run** to build and install. The app needs internet for the first forecast of each place. Open any
screen file and use the editor **Preview** button to render its previews.

Weather data by [Open-Meteo.com](https://open-meteo.com), CC BY 4.0.
