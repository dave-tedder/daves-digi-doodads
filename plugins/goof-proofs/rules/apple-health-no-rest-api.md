# Apple Health Has No REST API

Apple HealthKit is on-device only (iOS). There is no server-side API to pull health data from Apple Health programmatically.

## Workaround

The only extraction method is an Apple Shortcut that queries HealthKit and POSTs data to an external endpoint. This works but is fragile (relies on automation triggers, can fail silently, requires the shortcut to run on-device).

## When This Applies

When integrating health/fitness device data into a server-side app. If the device manufacturer (Withings, Oura, Garmin, Whoop, etc.) has their own cloud API with webhooks, use that directly rather than routing through Apple Health as a middleman. Apple Health integration is only worth building for data that doesn't exist anywhere else (Apple Watch-exclusive metrics, iPhone-only step counts, etc.).
