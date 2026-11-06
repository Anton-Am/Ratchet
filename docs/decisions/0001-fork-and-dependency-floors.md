# 0001 — Fork the package and pin the dependency floors

* Status: accepted
* Date: 2026-11-06

## Context

`cboden/ratchet` is used to serve WebSockets. The last upstream release, `v0.4.4` (2021-12-11), is not
PHP 8.4-safe: `#[\AllowDynamicProperties]` on `IoConnection` and the explicit `?RequestInterface`
signatures in `HttpServerInterface`, `Router`, `WsServer`, `OriginCheck` and `NoOpHttpServerController`
exist only on the untagged `0.4.x` branch. Packagist can only serve that branch as `0.4.x-dev`, i.e. a
moving dev version without a stable tag to pin.

Upstream's constraints also stay open to versions we cannot use:

* `guzzlehttp/psr7 ^1.7` allows `1.x`, and `2.x < 2.12.3`, which are affected by security advisories;
* `ratchet/rfc6455 ^0.3.1` resolves to `0.3.x`, which emits deprecations on PHP 8.4.

## Decision

1. Publish a packaging fork as `anton-am/ratchet` from the `0.4.x` branch, released as annotated tags
   `v0.4.N`.
2. Declare `replace: {"cboden/ratchet": "self.version"}` so packages requiring `cboden/ratchet ^0.4`
   (for example `thruway/ratchet-transport`) keep resolving while consumers migrate.
3. Raise the floors: `php >=8.0`, `guzzlehttp/psr7 ^2.12.3 || ^3.0`, `ratchet/rfc6455 ^0.4.1`,
   `symfony/http-foundation` / `symfony/routing ^5.4 || ^6.4 || ^7.0 || ^8.0`, `react/* ^1.0`.
4. Do not change library code for packaging reasons; the PHP 8.4 fixes are upstream's and stay as they are.

## Consequences

* Consumers get a stable, PHP 8.4-clean version that can be pinned, and patching `vendor/` is no longer
  needed.
* The fork owns its release process and its CI gates (deprecation gate, PHPCS, PHPStan, PHPUnit on
  PHP 8.0 – 8.5).
* `anton-am/ratchet` must keep the `cboden/ratchet` public API intact, otherwise `replace` becomes a lie.
  See rule 4 in [`../conventions.md`](../conventions.md).
* Alternative considered and rejected: `"cboden/ratchet": "0.4.x-dev"` — a dev version without a tag and
  without control over constraints.
