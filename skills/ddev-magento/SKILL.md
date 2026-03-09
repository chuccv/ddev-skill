---
name: ddev-magento
description: "Advanced management for Magento 2 on DDEV with MariaDB, Redis, Elasticsearch 8, RabbitMQ, Varnish, PhpMyAdmin, and XHGui/XHProf."
---

# Workflow

This skill provides comprehensive management for high-performance Magento 2 environments on DDEV.

## 1. Environment & Services
*   **Start/Stop:** Use `ddev start` or `ddev stop`.
*   **Project Info:** Run `ddev describe` to see URLs for:
    *   **Web Server:** (Main store URL)
    *   **PhpMyAdmin:** Database management interface.
    *   **Mailpit:** Email testing interface.
    *   **XHGui:** Performance profiling UI.
    *   **RabbitMQ:** Management dashboard.

## 2. Magento 2 Operations
*   **Full Upgrade:** Run `ddev upgrade` (Custom command) or `ddev magento setup:upgrade`.
*   **Reindexing:** Run `ddev reindex`.
*   **Compilation:** `ddev magento setup:di:compile`.
*   **Static Assets:** `ddev magento setup:static-content:deploy -f`.
*   **Cache:** `ddev magento cache:flush`.

## 3. Specialized Services Management
*   **Elasticsearch 8:** Access via `http://elasticsearch:9200` inside the container. Check status: `ddev exec curl -s elasticsearch:9200`.
*   **Redis:** Direct CLI access via `ddev redis-cli`.
*   **Varnish:** Managed via the Varnish container. Purge cache using `ddev magento cache:flush`.
*   **RabbitMQ:** Configure in `env.php` using host `rabbitmq`.
*   **Profiling (XHGui/XHProf):**
    *   Enable profiling for web requests via config.
    *   Use `ddev profile-cli <command>` to profile Magento CLI commands.

## 4. Development & Troubleshooting
*   **Logs:** 
    *   `ddev logs -f` (All logs)
    *   `ddev logs -f web` (Nginx/PHP logs)
    *   `ddev logs -f db` (MariaDB logs)
*   **SSH Access:** `ddev ssh` to enter the web container.
*   **Database:** `ddev mysql` for CLI access or use PhpMyAdmin URL from `ddev describe`.
*   **Snapshot:** `ddev snapshot` to backup the database before risky operations.

## 5. Setup & Domain
*   **Fresh Install:** `ddev setup-install`.
*   **Domain Config:** `ddev setup-domain` to set up store URLs.
