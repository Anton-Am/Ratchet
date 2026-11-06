# Conventions (Ratchet 0.4.x personal packaging fork)

Short reference for how this fork is maintained. Longer version: [`../CONTRIBUTING.md`](../CONTRIBUTING.md).

The package is maintained as a standalone library: no references to the code of a consuming application,
no temporary hooks "for one task". Non-obvious decisions are recorded in `docs/decisions/NNNN-*.md`.

## Branches and tags

| What | Rule |
|---|---|
| Supported branch | `0.4.x` only |
| Integration | feature/fix branches → PR → merge into `0.4.x` |
| Version | git annotated tag `v0.4.N`, no `version` field in `composer.json` |
| Who tags | the maintainer, after the merge into `0.4.x` |
| Package name | `anton-am/ratchet`, `replace: cboden/ratchet` (drop-in for `^0.4`) |

## Engineering rules

| # | Rule | How we apply it | Anti-example |
|---|---|---|---|
| 1 | **Do not reinvent the wheel** | Check upstream `0.4.x`, the `react/*` ecosystem, `ratchet/rfc6455` and the PSR interfaces first. A new class/abstraction only when there is no ready solution. Every change should be suitable for an upstream PR | Own WebSocket handshake, own PSR-7 parser, own `MessageComponentInterface` implementation |
| 2 | **Minimal distance from upstream** | Change only what the task requires; no "while I'm here" refactorings or renames | Rewriting `WsServer` "on the side" while fixing `composer.json` |
| 3 | **Single source of truth** | Shared logic (frame parsing, header handling, framing) lives in one method/class reused by `WsServer`, `HttpServer`, `App`. Do not copy snippets between classes | A copy of the `Origin` check in both `OriginCheck` and `Router` |
| 4 | **Backwards compatibility with 0.4.x** | Public signatures and behaviour do not change; only additive changes (new optional parameters/methods) are allowed. Removal only in a major version | Renaming `onOpen()` or reordering arguments |
| 5 | **Deprecation gate (PHP 8.4)** | `./docker/app check-deprecations` = 0 `Deprecated:` on 8.4, in CI and locally. Never silence errors via `@`, `error_reporting()` or attributes without a reason | `error_reporting(E_ALL & ~E_DEPRECATED)` in tests, `@$value` |
| 6 | **Do not swallow errors** | Exceptions carry context (class, transport, status); `trigger_error` only where upstream already does it; no empty `catch` | `try { … } catch (\Throwable $e) {}` |
| 7 | **Test first, then fix** | Every bug fix starts with a failing test (regression), then the fix; no PR is merged without a test. For transport work — an integration smoke test (echo server + client) | "Fixed by hand, the old tests are enough" |
| 8 | **New dependencies only with justification** | Nothing new in `require` without an issue; tooling (phpcs, phpstan, phpunit) only in `require-dev`. Tightening constraints is deliberate and recorded in the changelog | Pulling in `symfony/*` at runtime for a single function |
| 9 | **Static analysis and style** | PSR-12 (`phpcs`), PHPStan `level: 5`+; a new baseline entry only with a comment. Locally — `make lint` | A baseline that has grown over all of `src/` |
| 10 | **Atomic commits and PRs** | Conventional Commits (`fix:`, `feat:`, `chore:`), one logical change per commit; a PR links an issue, has green CI and a `CHANGELOG.md` entry; force-push to `0.4.x` is forbidden | One commit "fix everything + composer + docs" |
| 11 | **Release discipline** | Tag only from the head of `0.4.x`, annotated semver `v0.4.N`, changelog entry, Packagist webhook verified; retagging is not allowed — ship a new patch release instead | Rewriting `v0.4.5` after it was published |
| 12 | **Document behaviour** | Any behaviour change goes to README/CHANGELOG; non-obvious decisions go to `docs/decisions/` (for example "why `psr7` is pinned to `^2.12.3`") | "We changed the constraint and nobody knows why" |
| 13 | **Secure by default** | No secrets in code or logs; never disable TLS verification; vulnerable ranges (`psr7 < 2.12.3`, `rfc6455 0.3.x`) are excluded by constraints; keep `SECURITY.md` current | `'verify' => false`, a token in `var_dump` |
| 14 | **Style and naming** | PSR-12; `StudlyCaps` classes, `camelCase` methods, `UPPER_SNAKE` constants; the public API has no abbreviations and no consumer project brands in names | `class AcmeWsServer`, `function hndlMsg()` |

Extension points (for example connection lifecycle hooks, if they become necessary) are implemented as
**public optional interfaces** with tests and a README description — never by changing default behaviour.

## Release checklist

1. `make check` green (`./docker/app check-deprecations`, PHPCS, PHPStan, PHPUnit).
2. `PHP_VERSION=8.0 docker/app test` and `PHP_VERSION=8.5 docker/app test` green.
3. `composer validate --strict` valid.
4. `CHANGELOG.md` entry added.
5. `git tag -a v0.4.N -m "..."` + `git push origin 0.4.x v0.4.N`.
6. Packagist shows the new version (`https://packagist.org/packages/anton-am/ratchet`).

## PHP policy (deprecation gate)

* `php >=8.0`, CI matrix `8.0 … 8.5`.
* Every file in `src/` is linted with `error_reporting=E_ALL` by `./docker/app check-deprecations`
  (`php -l` per file); **any `Deprecated:` notice fails the build**. The same gate runs in CI (all
  versions), in `make lint`, in `docker/app test` and in `docker/app lint`.
* Dependency floors are chosen so that no deprecations are emitted on PHP 8.4/8.5:
  `ratchet/rfc6455 ^0.4.1` (0.3.x is noisy on 8.4) and `guzzlehttp/psr7 ^2.12.3`
  (1.x, and 2.x `< 2.12.3`, are affected by security advisories). See
  [`decisions/0001-fork-and-dependency-floors.md`](decisions/0001-fork-and-dependency-floors.md).

## Semver

`0.4.N` patches must stay backwards compatible with `cboden/ratchet 0.4.4`: public interfaces, constructor
signatures and the PSR-7 based `$conn->httpRequest` contract do not change. Behavioural or API changes go
into a minor/major line.

## Style

PSR-12 via `phpcs.xml.dist` — the sniffs that the upstream code base violates today are disabled with a
comment, and are re-enabled incrementally; PHPStan level 5 over `src/` with a small, shrink-only baseline;
4 spaces, LF, UTF-8, final newline (`.editorconfig`).
