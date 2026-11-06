# Contributing

This repository is the Personal packaging fork of [ratchetphp/Ratchet](https://github.com/ratchetphp/Ratchet).
The only supported branch is **`0.4.x`**.

## Before opening a pull request

```bash
composer install
make check                     # deprecation gate + phpcs + phpstan + phpunit
PHP_VERSION=8.0 docker/app test # same suite on a specific PHP version
```

A pull request is merged when:

* CI is green — PHPUnit on PHP 8.0, 8.1, 8.2, 8.3, 8.4, 8.5, the Docker matrix,
  PHPCS and PHPStan;
* the **deprecation gate reports 0 `Deprecated:` notices** on PHP 8.4
  (`./docker/app check-deprecations src`, run with `error_reporting=E_ALL`);
* `CHANGELOG.md` contains an entry for the change;
* the change is backwards compatible with `0.4.x`, or is explicitly marked as a
  breaking change and discussed in an issue first.

## Commits

[Conventional Commits](https://www.conventionalcommits.org/): `fix:`, `feat:`,
`chore:`, `docs:`, `test:`, `ci:`, `refactor:`. Keep the subject in the
imperative mood and under ~72 characters.

## Coding style

* PSR-12, enforced by `vendor/bin/phpcs` (see `phpcs.xml.dist`);
* PHPStan level 5 over `src/` (`phpstan.neon`); a baseline is allowed only with a
  comment explaining why the error is accepted;
* 4 spaces, LF, UTF-8, final newline (`/.editorconfig`).

## Releases

Releases are cut by the maintainer after the merge into `0.4.x`:

1. update `CHANGELOG.md`;
2. `git tag -a v0.4.N -m "..."` and `git push origin 0.4.x v0.4.N`;
3. Packagist picks the tag up automatically (GitHub webhook / Packagist app).

Tagging is the only place the version lives — `composer.json` intentionally has
no `version` field.

## PHP support policy

Minimum is `php >=8.0`; the CI matrix is `8.0 … 8.5`. New code must not emit
deprecations on any supported version, and `0.3.x` of `ratchet/rfc6455` is not
allowed because it is not clean on PHP 8.4.

See [`docs/conventions.md`](docs/conventions.md) for the short version of these rules.
