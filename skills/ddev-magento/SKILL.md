---
name: ddev-magento
description: Use when setting up, managing, or troubleshooting a Magento 2 project on DDEV — covers fresh install, service configuration (Redis, OpenSearch, RabbitMQ, Varnish), multi-store, Xdebug, magerun2, and common dev workflows.
---

# DDEV Magento 2

## Overview

Magento 2 on DDEV uses project type `magento2` with docroot `pub`. Services (OpenSearch, Redis, RabbitMQ, Varnish) are added as DDEV add-ons. Always run Magento CLI via `ddev magento` (not `bin/magento` directly).

## Fresh Install

```bash
# 1. Configure DDEV
ddev config --project-type=magento2 --docroot=pub --upload-dirs=media \
  --disable-settings-management --php-version=8.3 --database=mysql:8.0

# 2. Add OpenSearch
ddev add-on get ddev/ddev-opensearch

# 3. Install Magento via Composer
ddev start
ddev composer create-project --repository https://repo.magento.com/ \
  magento/project-community-edition
rm -f app/etc/env.php

# 4. Run setup installer
ddev magento setup:install \
  --base-url="https://my-site.ddev.site/" \
  --cleanup-database --db-host=db --db-name=db --db-user=db --db-password=db \
  --search-engine=opensearch --opensearch-host=opensearch --opensearch-port=9200 \
  --admin-firstname=Admin --admin-lastname=User \
  --admin-email=admin@example.com --admin-user=admin --admin-password=Password123

# 5. Dev setup
ddev magento deploy:mode:set developer
ddev magento module:disable Magento_TwoFactorAuth Magento_AdminAdobeImsTwoFactorAuth
ddev config --disable-settings-management=false
ddev magento setup:config:set --backend-frontname="admin_ddev" --no-interaction
ddev launch /admin_ddev
```

> **Composer auth:** username = public key, password = private key from [marketplace.magento.com](https://marketplace.magento.com/customer/accessKeys/).

## Quick Reference

| Task | Command |
|------|---------|
| Full upgrade | `ddev magento setup:upgrade` |
| DI compile | `ddev magento setup:di:compile` |
| Static deploy | `ddev magento setup:static-content:deploy -f` |
| Flush cache | `ddev magento cache:flush` |
| Reindex | `ddev magento indexer:reindex` |
| Magerun2 | `ddev exec magerun2 <command>` |
| Redis CLI | `ddev redis-cli` |
| Profile CLI cmd | `ddev profile-cli <command>` |
| Snapshot DB | `ddev snapshot` |
| SSH into container | `ddev ssh` |
| Tail logs | `ddev logs -f` |
| Show all URLs | `ddev describe` |

## Services

| Service | Internal host | Notes |
|---------|--------------|-------|
| OpenSearch | `opensearch:9200` | Recommended for M2.4+ |
| Elasticsearch 8 | `elasticsearch:9200` | Alternative |
| Redis | `redis:6379` | Session + cache |
| RabbitMQ | `rabbitmq` | Configure in `env.php` |
| Varnish | `varnish:80` | Official add-on; traffic routed through Varnish |
| PhpMyAdmin | URL from `ddev describe` | DB management |
| XHGui | URL from `ddev describe` | Performance profiling |

## Add-ons

| Add-on | Install | Purpose |
|--------|---------|---------|
| OpenSearch | `ddev add-on get ddev/ddev-opensearch` | Search engine (recommended M2.4+) |
| Redis | `ddev add-on get ddev/ddev-redis` | Cache + session store |
| RabbitMQ | `ddev add-on get ddev/ddev-rabbitmq` | Message queue |
| Cron | `ddev add-on get ddev/ddev-cron` | Run Magento scheduler every minute |
| Memcached | `ddev add-on get ddev/ddev-memcached` | Alternative cache backend |
| Varnish | `ddev add-on get ddev/ddev-varnish` | Full-page cache (official add-on) |

```bash
ddev add-on list               # list all available add-ons
ddev add-on list --installed   # list installed add-ons
ddev add-on search <keyword>   # search by keyword
```

After installing any add-on, run `ddev restart` to apply changes.

## OpenSearch Memory Limit

The `ddev/ddev-opensearch` add-on ships **no memory ceiling**: the container inherits
the whole host, and the JVM sizes its heap from host RAM. On a machine running several
DDEV projects at once, each idle OpenSearch quietly holds 1-3 GB. Cap every project.

Add `.ddev/docker-compose.opensearch_memory.yaml` — the name matters, it must sort
**after** `docker-compose.opensearch.yaml` so its `OPENSEARCH_JAVA_OPTS` wins the merge:

```yaml
services:
  opensearch:
    environment:
      - "OPENSEARCH_JAVA_OPTS=-Xms<half of heap> -Xmx<heap>"
    deploy:
      resources:
        limits:
          memory: <ceiling>
    healthcheck:
      disable: true
```

Then `ddev restart`. Verify with `docker inspect ddev-<project>-opensearch --format '{{.HostConfig.Memory}}'`.

**Set `-Xms` below `-Xmx`.** Every OpenSearch guide says to make them equal — that
advice is for production nodes, where it avoids heap resizing under load. On a dev
machine running several projects it just parks RAM: measured heap actually in use is
200-320 MB, so `-Xms512m` commits 512 MB at boot to hold 210 MB. Halving `-Xms` took
two projects from 1.08 GB / 0.99 GB down to 0.82 GB / 0.80 GB, verified across a full
`indexer:reindex` and 10 minutes of idle afterwards.

**A high ceiling costs nothing.** It caps growth, it does not reserve RAM — a container
capped at 1500M that uses 800 MB takes 800 MB from the host. Lowering the ceiling saves
nothing and only moves you closer to an OOM kill. Cut RSS with `-Xms`; leave headroom in
the ceiling.

**Sizing.** The ceiling covers heap **plus** native memory (mmap'd index segments,
k-NN graphs, ML models) — budget roughly 2x the heap, never the heap alone:

| Workload | Heap | Ceiling |
|---|---|---|
| Sample data only | 256m | 512M |
| Any real catalog, CE or EE | 512m | 1500M |
| Semantic / vector search (ML Commons models deployed) | 2g | 4g |

512M is tighter than it looks: two projects on real catalogs booted fine on
256m/512M, served traffic, reindexed successfully, and were OOM-killed minutes
later. Start at 512m/1500M unless the project only ever holds sample data.

**Verify by reindexing, not by watching the container start.** OpenSearch boots fine on
a ceiling it cannot survive indexing on:

```bash
ddev magento indexer:reindex catalogsearch_fulltext
docker ps -a --filter name=ddev-<project>-opensearch --format '{{.Status}}'
```

- `Exited (137)` = OOM-killed by the ceiling. Raise it one row in the table above.
- `429` write-rejections during reindex = heap too small.
- Container refuses to start = heap >= ceiling. Heap must stay strictly below.

Sitting at 95-99% of the ceiling is normal and not a leak — most of it is reclaimable
page cache for the index files. The floor for a real catalog is ~800 MB: heap in use
plus native memory no config reaches (Lucene mmap'd segments, thread stacks, metaspace,
direct buffers). Below that, cut data — stale `top_queries-*` indices from the
query-insights plugin and old `magento2_product_*_v*` generations pile up — not heap.

**Changing the ceiling without a restart:** `docker update --memory 4g --memory-swap 4g
ddev-<project>-opensearch` applies immediately and keeps deployed ML models loaded.
Write the same value into the YAML afterwards or the next `ddev restart` reverts it.

**The dashboards container too.** `ddev-<project>-opensearch-dashboards` is a browser UI
for inspecting indices; Magento never talks to it. It costs ~250-350 MB per project and
is also uncapped — cap it at 512M, or drop it from the add-on if nobody opens it.

## Dropping a Service an Add-on Generated

`docker-compose.<addon>.yaml` carries a `#ddev-generated` marker: edits are lost when the
add-on is reinstalled. Do not delete the service block there. Add an override file that
gives the service a profile instead — Compose skips any service whose profile is not
activated, and DDEV activates none of its own:

```yaml
# .ddev/docker-compose.no-os-dashboards.yaml
services:
  opensearch-dashboards:
    profiles:
      - disabled
```

`ddev restart` then reports `Waiting for additional project containers [opensearch]` with
no dashboards entry. Works for any generated service nothing else declares in `depends_on`
— check first, because Compose errors on a `depends_on` pointing at a skipped service.

Dropping `opensearch-dashboards` also frees the image once no project runs it:
`opensearchproject/opensearch-dashboards:latest` is ~2 GB of layers shared with nothing.

## Multi-Store Setup

Add file `.ddev/nginx_full/magento-stores.conf`:

```nginx
map $http_host $mage_run_code {
  default '';
  store1.ddev.site storecode1;
  store2.ddev.site storecode2;
}
```

Add hostnames to `.ddev/config.yaml`:
```yaml
additional_hostnames:
  - store1
  - store2
```

## Xdebug

```bash
ddev xdebug on     # enable
ddev xdebug off    # disable
ddev xdebug status # check status
```

## Sample Data

```bash
ddev magento sampledata:deploy
ddev magento setup:upgrade
```

## Common Mistakes

| Problem | Cause | Fix |
|---------|-------|-----|
| Search not working | OpenSearch add-on not installed | `ddev add-on get ddev/ddev-opensearch` |
| Admin login loop | 2FA still enabled | `ddev magento module:disable Magento_TwoFactorAuth` |
| Static files 404 | Static content not deployed | `ddev magento setup:static-content:deploy -f` |
| Composer auth fail | Wrong key | username = public key, password = private key |
| Symlink errors | M2 creates symlinks with `/var/www/html/...` paths | Use `ddev exec` instead of host symlinks |
| Settings overwritten | `--disable-settings-management` not set | Add flag during `ddev config` |
