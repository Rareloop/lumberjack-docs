---
description: >-
  Collections are a thin wrapper around arrays that make them much easier to
  work with.
---

# Collections

You can create a collection in a couple of ways:

```php
use Tightenco\Collect\Support\Collection;

$data = new Collection(['Jayne', 'Milo']);

// or using the global helper
$data = collect(['Jayne', 'Milo']);
```

If you are used to working with arrays, you can continue using them in the same way.

```php
$data = collect(['Jayne', 'Milo']);

// Get the first name, or if it doesn't exist set the value to null
$firstName = $data[0] ?? null;
```

However, collections come with a huge assortment of handy functions that can make your life easier.

```php
$data = collect(['Jayne', 'Milo']);

// Get the first name, or if it doesn't exist set the value to null
$firstName = $data->first();
```

The collection class that Lumberjack uses comes from Laravel, [via a split package by Tighten Co.](https://github.com/tightenco/collect)

For a list of the available methods, please refer to their documentation: [https://laravel.com/docs/5.8/collections#available-methods](https://laravel.com/docs/5.8/collections#available-methods)

{% hint style="info" %}
Note: Laravel's collections may change over time as the add/remove features etc. Make sure you are always referring to the correct version of their documentation.

Lumberjack will use the latest collections version for Laravel `v5.x` _(with the minimum version being `v5.6`)_
{% endhint %}

## Extending collections

Similar to post types and the query builder, you can add your own methods to the collection class using macros. For more information about this, please refer to Laravel's documentation:

[https://laravel.com/docs/5.8/collections#extending-collections](https://laravel.com/docs/5.8/collections#extending-collections)
