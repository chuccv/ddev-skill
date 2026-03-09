---
name: ddev-magento
description: "Manage and operate Magento 2 on DDEV with MySQL, Redis, Elasticsearch, RabbitMQ, and Varnish services."
---

# Workflow

This skill helps you interact efficiently with Magento 2 projects running on the DDEV platform.

## 1. Startup & Setup
*   **Start environment:** Run `ddev start`.
*   **Install Composer packages:** Run `ddev composer install`.
*   **Check service status:** Run `ddev describe` or `ddev status`.

## 2. Deployment & Upgrade Workflow
When code or database changes occur, follow these steps:
1.  **Upgrade database:** `ddev magento setup:upgrade` (or `ddev upgrade` if the project has a custom command).
2.  **Compile DI:** `ddev magento setup:di:compile`.
3.  **Deploy Static Content:** `ddev magento setup:static-content:deploy -f`.
4.  **Flush cache:** `ddev magento cache:flush`.

## 3. Cache Management
*   **Flush all caches:** `ddev magento cache:flush`.
*   **Clean specific cache:** `ddev magento cache:clean [type]`.
*   **Enable/Disable cache:** `ddev magento cache:enable` or `ddev magento cache:disable`.

## 4. Services Management
*   **Redis:** Access redis-cli via `ddev redis-cli`.
*   **MySQL:** Access MySQL via `ddev mysql`.
*   **RabbitMQ:** View the management interface via the URL provided by `ddev describe`.
*   **Elasticsearch/OpenSearch:** Check status using `ddev exec curl -s http://elasticsearch:9200`.
*   **Varnish:** Check Varnish configuration and status.

## 5. Troubleshooting
*   **View web server logs:** `ddev logs -f web`.
*   **View database logs:** `ddev logs -f db`.
*   **Restart environment:** `ddev restart`.
*   **SSH into container:** `ddev ssh`.

## 6. Custom Commands
If defined in the project, you can use:
*   `ddev reindex`: To reindex all data.
*   `ddev setup-install`: To reinstall Magento from scratch.
*   `ddev setup-domain`: To configure domains for the project.
