<p align="center">
  <img src="docs/apogee-logo.svg" width="128" height="128" alt="Apogee logo">
</p>

<h1 align="center">Apogee</h1>

The [Apogee](https://github.com/venomous-maker) web framework for
[Grav](https://github.com/venomous-maker/grav): one `Application` that wires
every Apogee module from your `.env` and `config/*.json`.

**The modules it wires:**

| Area | Modules |
|---|---|
| Foundations | support, container, config, events |
| HTTP | http, routing, validation, session, view, auth |
| Background work | queue, scheduler, mail |
| Command line | console |
| Data and logs | grav-cache, grav-redis, grav-orbit (the ORM), grav-logger |

**What it adds:**

- service providers, including deferred ones
- the framework's console commands and `make:*` generators
- maintenance mode

```grav
import "std"
import "apogee-framework"
use apogee.*

fn main() {
    let app = Application.configure(".")
        .withProviders([new AppServiceProvider() as ServiceProvider])
        .withRouting(fn(r: Router) { webRoutes(r) }, fn(r: Router) { apiRoutes(r) })
        .withMiddleware(fn(m: MiddlewareSetup) { m.alias("tenant", new IdentifyTenant()) })
        .withExceptions(fn(k: HttpKernel) { k.renderUsing("PaymentError", fn(e: Exception, req: Request) -> Response { return Response.json({"error": "payment"}, 402) }) })
        .withSchedule(fn(s: Schedule) { s.command("reports:send").dailyAt("07:00").onOneServer() })
        .withMigrations([new CreateUsers() as Migration, new CreatePosts()])
        .withSeeders([new DatabaseSeeder() as Seeder])
        .create()
    Process.exit(app.run(Process.args()))     // ./app serve | migrate | queue:work | schedule:run | make:model Post -m …
}
```

## The application

- **Paths:** `basePath`, `path(sub)`, `configPath`, `storagePath`,
  `resourcePath`, `databasePath`, `publicPath`.
- **Environment:** `isLocal()`, `isProduction()`, `environmentIs([...])`,
  `runningInConsole()`, `isDownForMaintenance()`.
- **Services:** `make::<T>()` resolves a service. `http()`, `router()`,
  `console()` and `schedule()` give the kernels.
- **Lifecycle:** `booting`, `booted`, `terminating` and `terminate` hooks.
- **Without a server or a terminal:** `app.handle(request)` sends a request
  through the app, and `app.call("migrate")` with `app.output()` runs a
  command. Both are for tests.

**Configuration** is read in this order:

1. `.env`, then `.env.<APP_ENV>` and `.env.local`
2. `config/*.json` (`config/app.json` becomes `app.*`)
3. the defaults in `defaults.grav`, each taken from an environment variable
   such as `DB_CONNECTION`, `QUEUE_CONNECTION`, `MAIL_MAILER` or `CACHE_STORE`

## Service providers

```grav
class AppServiceProvider extends ServiceProvider {
    fn register() { self.app.container.singleton::<Payments>(fn(c: Container) -> Payments { return new Payments(Config.getString("services.stripe.key")) }) }
    fn boot(events: Events, gate: Gate) {                 // parameters from the container
        events.listen::<OrderPaid>(fn(e: OrderPaid) { … })
        gate.define("admin", fn(u: any, a: Array<any>) -> any { return (u as User).isAdmin })
    }
}
class ReportsProvider extends ServiceProvider {
    fn provides() -> Array<string> { return [nameof(Reports)] }   // deferred: registered when a Reports is first made
    fn register() { … }
}
```

Each framework provider builds its module from the configuration, as a
singleton made on first use:

| Provider | What it sets up |
|---|---|
| Logging | `single`, `daily`, `stderr`, `stack` and `null` channels |
| Events | discovery of listener classes; queued listeners go to the queue |
| Cache | `memory`, `file`, `redis` or `database` |
| Database | the default connection, a pool, the migrator; Orbit models carried by id in queued jobs; validation's `unique`/`exists` through Orbit |
| Queue | `sync`, `memory`, `database` or `redis`; failed jobs in a file or table; batches in the cache |
| Views | the template paths, the shared `appName` and `appUrl`, and `route()` |
| Mail | `log`, `smtp`, `array` and `sendmail` mailers; notifications |
| Session | `file`, `memory` or `redis` |
| Auth | a gate with discovered policies; `web` (session), `api` (token) and `basic` guards over `auth.model` |
| Routing | the router and HTTP kernel (see below) |
| Schedule | the schedule, in the app's time zone |
| Console | the commands below, plus your discovered `ConsoleCommand`s |

**Routing in detail:**

- **Middleware aliases:** `session`, `csrf`, `auth`, `guest`, `can` and
  `throttle`.
- **Groups:** `web` (session and CSRF) and `api` (throttled, under `/api`).
- **Every request:** the maintenance check runs first, including on unknown
  paths.
- **After the response:** after-response jobs run.
- **Errors:** they're reported to the log.

## Commands

| | |
|---|---|
| App | `about [--json]`, `env`, `key:generate [--show]`, `config:show [key]`, `route:list [--method --name --path --json]`, `serve [--host --port --workers]`, `down [--secret --with-secret --retry --refresh --status --message --allow]`, `up`, `inspire` |
| Database | `migrate [--seed]`, `migrate:status`, `migrate:rollback [--step]`, `migrate:reset`, `migrate:refresh [--seed]`, `migrate:fresh [--seed]`, `db:seed [--class]` (each asks first in production unless `--force`) |
| Queue | `queue:work [connection] [--queue=high,default --tries --timeout --sleep --backoff --max-jobs --max-time --memory --rest --threads --stop-when-empty --once]`, `queue:failed`, `queue:retry id… / --all`, `queue:forget`, `queue:flush [--hours]`, `queue:restart`, `queue:clear`, `queue:monitor queues [--max]` |
| Schedule | `schedule:run`, `schedule:work`, `schedule:list`, `schedule:test`, `schedule:interrupt`, `schedule:clear-cache` |
| Other | `cache:clear`, `view:clear`, `mail:test to [--mailer]` |
| Generators | `make:command`, `controller [--resource --api --invokable --model]`, `middleware`, `request`, `resource`, `model [-m -c -f -s --policy -a]`, `job [--unique]`, `event`, `listener [--event --queued]`, `mail [--markdown]`, `notification`, `policy [--model]`, `provider`, `rule`, `migration [--create --table]`, `seeder`, `factory`, `test` |

## Serving

`./app serve` runs the standard library's HTTP server: one event-loop thread
per CPU, with every connection kept alive. Out of the box it takes as many
connections at once as the process may open files for: it raises the soft
descriptor limit to the hard one. An idle connection costs about 1 KB, so
50,000 at once fit in about 60 MB. `config/http.json` tunes it:

```json
{
    "host": "*", "port": 8000,
    "workers": 0,
    "max_connections": 0,
    "idle_timeout_ms": 15000,
    "header_timeout_ms": 20000,
    "max_body_bytes": 10485760,
    "max_requests_per_connection": 0,
    "compress": false
}
```

`workers` is the number of event-loop threads (0: one per CPU). Raise it when
handlers wait on slow services, since a handler runs on its connection's loop.
`max_connections` 0 means "as many as the descriptor limit allows"; more get a
503. For a burst of thousands of new connections at once, the kernel's accept
queue (`net.core.somaxconn`, often 4096) may also want raising.

Build the server with `grav build --release` for production. It drops the
stack-trace bookkeeping and serves about a third more requests a second.

## Maintenance mode

`./app down --secret=letmein --retry=60` sends a 503 to everyone, with a
`Retry-After` header and JSON or a page depending on the request. These still
get through:

- the IPs passed with `--allow`
- `/up`, the health check
- browsers that visited `/letmein`, which get a signed bypass cookie

The state is a file, so every process sharing the storage directory sees it.

## Tests

```
grav test tests/framework.grav
```

The tests run an app from a temporary folder through its configuration,
providers, migrations, seeders, HTTP routes, validation, token auth, a queued
job that sends mail, maintenance mode and the commands.
