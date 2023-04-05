---
description: >-
  Collections are a thin wrapper around arrays that make them much easier to
  work with.
---

# Collections

You can create a collection in a couple of ways:

```php
use Illuminate\Support\Collection;

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

For a list of the available methods, please refer to their documentation: [https://laravel.com/docs/9.x/collections#available-methods](https://laravel.com/docs/9.x/collections#available-methods)

{% hint style="info" %}
Note: Laravel's collections may change over time as they add/remove features etc. Make sure you are always referring to the correct version of their documentation.
{% endhint %}

### Extending collections

Similar to post types and the query builder, you can add your own methods to the collection class using macros. For more information about this, please refer to Laravel's documentation:

[https://laravel.com/docs/9.x/collections#extending-collections](https://laravel.com/docs/9.x/collections#extending-collections)
