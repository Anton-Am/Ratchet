## What and why

<!-- What does this change do, and which problem does it solve? Link the issue. -->

## Checklist

- [ ] CI is green (PHPUnit on PHP 8.0 – 8.5, Docker matrix, PHPCS, PHPStan)
- [ ] Deprecation gate clean: `./docker/app check-deprecations src` reports 0 `Deprecated:` on PHP 8.4
- [ ] `composer validate --strict` passes
- [ ] `CHANGELOG.md` entry added
- [ ] Backwards compatible with `0.4.x` (or breaking change marked and agreed in an issue)
- [ ] Branch target is `0.4.x`
