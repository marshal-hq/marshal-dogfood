# marshal-dogfood

This is the sandbox repo I use to test the [Marshal](https://marshalhq.dev) GitHub Action before shipping changes to it. Not a real product. Don't use any of this code for anything.

The `pom.xml` is a Spring Boot 3 project with ~20 direct dependencies — Spring starters, Apache Commons, Jackson, Guava, and a handful of smaller libs. After transitive resolution it expands to ~150–200 deps, which is enough to exercise the scanner's parallel fetch path and give the PR comment something real to show.

## What I use this repo for

Opening a PR that bumps a dep version, then checking:

- does the Action post exactly one comment
- does a second push edit that comment instead of creating another one
- does `fail-on: fail` actually fail the check
- what does the risk output look like for a dep that changed maintainers or dropped a signature

That's it. The Spring Boot app itself doesn't do anything interesting and never will.

## How the Action is wired

```yaml
# .github/workflows/marshal.yml
name: Marshal

on:
  pull_request:
    paths:
      - 'pom.xml'

jobs:
  marshal:
    runs-on: ubuntu-latest
    permissions:
      contents: read
      pull-requests: write
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 1

      - name: Marshal scan
        uses: marshal-hq/marshal-action@dev
        with:
          pom-path: pom.xml
          threshold: red
          comment-on-pr: 'true'
          fail-on: fail
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

## If you're reading this

Marshal is a behavioral supply-chain security scanner for JVM dependencies. It watches Maven Central for things CVE scanners structurally can't catch — maintainer changes, signature drops, new install hooks, sudden dependency explosions. The CLI and GitHub Action are open-source. Hosted watching is paid.

→ [marshalhq.dev](https://marshalhq.dev)