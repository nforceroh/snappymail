# snappymail

[Snappymail](https://github.com/the-djmaze/snappymail) webmail client, built on
[nginx-php](https://github.com/nforceroh/nginx-php) with the `gnupg` PHP extension
compiled from source so PGP operations work out of the box.

```dockerfile
FROM ghcr.io/nforceroh/snappymail:latest
```

## What's inside

- Snappymail 2.38.2 unpacked into `/data/web` (the nginx-php web root)
- PHP 8.5 with the extension set Snappymail needs (`imap`, `pdo_sqlite`,
  `pdo_mysql`, `tidy`, `sodium`, `zip`, `apcu`, `imagick`, `uuid`, `intl`, ...)
- `php-gnupg` compiled from source against the system `gpgme`
- nginx serving Snappymail on port 8080, php-fpm on localhost:9000

## Runtime layout

| Path | Purpose |
| --- | --- |
| `/data/web` | pristine Snappymail source, copied in at build time |
| `/snappymail` | volume; live data tree, seeded from `/data` on first boot |
| `/etc/nginx/conf.d/default.conf` | nginx vhost serving `/snappymail/web` on 8080 |
| `/etc/php85/conf.d/snappymail.ini` | PHP limits tuned for webmail |
| `/etc/php85/conf.d/gnupg.ini` | loads the compiled `gnupg.so` extension |

On first boot the `init-snappymail` s6 service copies `/data` into the
`/snappymail` volume, enables error and auth logging to files under
`/snappymail/web/data/_data_/_default_/logs/`, and fixes ownership to `www-data`.

## Services

s6-overlay runs the nginx-php longruns (`svc-nginx`, `svc-php`) plus one
image-specific init service:

- `init-snappymail` — one-shot data seeding and log setup before the daemons start

## Build-time tunables

| ARG | Default | Description |
| --- | --- | --- |
| `BUILD_DATE` | `now` | label metadata |
| `VERSION` | `unknown` | label metadata; CI passes the build date |
| `SNAPPYMAIL_SOURCE` | `release` | `release` downloads the versioned tarball; `ref` builds a git ref |
| `SNAPPYMAIL_VERSION` | `2.38.2` | Snappymail release to download when `SNAPPYMAIL_SOURCE=release` |
| `SNAPPYMAIL_REF` | *(empty)* | git ref (branch, tag, or SHA) to build when `SNAPPYMAIL_SOURCE=ref` |
| `PHP_GNUPG_REF` | `gnupg-1.5.4` | php-gnupg tag to compile |

## Runtime environment

| Variable | Default | Description |
| --- | --- | --- |
| `LOG_TO_STDERR` | `true` | enable Snappymail error logging to `errors.log` with PHP errors included |
| `UPLOAD_MAX_SIZE` | `25M` | reserved for future use; PHP limits are set in `snappymail.ini` |
| `MEMORY_LIMIT` | `128M` | reserved for future use; PHP limits are set in `snappymail.ini` |
| `SECURE_COOKIES` | `true` | reserved for future use; cookie flags are set in `snappymail.ini` |

## Local development

```sh
make build   # build the image
make test    # build, then run tests/run_tests.sh
make run     # run with 8080 published
```

CI is the only publisher — see
[nforceroh/.github](https://github.com/nforceroh/.github) for the shared build and
cleanup workflows. Published tags: `latest`, `main`, date and sha tags.
