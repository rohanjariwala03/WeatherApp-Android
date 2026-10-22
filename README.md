# WeatherApp-Android

An Android weather app written in Java. It pulls a daily forecast for a city from the OpenWeatherMap API, caches it in a Room database, and shows today's weather plus a list of the coming days. I built it in 2021 around the Android architecture components (ViewModel, LiveData, Room) with Dagger for dependency injection.

## What it does

- Today tab: city name, date, weather icon and description, temperature and "feels like" for the current part of the day (morning, day, evening or night), humidity, wind and UV index.
- Weekly tab: one row per day for the configured number of days.
- Share tab: opens the Android share sheet with a line like "Today's weather is light rain with temperature: ..." built from the last forecast shown on the Today tab.
- Settings (from the navigation drawer): change the city and the number of days. The defaults are Paris and 7 days.

The search icon in the toolbar and the About item in the drawer are placeholders with no behaviour behind them.

## How it's put together

- `api/WeatherService` is a Retrofit interface for the `forecast/daily` and `uvi` endpoints. Calls return `LiveData` through a custom call adapter (`util/LiveDataCallAdapterFactory`).
- `repository/ForecastRepository` uses a `NetworkBoundResource` to load from Room, fetch from the network, and save the result back to the database.
- `viewmodel/` holds the ViewModels for the forecast and UV index, created through an injected factory.
- `di/` has the Dagger 2 component and modules (`dagger-android` for injecting into the activity and fragments).
- `ui/` has the Today, Weekly and Settings fragments, wired up with the Navigation component, a bottom navigation bar and a drawer. Views use Data Binding and Butter Knife.

## Stack

- Java 8, Android Gradle Plugin 4.1.2, Gradle 6.5
- compileSdk / targetSdk 30, minSdk 21
- Retrofit 2.3, Room and Lifecycle (the old `android.arch` artifacts), Navigation 2.3
- Dagger 2.16, Butter Knife 10, Glide, Picasso, Timber

Library versions are kept in `versions.gradle`.

## Running it

You need an OpenWeatherMap API key. Put it in `gradle.properties` (or in `~/.gradle/gradle.properties` so it stays out of git), keeping the quotes:

```
WeatherApiKey="your_api_key"
```

It is exposed to the code as `BuildConfig.WEATHER_API_KEY`. Then open the project in Android Studio, or build with:

```
./gradlew assembleDebug
```

## Known issues

- The Java code references four resources that aren't in `res/values` (`format_temperature`, `format_wind_kmh`, `colorPrimaryToday`, `transparent`), so the project won't compile until they're added back.
- `AndroidManifest.xml` doesn't declare the `INTERNET` permission, and `BASE_URL` uses plain `http://`, which Android 9+ blocks by default. Both need fixing before the app can fetch data.
- The `forecast/daily` endpoint is only available on OpenWeatherMap's paid plans, and the standalone `uvi` endpoint has since been retired, so a free key won't return data for either.
