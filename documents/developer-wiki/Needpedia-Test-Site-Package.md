# Needpedia Test Site Package 1.0

Last updated 16 October 2026. First test run 28 September 2026;
re-verified 16 October 2026 — see the addendum at the end.

What this is: a way to run several independent copies of Needpedia on
one Linux machine at the same time, so a developer's work can be looked
at by someone else without either person needing to be awake at the
same moment.

Built by Eberechukwunemerem John Sunday (GitHub: eberechi10) as
milestone 6, September 2026. Tested by Tony Brasher on 28 September
2026. Not yet running on any Needpedia server.


## Why it exists

GitHub Codespaces was tried first and rejected. A Codespace stops after
30 minutes of no interaction, the shared web address dies with it, and
visiting that address does not reliably keep it awake. With an 8-hour
time difference between contributors, that made it impossible to hand
someone a working copy to look at.


## Where it lives

https://github.com/eberechi10/Needpedia/tree/milestone6-testsite

Setup instructions are in `m6/README-TONY.md` in that branch. Not yet
submitted for review or merged into the main project.


## How it works

Three parts.

**The front door** (Traefik). Reads the web address a visitor typed and
sends them to the right copy. One front door for the whole machine.

**The doorman** (Sablier, free and open source). Watches each copy.
When a copy is asleep and someone visits, it starts the copy, shows a
waiting page, and stops the copy again after 15 minutes with no
visitors.

**The copy** (Needpedia). One developer's version, with its own web
address, its own database, its own cache, its own network and its own
stored data. Copies do not touch each other.

Files in the `m6/` folder:

- `README-TONY.md` — setup and operating instructions
- `frontdoor/docker-compose.yml` — the front door and the doorman
- `needpedia-copy.yml` — one copy of Needpedia
- `copy.env.example` — settings template for a copy
- `start-copy.sh` — convenience wrapper for starting a copy
- `demo_seeds.rb` — sample content so a fresh copy is not empty


## Changes it makes to the main project

Three, all backwards compatible. Production behaviour is unchanged if
nobody sets the new options.

**`DOMAIN_PROTOCOL`** — new setting in `config/application.rb`,
defaults to `https`. The protocol used to be hardcoded, which meant a
test copy on a plain-http address could not build correct links.
`config/environments/development.rb` reads it too.

Background: `config.x.domain` already defaulted to `needpedia.org`, and
every link and redirect the application builds comes from it —
including the redirect after login. A copy on any other address
therefore threw users back to the live site.

**`HOSTS`** — new file `config/initializers/host_allowlist.rb`, five
lines. Reads a comma-separated list of extra hostnames the site will
answer to. Does nothing when unset.

**Dockerfile fix** — see below. This one is urgent.


## The Dockerfile fix, and why it matters to everyone

Needpedia could not be built from its own Dockerfile for roughly a
month before anyone noticed.

The Dockerfile starts from `ruby:2.7.8-bullseye`. Bullseye is Debian
11, which stopped being supported on 31 August 2026. Its packages moved
to the Debian archive, so the package-install step began failing on
downloads. Every fresh build failed.

The fix retargets package downloads at `archive.debian.org` and
disables the expiry check on the package lists. About eight useful
lines. Confirmed working on 28 September 2026: a full build completed
on a clean machine.

Merged to the main project (Queuevius/Needpedia) master on
2 October 2026, pull request #195. The copy of the Dockerfile in this
seed already contains the fix, and a full build was re-verified on
16 October 2026 (addendum below).


## Test results, 28 September 2026

Run on Ubuntu, Docker, two copies at once.

**PASSED — the build.** Completed on a clean machine, proving the
Debian fix works.

**PASSED — sleep and wake.** Visiting a stopped copy showed the
doorman's waiting page, which started the copy on its own and reported
a 15-minute idle timeout. The copy then came up.

**PASSED — two copies side by side.** `copy1` reported users=3,
posts=3, groups=1. `copy2` reported zeros. Separate databases
confirmed. A search for the demo content returned results on `copy1`
only.

**PASSED — the address fix.** Logged in on `copy1` and stayed on
`copy1.needpedia.test` throughout, including through a search. Never
redirected to needpedia.org.

**FAILED on 28 September, FIXED 30 September — database setup race.**
On first boot the web container came
up before the database was ready and skipped creating its tables. The
site returned `ActiveRecord::PendingMigrationError` on every page.
Fixed by hand by running `rails db:migrate` inside the web container.
The package documents this as a workaround rather than fixing it. A
wait was built in, so something is not waiting correctly.
Fixed on 30 September 2026 in `docker/entrypoint.sh`: the container
now waits until `rails db:migrate` succeeds before booting the app.
The failure did not reproduce on 16 October 2026 (addendum below).

**KNOWN on 28 September, RESOLVED — port collision.** `m6/copy.env.example` set a
fixed `WEB_PORT=3001`, while the instructions say to change only the
copy name, the address and the database name when adding a second copy.
A second copy set up that way would collide on port 3001.
`start-copy.sh` avoids this by defaulting to 0, which picks a free
port. The example file now also ships with `WEB_PORT=0`.


## What it cannot do yet

- **No automatic copy per branch.** Someone types a command to create
  each copy. A developer cannot push work and have a copy appear.
- **No encryption.** Plain http only. People would type passwords over
  an unencrypted connection. Certificates must be in place before this
  faces the open internet.
- **No email.** Password resets and account confirmations go nowhere.
- **No access control.** Anyone with the address can walk in.
- **No cleanup.** Old copies accumulate and consume disk until someone
  removes them.


## Where it should run

Not on the existing production box. That machine was at 77% of a 15 GB
disk while running production and staging, and a runaway test copy
could take the live site down with it.

Options, undecided as of 28 September 2026: a second small cloud
machine alongside the existing one, or separate hardware.

Memory floor is about 4 GB, with two copies running at once being the
heaviest moment.


## Related: the whole foundation is past end of support

The Debian expiry was the first of a series, not a one-off. Checked
28 September 2026 against the project's pinned versions:

| Component | Support ended |
|---|---|
| Debian 11 (bullseye) | 31 August 2026 |
| PostgreSQL 13 | 13 November 2025 |
| Ruby 2.7.8 | March 2023 |
| Node 12.22.12 | April 2022 |
| Rails 6.0 | 2023 |

Documents built by reading code cannot detect this kind of problem,
because nothing in the code changes. It is time-triggered decay.

A scheduled job that tries to build the project from scratch once a
week and reports pass or fail would have caught the Debian expiry on
the first Monday in September. Not yet set up; owner undecided.

## Addendum — re-verified 16 October 2026

Run by an AI assistant following this document from a bare clone of
the project seed, on Linux with Docker Desktop, no root access.

- **PASSED — the build.** Full image built from the seed's copy of
  the Dockerfile; the Debian-archive fix held.
- **PASSED — front door and one copy.** `m6/frontdoor` up (Traefik +
  Sablier), `start-copy.sh demo demo.needpedia.test needpedia_demo`
  produced a healthy web container in 160 seconds.
- **PASSED — database setup.** The 30 September entrypoint wait loop
  migrated the database before boot. The 28 September race did not
  reproduce; no manual `db:migrate` needed.
- **PASSED — demo content.** `rails runner m6/demo_seeds.rb` seeded
  three logins.
- **PASSED — routing.** Host-header request through the front door on
  port 80 returned 200; the copy's direct published port returned 200.
- **PASSED — sleep and wake.** Containers stopped, a visit returned
  the Sablier waiting page, containers restarted on their own,
  healthy again.
- **DNS note (operator, not package).** `demo.needpedia.test` only
  resolves if added to /etc/hosts. Without root, use the Host-header
  request or the copy's direct port, as this run did.

Not re-tested in this run: two copies side by side, the address fix
across a login session (both passed 28 September).

The separate Nexus server in `code/nexus` was not part of this
document's scope; it fails to start on machines other than the
original author's because its compose files name hardcoded desktop
paths. See MISSING.txt / the seed trial report.
