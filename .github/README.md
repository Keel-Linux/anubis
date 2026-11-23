# anubis

Debian packaging of [Anubis](https://anubis.techaro.lol) 1.27.0, the
proof-of-work web gate of Techaro, for Keel Linux on Debian trixie.
Source and binary package `anubis`.

## Why Keel carries it

Keel Web puts Anubis behind Nginx, never at the edge (handbook decision
0030), and everything Keel installs is a `.deb` in the Keel repository
(0039). Step 5 of the first implementation of 0041 builds it from source
into the Keel repository, upstream's own `.deb` serving as a reference
only.

## Debian status

Not in Debian yet: [ITP #1102132](https://bugs.debian.org/1102132), with
the Debian Go team's work in progress on salsa,
[go-team/packages/anubis](https://salsa.debian.org/go-team/packages/anubis).
This repository builds on that work, in coordination with the ITP's
owner, and keeps it unchanged as the `debian/sid` branch.
`debian/TODO.Debian` lists what remains before the two can converge.

## Layout

[DEP-14](https://dep-team.pages.debian.net/deps/dep14/), as on salsa:

- `debian/sid`: the Go team's packaging, as on salsa, never rewritten.
- `upstream`: salsa's upstream branch plus the 1.27.0 import, with
  upstream's git history.
- `keel/trixie`: this packaging, the default branch.

Tags are `upstream/<version>` and `keel/<debian-version>`. There is no
pristine-tar branch: the `npm` component tarball is assembled from
`package-lock.json`, not released upstream, so `gbp buildpackage` builds
both orig tarballs from the `upstream/<version>` tag
(`debian/README.source`).

## Building

On trixie, with trixie-backports enabled for `golang-1.26-go` (go.mod
requires 1.26.3; trixie's Go is 1.24). The build uses no network.

```
sudo apt-get install git-buildpackage
gbp clone https://github.com/Keel-Linux/anubis.git
cd anubis
sudo apt-get build-dep ./
gbp buildpackage -us -uc
```

## Tests

The autopkgtest, `debian/tests/behind-nginx`, runs `anubis@default`
behind trixie's nginx: a browser and an unverified Googlebot are
challenged, a Googlebot from Google's range and a plain client reach the
backend, the instance listens on the loopback only, runs as a dynamic
user and refuses to start without its key. It needs systemd and breaks
the testbed; in a disposable trixie container or machine booted with
systemd:

```
autopkgtest --ignore-restrictions=isolation-container,breaks-testbed \
    anubis_*.dsc anubis_*.deb -- null
```

CI builds the package in a `debian:trixie` container with
trixie-backports, runs lintian (any error or warning fails), and runs the
autopkgtest as above in a trixie LXC system container on the self-hosted
runner keel-lxc-1. The build runs on a GitHub-hosted runner, as
Keel-Linux/common builds its packages.

## License

Expat (MIT), as upstream (`LICENSE`) and the packaging
(`debian/copyright`).
