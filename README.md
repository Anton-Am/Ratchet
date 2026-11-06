# Ratchet

[![CI status](https://github.com/Anton-Am/Ratchet/actions/workflows/ci.yml/badge.svg)](https://github.com/Anton-Am/Ratchet/actions)
[![Autobahn Testsuite](https://img.shields.io/badge/Autobahn-passing-brightgreen.svg)](http://socketo.me/reports/ab/index.html)
[![Latest Stable Version](https://poser.pugx.org/anton-am/ratchet/v/stable.png)](https://packagist.org/packages/anton-am/ratchet)

A PHP library for asynchronously serving WebSockets.
Build up your application through simple interfaces and re-use your application without changing any of its code just by combining different components.

## Personal fork

This repository is a **personal packaging fork** of [ratchetphp/Ratchet](https://github.com/ratchetphp/Ratchet),
maintained by [Anton-Am](https://github.com/Anton-Am) on the `0.4.x` branch.
The library code is upstream's; what changes is how it is packaged and tested.

Why fork instead of using `cboden/ratchet` directly:

* upstream's last release, `v0.4.4` (2021), is **not** PHP 8.4-safe; the PHP 8.4 fixes live only on the
  untagged `0.4.x` branch, which Packagist can serve as `0.4.x-dev` — moving code with no stable tag to pin;
* the dependency floors need to be raised: `guzzlehttp/psr7 ^2.12.3` (1.x, and 2.x `< 2.12.3`, are affected
  by security advisories) and `ratchet/rfc6455 ^0.4.1` (0.3.x emits deprecations on PHP 8.4);
* the package declares `replace: cboden/ratchet`, so it is a drop-in replacement for anything requiring
  `cboden/ratchet ^0.4` (for example `thruway/ratchet-transport`) while projects migrate.

Differences from upstream `v0.4.4`:

* `php >=8.0`, tested on PHP 8.0 – 8.5 (CI matrix plus `docker/app test`);
* `guzzlehttp/psr7 ^2.12.3 || ^3.0`, `ratchet/rfc6455 ^0.4.1`,
  `symfony/http-foundation` / `symfony/routing` `^5.4 || ^6.4 || ^7.0 || ^8.0`;
* test suite runs on PHPUnit 9.6 / 10.5 / 11.5;
* CI gates: `composer validate --strict`, a deprecation gate (`error_reporting=E_ALL` scan of `src/`),
  PHPCS (PSR-12) and PHPStan level 5.

```bash
composer require anton-am/ratchet:^0.4
```

After that `composer show cboden/ratchet` reports the package as *replaced* by `anton-am/ratchet`.

## Reviving Ratchet!

We're currently aiming to revive Ratchet to get it up to date with the latest versions and use this as a starting point for bigger updates to come.
We need your help to achieve this goal, see [ticket #1054](https://github.com/ratchetphp/Ratchet/issues/1054) for ways to help out. ❤️

## Requirements

Shell access is required and root access is recommended.
To avoid proxy/firewall blockage it's recommended WebSockets are requested on port 80 or 443 (SSL), which requires root access.
In order to do this, along with your sync web stack, you can either use a reverse proxy or two separate machines.
You can find more details in the [server conf docs](http://socketo.me/docs/deploy#server_configuration).

### Documentation

User and API documentation is available on Ratchet's website: http://socketo.me

See https://github.com/cboden/Ratchet-examples for some out-of-the-box working demos using Ratchet.

Need help?  Have a question?  Want to provide feedback?  Write a message on the [Google Groups Mailing List](https://groups.google.com/forum/#!forum/ratchet-php).

---

### A quick example

```php
<?php
use Ratchet\MessageComponentInterface;
use Ratchet\ConnectionInterface;

    // Make sure composer dependencies have been installed
    require __DIR__ . '/vendor/autoload.php';

/**
 * chat.php
 * Send any incoming messages to all connected clients (except sender)
 */
class MyChat implements MessageComponentInterface {
    protected $clients;

    public function __construct() {
        $this->clients = new \SplObjectStorage;
    }

    public function onOpen(ConnectionInterface $conn) {
        $this->clients->attach($conn);
    }

    public function onMessage(ConnectionInterface $from, $msg) {
        foreach ($this->clients as $client) {
            if ($from != $client) {
                $client->send($msg);
            }
        }
    }

    public function onClose(ConnectionInterface $conn) {
        $this->clients->detach($conn);
    }

    public function onError(ConnectionInterface $conn, \Exception $e) {
        $conn->close();
    }
}

    // Run the server application through the WebSocket protocol on port 8080
    $app = new Ratchet\App('localhost', 8080);
    $app->route('/chat', new MyChat, array('*'));
    $app->route('/echo', new Ratchet\Server\EchoServer, array('*'));
    $app->run();
```

    $ php chat.php

```javascript
    // Then some JavaScript in the browser:
    var conn = new WebSocket('ws://localhost:8080/echo');
    conn.onmessage = function(e) { console.log(e.data); };
    conn.onopen = function(e) { conn.send('Hello Me!'); };
```

## Install

The recommended way to install this library is [through Composer](https://getcomposer.org/).
[New to Composer?](https://getcomposer.org/doc/00-intro.md)

This will install the latest supported version:

```bash
composer require anton-am/ratchet:^0.4
```

See also the [CHANGELOG](CHANGELOG.md) for details about version upgrades.

This fork requires **PHP 8.0 or newer** and is tested on PHP 8.0 – 8.5.
It does not require any PHP extensions and aims to run on any platform.
It's *highly recommended to use the latest supported PHP version* for this project.

## Development

```bash
docker/app init                 # composer install inside Docker, writes vendor/ on the host
make check                      # deprecation gate + phpcs + phpstan + phpunit
PHP_VERSION=8.4 docker/app test # full suite inside Docker (8.0 … 8.5)
docker/app lint                 # deprecation gate + phpcs + phpstan inside Docker
```

See [`CONTRIBUTING.md`](CONTRIBUTING.md) and [`docs/conventions.md`](docs/conventions.md).

## License

MIT, see [LICENSE file](LICENSE).
