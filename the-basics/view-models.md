# View Models

View Models are useful for preparing data for your Twig views. It’s common (and typically desirable) for the shape of the data needed by your views to be different from how it is stored in the database or Post objects. How this data is transformed can be encapsulated in a View Model, cutting down repetitive code you have in your Controllers.

## Example Controller

Below is an example of a controller that prepares a `Post` object for a media card.

```php
<?php

namespace App;

use Rareloop\Lumberjack\Post;
use Timber\Timber;
use Rareloop\Lumberjack\Http\Responses\TimberResponse;

class SingleController
{
    public function handle()
    {
        $context = Timber::get_context();
        $post = new Post;

        $date = new \DateTime($post->post_date);

        $context['card'] = [
            'title' => $post->title,
            'description' => wp_trim_words($post->content, 3),
            'published' => $date->format('Y-m-d'),
        ];

        return new TimberResponse('single.twig', $context);
    }
}
```

## Creating a View Model

First, create a View Model. With the example above, we want to transform a `Post` object to an array that has the following keys:

* `title`
* `description`
* `published`

First off, lets create an empty view model.

```php
<?php

namespace App\ViewModels;

use Rareloop\Lumberjack\ViewModel;

class MediaCardViewModel extends ViewModel
{
    public function __construct()
    {

    }
}
```

Now lets pass in the post object into the view model and keep a (protected) reference to it.

```php
<?php

namespace App\ViewModels;

use Timber\Post;
use Rareloop\Lumberjack\ViewModel;

class MediaCardViewModel extends ViewModel
{
    protected $post;

    public function __construct(Post $post)
    {
        $this->post = $post;
    }
}
```

View models are automatically converted to arrays when passed into a `twig` template. **Public methods are used as keys in this array, and the method is executed to get the value.**

Below we have added 3 public methods (`title`, `description` and `published`). These are the keys that our view needs.

```php
<?php

namespace App\ViewModels;

use Rareloop\Lumberjack\Post;
use Rareloop\Lumberjack\ViewModel;

class MediaCardViewModel extends ViewModel
{
    protected $post;

    public function __construct(Post $post)
    {
        $this->post = $post;
    }

    public function title(): string
    {
        return $this->post->title();
    }

    public function description(): string
    {
        return wp_trim_words($this->post->content, 3);
    }

    public function published(): string
    {
        $date = new \DateTime($this->post->post_date);
        return $date->format('Y-m-d');
    }
}
```

Now we can refactor our controller to use our view model instead:

```php
<?php

namespace App;

use Rareloop\Lumberjack\Post;
use Timber\Timber;
use App\ViewModels\MediaCardViewModel;
use Rareloop\Lumberjack\Http\Responses\TimberResponse;

class SingleController
{
    public function handle()
    {
        $context = Timber::get_context();
        $post = new Post;

        $context['card'] = new MediaCardViewModel($post);

        return new TimberResponse('mytemplate.twig', $context);
    }
}
```

The above will ensure the `$context` looks like the following when the view model has been converted to an array (without having to have prepared the `card` structure in the Controller):

```php
$context = [
    // ...

    'card' => [
        'title' => 'Post Title',
        'description' => 'Lorem ipsum dolor...',
        'published' => '2019-02-23',
    ],
]
```

{% hint style="info" %}
Remember: All view models (and collections) in the context are automatically flattened to arrays before being passed to `twig` views.
{% endhint %}

### Manually converting view models to arrays

If you need to convert a view model to an array before it gets passed into a view, then you can use the `toArray()` method.

```php
$mediaCard = new MediaCardViewModel($post);

$context['card'] = $mediaCard->toArray();
$context['card']['link'] = $post->link();
```

### Changing behaviour of array conversion

There may be times where the automatic array conversion doesn't do quite what you need. For example, if your view needed the following data:

```php
$context['cards'] = [
    [
        'title' => 'Post Title',
        'description' => 'Lorem ipsum dolor...',
        'published' => '2019-02-23',    
    ],
    [
        'title' => 'Post Title 2',
        'description' => 'Lorem ipsum dolor...',
        'published' => '2019-02-26',    
    ],
];
```

This can be achieved by writing your own `toArray` method on your view model.

```php
<?php

namespace App\ViewModels;

use Rareloop\Lumberjack\Post;
use Rareloop\Lumberjack\ViewModel;
use App\ViewModels\MediaCardViewModel;
use Tightenco\Collect\Support\Collection;

class MediaCardsViewModel extends ViewModel
{
    protected $posts;

    public function __construct(Collection $posts)
    {
        $this->posts = $posts;
    }

    public function cards()
    {
        // Create a MediaCardViewModel for each post
        return $this->posts->map(function (Post $post) {
            return MediaCardViewModel($post);
        });
    }

    /**
     * Overwrite the toArray method to return array of view models, with no key
     */
    public function toArray(): array
    {
        return $this->cards->toArray();
    }
}
```

## Keeping view models reusable

When writing a view model, the `__construct()` should accept all the data it needs in order to do the transformation.

For example, if you had a testimonial that has a quote and a citation, the view model could look something like this:

```php
<?php

namespace App\ViewModels;

use Rareloop\Lumberjack\ViewModel;

class TestimonialViewModel extends ViewModel
{
    protected $quote;
    protected $citation;

    public function __construct($quote, $citation)
    {
        $this->quote = $quote;
        $this->citation = $citation;
    }

    public function quote()
    {
        return $this->quote;
    }


    public function citation()
    {
        return $this->citation;
    }
}
```

Now your view model is really generic, and does not care about where the data is coming from. If you give it a quote and a citation, it will make sure the `twig` view has the correct data.

#### Named Constructors

If your view model does not know about how to get data, then you will have to fetch that data in each controller that uses the view model. For example:

```php
<?php

namespace App;

use Rareloop\Lumberjack\Post;
use Timber\Timber;
use App\ViewModels\TestimonialViewModel;
use Rareloop\Lumberjack\Http\Responses\TimberResponse;

class SingleController
{
    public function handle()
    {
        $context = Timber::get_context();
        $post = new Post;

        // Get the data from somewhere, for example from ACF
        // You would have to duplicate these two lines in each controller
        $quote = get_field('testimonial_quote', $post->id);
        $citation = get_field('testimonial_citation', $post->id);

        $context['testimonial'] = new TestimonialViewModel($quote, $citation);

        return new TimberResponse('mytemplate.twig', $context);
    }
}
```

**In order to keep your view models generic, and your controllers light (and DRY) you can create something called a "named constructor" on your view model.**

This is simply a `static` method on your view model that constructs the view model _for a specific use case_.

In our example, we are creating a testimonial from a `Post` object. So we can add the following method to our view model:

```php
<?php

namespace App\ViewModels;

use Rareloop\Lumberjack\Post;
use Rareloop\Lumberjack\ViewModel;

class TestimonialViewModel extends ViewModel
{    
    protected $quote;
    protected $citation;

    public static function forPost(Post $post)
    {
        // Get the data from somewhere, for example from ACF
        $quote = get_field('testimonial_quote', $post->id);
        $citation = get_field('testimonial_citation', $post->id);

        // Create a new instance of this class    
        return new static($quote, $citation);
    }

    public function __construct($quote, $citation)
    {
        $this->quote = $quote;
        $this->citation = $citation;
    }

    public function quote()
    {
        return $this->quote;
    }


    public function citation()
    {
        return $this->citation;
    }
}
```

And we can now refactor our controller to use the named constructor like so:

```php
<?php

namespace App;

use Rareloop\Lumberjack\Post;
use Timber\Timber;
use App\ViewModels\TestimonialViewModel;
use Rareloop\Lumberjack\Http\Responses\TimberResponse;

class SingleController
{
    public function handle()
    {
        $context = Timber::get_context();
        $post = new Post;

        $context['testimonial'] = TestimonialViewModel::forPost($post);

        return new TimberResponse('mytemplate.twig', $context);
    }
}
```

{% hint style="info" %}

{% endhint %}

You can have multiple named constructors on a view model to construct it with different data. For example you could have a `PostTeasersViewModel` which transforms a collection of posts ready for a list view.

And you could have the following named constructor:

* `latestPosts($limit = 3)`- which knows how to get the latest _n_ posts.
* `relatedPosts(Post $post)`- which knows how to get posts related to given post

## Using Hatchet

If you are using [hatchet](https://github.com/Rareloop/hatchet) (Lumberjack's CLI), you can easily create view models with the following command:

```bash
php hatchet make:viewmodel TestimonialViewModel
```
