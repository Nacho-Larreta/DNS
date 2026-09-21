# DNS

DNS allowlists for infrastructure that network-wide blocklists take down as
collateral.

## Why this exists, and why it is not a blocklist fork

A Pi-hole running as the DNS for a whole network will sinkhole first-party
observability endpoints, because the large community blocklists treat all
telemetry the same. That is the right default for third-party ad networks.
It is the wrong outcome for the crash reporter of your own application.

The obvious fix is to fork the blocklist and strip the entries out. It does
not hold up:

- **It only covers the list you forked.** Two enabled blocklists means two
  forks, and every list added later needs another one.
- **It rots.** The upstream lists regenerate daily from a dozen sources. A
  static fork is stale within a week; a live one is a pipeline to maintain
  forever, including whenever upstream changes format.
- **Pi-hole already solves this.** An allowlist entry wins over every
  blocklist, by design. Nine lines beat re-deriving six hundred thousand.

So this repository publishes the allowlist, not a derived blocklist. It
depends on nothing upstream, so there is nothing to keep in sync.

## Use it

Pi-hole → **Lists** → paste the raw URL → **Add allowlist** (the green
button; *not* Add blocklist):

```
https://raw.githubusercontent.com/Nacho-Larreta/DNS/main/allowlists/buenhogar.txt
```

Then run `pihole -g` to pull it in — subscribed lists need that, unlike the
single domains you add under **Domains**.

### Verify

From a machine using this Pi-hole as its resolver:

```bash
dig +short sentry.io
dig +short us.i.posthog.com
```

Both must return real addresses. `0.0.0.0` means the entry did not take:
check that the list was added as an allowlist rather than a blocklist, that
`pihole -g` ran, and flush with `pihole restartdns` if the resolver cached
the sinkhole.

## What is in here

`allowlists/buenhogar.txt` — Sentry and PostHog endpoints for buenhogar.ai.

The hosts are the ones the `buenhogar_kotlin` repository actually
references, measured against the source rather than copied from vendor
documentation. Two consequences worth knowing:

- **`sentry.io` is a build dependency, not only a runtime one.** Both the
  backend Gradle task and `@sentry/nextjs` upload source maps to it during
  the image build. Sinkholing it fails the build itself.
- **PostHog resolves by region.** The ingest host follows
  `BUENHOGAR_POSTHOG_REGION`, so both US and EU are listed; covering only
  one would break the day that value changes.

## Scope

These are first-party endpoints for an application we operate: crash
reports and product analytics for our own users. No ad networks, no
third-party advertising, nothing that tracks anyone across sites they did
not visit.

Allowlisting is network-wide. Anyone subscribing this accepts that these
domains resolve for every device behind that resolver — which is the point,
and worth stating out loud rather than burying.
