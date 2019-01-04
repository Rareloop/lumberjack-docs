# Query Builder

Lumberjack has a built-in query builder which provides an expressive, fluent and explicit way of querying data in WordPress. It can be used instead of [WP\_Query](https://codex.wordpress.org/Class_Reference/WP_Query) to query posts \(of any type\) and means you do not have to worry about "the loop".

There are 2 ways in which you can use the query builder. Either on a [Post Type](post-types.md):

```php
use App\PostTypes\Project;

$projects = Project::builder()
    ->orderBy('date', 'asc')
    ->get();
```

Or by using the query builder directly:

```php
use App\PostTypes\Project;
use App\PostTypes\CaseStudy;

$posts = QueryBuilder::wherePostType([
    Project::getPostType(),
    CaseStudy::getPostType(),
])
->orderBy('date', 'asc')
->get();
```

## Using the query builder on a Post Type

This is the most common way to use the query builder. You can start querying posts simply by using the `Rareloop\Lumberjack\Post` class. To start a query, use the `builder()` method.

```text
Post::builder();
```

This creates an instance of `Rareloop\Lumberjack\ScopedQueryBuilder`. This class does a couple of important things:

### Returns the correct post objects

When querying a post type, you will always get the same post object back in your results rather than `WP_Post` objects.

For example, if you have a custom post type called `Employee`, when you query projects you will always get `Employee` objects back as results. Lets see what this looks like:

```php
use App\PostTypes\Employee;

$employees = Employee::builder()->get();

dump($employees);

/*
    Collection {
        #items: array:2 [
            0 => App\PostTypes\Employee,
            1 => App\PostTypes\Employee,
        ]
    }
*/
```

{% hint style="info" %}
Be sure to [check out the section on Collections](query-builder.md) if you're unfamiliar with them
{% endhint %}

The collection contains instances of the `Employee` class. This is extremely powerful as you now have access to all the behaviours that come with employees, as defined in your post type class. In this case, you may have a `photoUrl()` method on an `Employee` that knows \(encapsulates\) how to get the correct size image from the featured image:

```php
class Employee extends Post
{
    ...

    public function photoUrl() : string
    {
        $thumbnail = $this->thumbnail();

        if (empty($thumbnail)) {
            return null;
        }

        return $thumbnail->src('large');
    }
}
```

And when iterating through your results, you can safely use this `photoUrl()` method:

```php
use App\PostTypes\Employee;

$employees = Employee::builder()->get();

$employee = $employees->first();

// This will echo out the featured image url if there is one
echo $employee->photoUrl();
```

{% hint style="info" %}
All post types extend [Timber's Post object](https://timber.github.io/docs/reference/timber-post/), so you get access to all of their behaviour out of the box.

The example above is making use of Timber's `thumbnail()` method on a [Post](https://timber.github.io/docs/reference/timber-post/), and `src()` method on an [Image](https://timber.github.io/docs/reference/timber-image/).
{% endhint %}

### Query scopes

Sometimes you will need to perform the same filter on a query in multiple places within your theme.

```php
// Get all featured posts, newest first
$featuredPostIds = [1, 2];

$posts = Post::whereIdIn($featuredPostIds)
    ->orderBy('date', 'desc')
    ->get();

...

// Get the latest 3 featured posts
$featuredPostIds = [1, 2];

$posts = Post::whereIdIn($featuredPostIds)
    ->orderBy('date', 'desc')
    ->limit(3)
    ->get();
```

In this example, we are scoping the query to only show featured images. However there's 2 issues with this:

1. We doing it twice, without reusing any code
2. It can be unclear as to what you are doing

Instead, We can encapsulate the knowledge about how to find featured posts by creating a **query scope** on our post type:

```php
namespace App\PostTypes;

use Rareloop\Lumberjack\Post as LumberjackPost;

class Post extends LumberjackPost
{
    ...

    public function scopeFeatured($query)
    {
        $featuredPostIds = [1, 2];

        return $query->whereIdIn($featuredPostIds);    
    }
}
```

Now we have a query scope, we can refactor our previous queries, making them more declarative and easier to change:

```php
// Get all featured posts, newest first
$posts = Post::featured()
    ->orderBy('date', 'desc')
    ->get();

...

// Get the latest 3 featured posts
$posts = Post::featured()
    ->orderBy('date', 'desc')
    ->limit(3)
    ->get();
```

{% hint style="info" %}
Query scopes must start with the word `scope`, and must follow with the name of the method you want available to the builder. The method should also be defined in `CamelCase`. For example:

`scopeFoo()` will allow you to call `foo()` on a query.

`scopeFooBar()` will allow you to call `fooBar()` on a query.
{% endhint %}

You can also pass through parameters into the query scope:

```php
namespace App\PostTypes;

use Rareloop\Lumberjack\Post as LumberjackPost;

class Post extends LumberjackPost
{
    ...

    public function scopeExclude($query, $postId)
    {        
        return $query->whereIdNotIn([$postId]);    
    }
}

...

// Get all featured posts, newest first, excluding post id 1
$posts = Post::exclude(1)
    ->orderBy('date', 'desc')
    ->get();
```

## Available methods

All the available methods can be chained, with the exclusion of `getParameters` and `get`.

* getParameters
* wherePostType
* limit
* offset
* orderBy
* whereIdIn
* whereIdNotIn
* whereStatus
* whereMeta
* whereMetaRelationshipIs
* get
* clone

