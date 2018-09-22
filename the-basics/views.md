# Views

Here we're going to walkthrough how a custom post type links through to a controller, and then finally ends up in a view.

## Post Type Data
After you've [setup a custom post type](post-types.md#register-custom-post-types) you'll want to make effort to keep data logic constrained to the Post Types itself; this is done by adding methods on the custom post type class for resolving your data.

Here is an example where we're getting a `price` field from Advanced Custom Fields.
```
// app/PostTypes/Product.php
namespace App\PostTypes;
​
use Rareloop\Lumberjack\Post;
use Rareloop\Lumberjack\QueryBuilder\Post as QueryBuilderPost;
​
class Product extends Post
{
	...

	public function price()
	{
		return get_field('price', $this->id);
	}
}
```

## Single Controller
As covered in the [WordPress Controllers](wordpress-controllers.md) section, the normal WordPress `page.php` and `single.php` type files are now used as controllers to provide the correct response, and to make sure that only the data that is needed is passed through.

Here we are getting the value returned by our new `price` method and assigning it to the context variable so it can be used in our twig template.
```
// single-product.php
namespace App;

use Timber\Timber;
use Rareloop\Lumberjack\Post;
use Rareloop\Lumberjack\Http\Responses\TimberResponse;
use App\PostTypes\ProductType;

class SingleProductController
{
	public function handle()
	{
        	$context = Timber::get_context();
        	$product = new Product();
        	$context['post'] = $product;
        	$context['title'] = $product->title;
        	$context['content'] = $product->content;

		// get the price field
		$context['price'] = $product->price();
        
        	return new TimberResponse('templates/products-page.twig', $context);
	}
}
```

## Twig Template
Finally in our twig template we can now output our price value the same way we access the rest of the context.
```
// views/templates/product-page.twig

{% extends "base.twig" %}

{% block content %}
	<main>
		<h1>{{ title }}</h1>
		{{ content|raw }}
		<h3>Price: {{ price }}</h3>
	</main>
{% endblock %}
```
