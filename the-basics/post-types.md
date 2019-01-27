# Post Types

## Introduction

Typically in WordPress when you're querying posts you get [`WP_Post`](https://codex.wordpress.org/Class_Reference/WP_Post) objects back. Timber have taken this a step further and return a `Timber/Post` object instead. This [has a ton of great helper methods and properties](https://timber.github.io/docs/reference/timber-post/) which makes it easier and more expressive to use.

```php
use Timber\Post;

$post = new Post(1);
$posts = Timber::get_posts($wpQueryArray);
```

Lumberjack has its own Post object which makes it easier and more expressive to run queries.

```php
use Rareloop\Lumberjack\Post;

$post = new Post(1);
$collection = Post::query($wpQueryArray);
```

This becomes especially powerful when you start registering **Custom Post Types**.

```php
use App\PostTypes\Product;

$post = new Product(1);
$collection = Product::query($wpQueryArray);
```

In this example `$collection` contains `App\PostType\Product` objects. That allows you to add your own methods to a product and encapsulate logic in one place.

```php
use App\PostTypes\Product;

$collection = Product::query($wpQueryArray);

foreach ($collection as $product) {
    echo $product->price();
}
```

```php
namespace App\PostTypes;

use Rareloop\Lumberjack\Post;

class Product extends Post
{
    // ...

    public function price()
    {
        // Get the price from the ACF field for this product
        return get_field('price', $this->id);
    }
}
```

## Register Custom Post Types

First, create a new file in `app/PostTypes/`. We recommend using singular names. For this example, lets add a `Product.php` file there.

You can use this boilerplate to get you started:

```php
namespace App\PostTypes;

use Rareloop\Lumberjack\Post;

class Product extends Post
{
    /**
     * Return the key used to register the post type with WordPress
     * First parameter of the `register_post_type` function:
     * https://codex.wordpress.org/Function_Reference/register_post_type
     *
     * @return string
     */
    public static function getPostType()
    {
        return 'products';
    }

    /**
     * Return the config to use to register the post type with WordPress
     * Second parameter of the `register_post_type` function:
     * https://codex.wordpress.org/Function_Reference/register_post_type
     *
     * @return array|null
     */
    protected static function getPostTypeConfig()
    {
        return [
            'labels' => [
                'name' => __('Products'),
                'singular_name' => __('Product'),
                'add_new_item' => __('Add New Product'),
            ],
            'public' => true,
        ];
    }
}
```

Lumberjack will handle the registering of the post type for you. In order to do that, it requires 2 methods \(documented above\):

* `getPostType()`
* `getPostTypeConfig()`

In order for Lumberjack to register your post type, you need to add the class name to the `config/posttypes.php` config file.

```php
return [
    /**
     * List all the sub-classes of Rareloop\Lumberjack\Post in your app that you wish to
     * automatically register with WordPress as part of the bootstrap process.
     */
    'register' => [
        App\PostTypes\Product::class,
    ],
];
```

And that's it! You can now start using your new Custom Post Type.

{% hint style="info" %}
**Tip**: Try and avoid using ACF's `get_field` outside of a Post Type class where possible. This will help make your application easy to change.
{% endhint %}

```php
$product = new Product;

// Bad
echo get_field('price', $product->id);

// Good: The knowledge of how to get the price is encapsulated within the Product class
echo $product->price();
```

## Available Methods for `Rareloop\Lumberjack\Post`

Lumberjack's `Post` class extends `Timber\Post`, and adds some convenient methods for you:

```php
use Rareloop\Lumberjack\Post;
use App\PostTypes\Product;

// Get all published posts, with 10 per page, ordered ascending by title
$posts = Post::all(10, 'title', 'asc');

// Accepts the WP_Query args as an array. By default it will filter by published posts for the correct post type too
$products = Product::query(['s' => 'Toy Car']);
```

## Extending Post Types

The Lumberjack `Post` class can be extended with custom functionality at runtime \(the class is "macroable"\). The following example adds an `acf` method to the `Post` class that can be used to access Advanced Custom Field \(ACF\) field values:

```php
use Rareloop\Lumberjack\Post;

// Add custom function
Post::macro('acf', function ($field) {
    return get_field($field, $this->id);
});

// Use the functionality
$post = new Post;
$value = $post->acf('custom_field_name');
```

