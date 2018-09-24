# Installation

## Requirements

* PHP &gt;=7.1
* [Composer](https://getcomposer.org)

{% hint style="warning" %}
_Currently, using a child-theme to extend a Lumberjack theme is unsupported._
{% endhint %}

## Using the Installer \(Recommended\)

This is the recommended way to create a new site and assumes you want to build on top of [Bedrock](https://roots.io/bedrock). If you're new to using WordPress with Composer this is the simplest \(and quickest\) way to get up and running as the installer will do all the heavy lifting for you.

### Pre-requisites

Download the Lumberjack Bedrock Installer with Composer _\(you only need to do this the first time\)_:

```bash
composer global require lumberjack-bedrock
```

Make sure that Composer's global `vendor` bin directory is in your `$PATH` so that the installer can be used.

### Create a new site

Once the installer is available on your computer you can create a new Lumberjack/Bedrock site using:

```bash
lumberjack-bedrock new my-site
```

This will create a new folder `my-site` in the current working directory, install the latest version of Bedrock, add the `lumberjack-core` dependency and download the most recent Lumberjack starter theme.

To complete the install do the following:

1. Copy `.env.example` to `.env` and update environment variables:
   * `DB_NAME` - Database name
   * `DB_USER` - Database user
   * `DB_PASSWORD` - Database password
   * `DB_HOST` - Database host
   * `WP_ENV` - Set to environment \(`development`, `staging`, `production`\)
   * `WP_HOME` - Full URL to WordPress home \(e.g. [http://example.com](http://example.com)\)
   * `WP_SITEURL` - Full URL to WordPress including subdirectory \(e.g. [http://example.com/wp](http://example.com/wp)\)
   * `AUTH_KEY`, `SECURE_AUTH_KEY`, `LOGGED_IN_KEY`, `NONCE_KEY`, `AUTH_SALT`, `SECURE_AUTH_SALT`, `LOGGED_IN_SALT`, `NONCE_SALT` - Generate with [wp-cli-dotenv-command](https://github.com/aaemnnosttv/wp-cli-dotenv-command) or from the [Roots WordPress Salt Generator](https://cdn.roots.io/salts.html)
2. Set your site vhost document root to `/path/to/my-site/web/`
3. Access WP admin at `http://example.com/wp/wp-admin` and activate the Lumberjack theme.

### Additional Options

For more information on additional installer options, please see the [Lumberjack Installer](https://github.com/Rareloop/lumberjack-bedrock-installer) package on GitHub.

## Manual Installation

If you don't plan to use Bedrock and have another Composer based setup for WordPress, you can do the following:

1. Download the [Lumberjack Starter Theme](https://github.com/Rareloop/lumberjack) and add it to your WordPress theme directory.
2. Now add the Lumberjack Core dependency via Composer:

   ```bash
    composer require rareloop/lumberjack-core
   ```

3. If your setup doesn't include your Composer `vendor/autoload.php` file outside of the theme, you'll need to add the following to the top of the themes `functions.php`:

   ```php
    require_once('path/to/composer/vendor/autoload.php');
   ```

## Add to existing theme

Lumberjack can be added to an existing theme as long as Composer is being used as part of your setup. You're able to use as little or as much as you need and the framework will play nicely alongside more traditional WordPress code.

1. Add the Lumberjack Core dependency via Composer:

   ```bash
    composer require rareloop/lumberjack-core
   ```

2. Copy the following directories from the [Lumberjack Starter Theme](https://github.com/Rareloop/lumberjack) into your theme:
   * `app`
   * `bootstrap`
   * `config`
   * `views`
3. Create an empty `routes.php` file at the root of your theme directory.
4. Add the following to the top of your `functions.php` file:

   ```php
    use App\Http\Lumberjack;

    // Create the Application Container
    $app = require_once('bootstrap/app.php');

    // Bootstrap Lumberjack from the Container
    $lumberjack = $app->make(Lumberjack::class);
    $lumberjack->bootstrap();

    // Import our routes file
    require_once('routes.php');

    // Set global params in the Timber context
    add_filter('timber_context', [$lumberjack, 'addToContext']);
   ```

5. If your setup doesn't include your Composer `vendor/autoload.php` file outside of the theme, you'll need to also add the following to the top of the themes `functions.php`:

   ```php
    require_once('path/to/composer/vendor/autoload.php');
   ```

