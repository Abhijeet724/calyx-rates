# Calyx Rates

Flutter task submission for Calyx Solutions.

## What it does

1. **Login** against `POST /pilot/api/userlogin/checklogin`.
2. **Profile circle** on the dashboard: shows `customer_profileimage` when
   the login response has one, otherwise the first letter of the user's
   name in capitals inside a colored circle.
3. **Default country rates** on the dashboard from
   `POST /pilot/api/rate/dashboardrates` (JWT bearer token from login):
   - uses the login response's `payoutCountryId` when it is set;
   - when `payoutCountryId` is empty, shows the first available country's
     rates.

## Assumptions (not explicitly defined in the task)

- **Empty `payoutCountryId`:** the app shows rates for the first available
  country. The ordering comes from the API's own country list at
  `POST /pilot/api/country/countrylist` (only `clientID` + `branchID`, no
  token; for this tenant 71 countries, Australia first, Nigeria second).
  The app walks that list in order (parallel batches of 5) and uses the
  first country that returns rate records. For the provided test account
  the list's first entry, Australia, returns "No Records Found" for rates,
  so the app shows Nigeria (countryID 6: NGN, USD, INR rates). If the
  country-list call itself fails, a small configured candidate ID list is
  used as a backup.
- **Selected country returns no data:** if `payoutCountryId` is set but its
  rate request returns no records, the app applies the same
  first-available-country fallback. If no country returns records at all,
  the dashboard shows a clear "no rates available" state with a retry
  button instead of failing silently.

## Test credentials

No credentials are stored in the app. The login form starts empty; sign in
with the test account supplied in the Calyx task brief.

## Stack and build

Flutter 3.47.6 (stable), Material 3, `http` only. Structure: `lib/api`
(API client), `lib/models`, `lib/screens`, `lib/widgets`, `lib/config.dart`.
Unit tests in `test/api_test.dart`.

```
flutter pub get
flutter analyze
flutter test
flutter build apk --release
```

CI (`.github/workflows/build-apk.yml`) runs analyze + tests and builds the
release APK (debug-signed, standard for a task build).
