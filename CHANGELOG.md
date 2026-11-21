# Changelog

Every release of apogee-framework, newest first. Each is a `vX.Y.Z` tag with a GitHub
release carrying its section; the `grav` line in `grav.toml` says which
compiler versions it builds with.

## Unreleased

Needs grav 1.10.1 or newer.

- Twelve more modules, each wired from the configuration by a provider:
  crypt, storage, lang, broadcast and sockets, pennant, scout, telescope,
  pulse, queue-dashboard, socialite and http-client. The dashboards are open
  locally and gated (`viewTelescope`, `viewPulse`, `viewQueueDashboard`)
  elsewhere.
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
