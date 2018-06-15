---
description: A quick start guide to getting started with Lumberjack with Bedrock.
---

# Quick Start

To get started with Lumberjack on top of [Bedrock](https://roots.io/bedrock/), just do the following:

1. Run the following in your terminal:

   ```bash
   git clone --depth=1 git@github.com:roots/bedrock.git bedrock-lumberjack-site && rm -rf bedrock-lumberjack-site/.git && cd bedrock-lumberjack-site && composer require rareloop/lumberjack-core && cd web/app/themes && git clone --depth=1 git@github.com:rareloop/lumberjack.git lumberjack && rm -rf lumberjack/.git && cd ../../..
   ```

   This will install the latest version of Bedrock, add the `lumberjack-core` dependency and download the most recent Lumberjack starter theme.

2. Copy `.env.example` to `.env` and update environment variables:
   * `DB_NAME` - Database name
   * `DB_USER` - Database user
   * `DB_PASSWORD` - Database password
   * `DB_HOST` - Database host
   * `WP_ENV` - Set to environment \(`development`, `staging`, `production`\)
   * `WP_HOME` - Full URL to WordPress home \([http://example.com\](http://example.com\)\)
   * `WP_SITEURL` - Full URL to WordPress including subdirectory \([http://example.com/wp\](http://example.com/wp\)\)
   * `AUTH_KEY`, `SECURE_AUTH_KEY`, `LOGGED_IN_KEY`, `NONCE_KEY`, `AUTH_SALT`, `SECURE_AUTH_SALT`, `LOGGED_IN_SALT`, `NONCE_SALT` - Generate with [wp-cli-dotenv-command](https://github.com/aaemnnosttv/wp-cli-dotenv-command) or from the [Roots WordPress Salt Generator](https://cdn.roots.io/salts.html)
3. Set your site vhost document root to `/path/to/site/web/` \(`/path/to/site/current/web/` if using deploys\)
4. Access WP admin at `http://example.com/wp/wp-admin` and activate the Lumberjack theme

