# Changelog

Every release of apogee-framework, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

## 0.4.0 (2026-11-25)

Needs grav 1.12.0 or newer.

- Modules connect themselves: every class extending `ApogeePackage`
  (apogee-package) is found when the app is created, its defaults applied,
  its migrations added to `migrate`, and it is registered and booted;
  `app.packages` lists them.
- The bundled modules are packages now: their settings, wiring, commands and
  middleware moved out of the framework into each module, and their tables
  into migrations (run `migrate`; they were made at boot before).

### Dependencies

- Uses apogee-http 0.3.0, apogee-routing 0.3.0, apogee-validation 0.2.1, apogee-session 0.2.2, apogee-view 0.2.1, apogee-auth 0.2.2, apogee-queue 0.3.1, apogee-console 0.2.2, apogee-scheduler 0.2.3, apogee-mail 0.3.0, apogee-package 0.1.0, apogee-crypt 0.2.0, apogee-storage 0.2.0, apogee-lang 0.2.0, apogee-sockets 0.2.0, apogee-broadcast 0.2.0, apogee-queue-dashboard 0.3.0, apogee-http-client 0.2.0, apogee-pennant 0.2.0, apogee-scout 0.2.0, apogee-telescope 0.2.0, apogee-pulse 0.2.0, apogee-socialite 0.2.0, grav-cache 0.2.2, grav-orbit 0.3.0 (from their GitHub tags).

## 0.3.1 (2026-11-21)

- The newest releases of the modules: Pennant keeps resolved values a moment
  (a flag check costs a lookup, not a query), Telescope's N+1 check and Pulse's
  recording are cheaper.
- Laid out by `grav fmt` and tidied as grav 1.11's lints suggest; no change in
  behaviour.

## 0.3.0 (2026-11-21)

Needs grav 1.10.1 or newer.

- Twelve more modules, each wired from the configuration by a provider:
  crypt, storage, lang, broadcast and sockets, pennant, scout, telescope,
  pulse, queue-dashboard, socialite and http-client. The dashboards are open
  locally and gated (`viewTelescope`, `viewPulse`, `viewQueueDashboard`)
  elsewhere; their routes run with the session and refuse cross-site
  requests that change something (`same-origin`).
- On SQLite, the modules write their tables through Orbit's connection, so
  they join a transaction in progress (a seeder's) instead of waiting on it.
- Commands: `sockets:serve`, `telescope:prune`, `telescope:clear`, `pulse:clear`.
- An app key too short to encrypt with leaves `Crypt` unset instead of
  stopping the app.

## 0.2.2 (2026-11-21)

### Requirements

- Uses apogee-queue v0.3.0, apogee-scheduler v0.2.2 and apogee-mail v0.2.2.

## 0.2.1 (2026-11-20)

### Requirements

- Uses the 0.2.1 releases of apogee-session, apogee-auth, apogee-queue, apogee-console, apogee-scheduler, apogee-mail, grav-cache, grav-redis and grav-logger.

## 0.2.0 (2026-11-20)

### Requirements

- Needs grav 1.9.0 or later, and uses the 0.2.0 releases of the packages it
  depends on.

## 0.1.1 (2026-11-19)

- Uses the newer releases of apogee-queue, apogee-console, apogee-scheduler, apogee-mail, grav-cache, grav-orbit, grav-db (faster row loading and query building in grav-orbit and grav-db).

## 0.1.0 (2026-11-19)

The first tagged release. Needs grav 1.7.0 or newer.

The [Apogee](https://github.com/venomous-maker) web framework for [Grav](https://github.com/venomous-maker/grav): one `Application` that wires every Apogee module from your `.env` and `config/*.json`.
