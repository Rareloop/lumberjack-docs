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
use Rareloop\Lumberjack\QueryBuilder;

$posts = QueryBuilder::wherePostType([
    Project::getPostType(),
    CaseStudy::getPostType(),
])
->orderBy('date', 'asc')
->get();
```

## Using the query builder on a Post Type

This is the most common way to use the query builder. You can start querying posts simply by using the `Rareloop\Lumberjack\Post` class. To start a query, use the `builder()` method.

```php
Post::builder();
```

This creates an instance of `Rareloop\Lumberjack\ScopedQueryBuilder`. This class does a couple of important things:

### Returns the correct post objects

When querying a post type, you will always get the same post object back in your results rather than `WP_Post` objects.

For example, if you have a custom post type called `Employee`, when you query employees you will always get `Employee` objects back as results. Lets see what this looks like:

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

* [getParameters](query-builder.md#getparameters)
* [wherePostType](query-builder.md#whereposttype-usdposttype)
* [whereIdIn](query-builder.md#whereidin-array-usdids)
* [whereIdNotIn](query-builder.md#whereidnotin-array-usdids)
* [whereStatus](query-builder.md#wherestatus)
* [whereMeta](query-builder.md#wheremeta-usdkey-usdvalue-usdcompare-usdtype-null)
* [whereMetaRelationshipIs](query-builder.md#wheremetarelationshipis-string-usdrelation)
* [limit](query-builder.md#limit-usdlimit)
* [offset](query-builder.md#offset-usdoffset)
* [orderBy](query-builder.md#orderby-usdorderby-usdorder-asc)
* [get](query-builder.md#get)
* [first](query-builder.md#first)
* [as](query-builder.md#as-usdpostclass)
* [clone](query-builder.md#clone)

### getParameters

**returns**: `array`

Get the current state of the query builder, as an array. These parameters can be directly fed into WP\_Query as arguments.

For example:

```php
$parameters = Post::builder()
    ->limit(3)
    ->orderBy('date', 'desc')
    ->getParameters();

// $parameters would look like this:
// [
//   "post_type" => "post"
//   "posts_per_page" => 3
//   "orderby" => "date"
//   "order" => "DESC"
// ]
```

This can be useful if you need to add your own arguments that the query builder does not support.

```php
$parameters = Post::builder()
    ->limit(3)
    ->orderBy('date', 'desc')
    ->getParameters();

$parameters['author_name'] = 'Adam';

$query = new WP_Query($parameters);
```

_Alternatively, you can add your own methods to the query builder using macros. See_ [_Extending the Query Builder_](query-builder.md#extending-the-query-builder)_._

### wherePostType\($postType\)

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$postType` | `string` \| `array` | e.g. 'post', 'page' |

Scope the query to a particular post type, or post types.

Populates `post_type` in `WP_Query`.

```php
// Single
$query->wherePostType('page');

// Multiple
$query->wherePostType (['page', 'jobs']):
```

When using the query builder from a post type, the query is automatically scoped to the correct post type.

```php
// This automatically sets the post type on the query to the Job post type
$jobs = Job::builder()->get();
```

_Reference:_ [_WP\_Query - Type Parameters_](https://codex.wordpress.org/Class_Reference/WP_Query#Type_Parameters)\_\_

### whereIdIn\(array $ids\)

| Parameters | Type | Description |
| :--- | :--- | :--- |
| `$ids` | `array` | An array of post IDs |

Scope the query to only look for specific post IDs.

Sets the `post__in` argument in `WP_Query`.

_Reference:_ [_WP\_Query - Post & Page Parameters_](https://codex.wordpress.org/Class_Reference/WP_Query#Post_.26_Page_Parameters)\_\_

### whereIdNotIn\(array $ids\)

| Parameters | Type | Description |
| :--- | :--- | :--- |
| `$ids` | `array` | An array of post IDs |

Scope the query to exclude specific post IDs.

Sets the `post__not_in` argument in `WP_Query`.

_Reference:_ [_WP\_Query - Post & Page Parameters_](https://codex.wordpress.org/Class_Reference/WP_Query#Post_.26_Page_Parameters)\_\_

### whereStatus\(\)

This method can either take an array of statuses, or multiple parameters for each status:

**Array of statuses:**

| Parameters | Type | Description |
| :--- | :--- | :--- |
| `$statuses` | `array` | An array of statuses. e.g. `publish` or `draft` |

```php
$query->whereStatus(['publish', 'draft']);
```

**Multiple parameters:**

| Parameters | Type | Description |
| :--- | :--- | :--- |
| `$status` | `string` | A status. e.g. `publish` or `draft` |
| `$status` | `string` | A status. e.g. `publish` or `draft` |
| ... | ... | ... |

```php
$query->whereStatus('publish', 'draft');
```

Scope the query to only include posts with the given status. By default WordPress will only look for published posts, so you only need to use this method if you need to get posts with other statuses.

Sets the `post_status` argument in `WP_Query`.

_Reference:_ [_WP\_Query - Status Parameters_](https://codex.wordpress.org/Class_Reference/WP_Query#Status_Parameters)\_\_

### whereMeta\($key, $value, $compare = '=', $type = null\)

| Parameters | Type | Description |
| :--- | :--- | :--- |
| `$key` | `string` | The meta key |
| `$value` | `string` | The meta value |
| `$compare` | `string` | Optional. Defaults to `=` |
| `$type` | `string` \| `null` | Optional. Defaults to `null`. Pass in a value here to define the custom field type. e.g. `numeric`. |

Scope posts that have the specified custom meta fields.

Adds an array of meta query arguments to the array of `meta_query`  arguments on `WP_Query`.

```php
'meta_query' => [
    [
        'key' => 'lead',
        'value' => 'Lorem ipsum %',
        'compare' = 'LIKE',
    ]
]
```

{% hint style="info" %}
**Note:** `meta_query` takes an **array** of meta query arguments **arrays** \(it takes an array of arrays\) 
{% endhint %}

The above example can be written like so:

```php
$query->whereMeta('lead', 'Lorem ipsum %', 'LIKE');
```

You can also add multiple meta queries.

```php
$query->whereMeta('lead', 'Lorem ipsum %', 'LIKE')
    ->whereMeta('price', [20, 100], 'BETWEEN', 'numeric');
```

This will yield the following parameters:

```php
'meta_query' => [
    [
        'key' => 'lead',
        'value' => 'Lorem ipsum %',
        'compare' = 'LIKE',
    ],
    [
        'key' => 'price',
        'value' => [20, 100],
        'compare' = 'BETWEEN',
        'type' => 'numeric',
    ]
]
```

_Reference:_ [_WP\_Query - Custom Field Parameters_](https://codex.wordpress.org/Class_Reference/WP_Query#Custom_Field_Parameters)\_\_

### whereMetaRelationshipIs\(string $relation\)

<table>
  <thead>
    <tr>
      <th style="text-align:left">Parameters</th>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>$relation</code>
      </td>
      <td style="text-align:left"><code>string</code>
      </td>
      <td style="text-align:left">
        <p>The type of relationship between meta queries.</p>
        <p></p>
        <p>Accepts <code>and</code> &amp; <code>or</code>
        </p>
      </td>
    </tr>
  </tbody>
</table>Sets the `relation` field for your meta queries, for `WP_Query`.

```php
$query->whereMeta('lead', 'Lorem ipsum %', 'LIKE')
    ->whereMeta('price', [20, 100], 'BETWEEN', 'numeric')
    ->whereMetaRelationshipIs('or');
```

This will yield the following parameters, adding the `'relation' => 'or'` to the meta query.

```php
'meta_query' => [
    'relation' => 'or',
    [
        'key' => 'lead',
        'value' => 'Lorem ipsum %',
        'compare' = 'LIKE',
    ],
    [
        'key' => 'price',
        'value' => [20, 100],
        'compare' = 'BETWEEN',
        'type' => 'numeric',
    ]
]
```

_Reference:_ [_WP\_Query - Custom Field Parameters_](https://codex.wordpress.org/Class_Reference/WP_Query#Custom_Field_Parameters)\_\_

### limit\($limit\)

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$limit` | `int` | e.g. 25 |

Set the number of results to get back from the query.

Sets the `posts_per_page` argument in `WP_Query`.

_Reference:_ [_WP\_Query - Pagination Parameters_](https://codex.wordpress.org/Class_Reference/WP_Query#Pagination_Parameters)\_\_

### offset\($offset\)

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$offset` | `int` | e.g. 50 |

Set the number of results to displace or pass over.

Sets the `offset` argument in `WP_Query`.

_Reference:_ [_WP\_Query - Pagination Parameters_](https://codex.wordpress.org/Class_Reference/WP_Query#Pagination_Parameters)\_\_

### orderBy\($orderBy, $order = 'asc'\)

| Parameter | Type | Description |
| :--- | :--- | :--- |
| `$orderBy` | `string` | e.g. 'menu\_order' |
| `$order` | `string` | Optional. Defaults to 'asc' \(ascending\) |

Sort retrieved posts by parameter, e.g. date, title, menu\_order.

Sets the `orderby` and `order` arguments in `WP_Query`.

```php
$query->orderBy('title', 'asc');
```

_Reference:_ [_WP\_Query - Order & Orderby Parameters_](https://codex.wordpress.org/Class_Reference/WP_Query#Order_.26_Orderby_Parameters)\_\_

### orderByMeta\($metaKey, $order = 'asc', $type = null\)

<table>
  <thead>
    <tr>
      <th style="text-align:left">Parameter</th>
      <th style="text-align:left">Type</th>
      <th style="text-align:left">Description</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><code>$metaKey</code>
      </td>
      <td style="text-align:left"><code>string</code>
      </td>
      <td style="text-align:left">The meta key to order by</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>$order</code>
      </td>
      <td style="text-align:left"><code>string</code>
      </td>
      <td style="text-align:left">Optional. Defaults to &apos;asc&apos; (ascending)</td>
    </tr>
    <tr>
      <td style="text-align:left"><code>$type</code>
      </td>
      <td style="text-align:left"><code>string</code>
      </td>
      <td style="text-align:left">
        <p>Optional. Defaults to null.</p>
        <p></p>
        <p>Sorting will be alphabetical for strings. When dealing with numbers, you
          can get some unexpected results (e.g. 1, 3, 34, 4, 56, 6, etc, rather than
          1, 3, 4, 6, 34, 56 as you might naturally expect).</p>
        <p></p>
        <p>Pass in &apos;numeric&apos; here if you plan on sorting numbers.</p>
      </td>
    </tr>
  </tbody>
</table>Sets the `orderby` argument for `WP_Query` to `meta_value` when ordering strings, and `meta_value_num` when ordering numbers.

_Reference:_ [_WP\_Query - Order & Orderby Parameters_](https://codex.wordpress.org/Class_Reference/WP_Query#Order_.26_Orderby_Parameters)\_\_

### as\($postClass\)

| Parameters | Type | Description |
| :--- | :--- | :--- |
| `$postClass` | `string` | The name of the post class that you want the results transformed to |

When using `WP_Query`, you get an array of `WP_Post` objects back. The query builder will instead return an array of `Rareloop\Lumberjack\Post` objects back.

You can use this method to change what object is returned from the query builder.

```php
use App\PostTypes\Event;

// Get an array of Event objects back
$query->wherePostType('event')
    ->as(Event::class)
    ->get();
```

{% hint style="info" %}
Note: When using the query builder on a post type, this conversion is done automatically. For example:

```php
// Get an array of Event object
$events = Event::get();
```
{% endhint %}

_Reference:_ [_Timber - get\_posts\(\)_](https://timber.github.io/docs/reference/timber/#get-posts)\_\_

### get\(\)

Execute the query and return a `Collection` of post objects.

```php
$posts = $query->whereMeta('price', 100, '>', 'numeric')
    ->get();
```

### first\(\)

Execute the query and return the first post object. Returns `null` if there are no results.

```php
$post = $query->whereMeta('price', 100, '>', 'numeric')
    ->first();
```

### clone\(\)

Duplicates a new instance of the query builder with all the current parameters. You can modify this query builder instance separately to the original.

This can be useful if you have similar queries with subtle differences, as it saves you duplicating the commonalities between them.

```php
$baseQuery = (new QueryBuilder)
    ->wherePostType(Announcement::getPostType())
    ->forCurrentUser(); // Some custom Query Scope, for example

$counts = [
    'all' => $baseQuery->clone()->whereStatus('publish', 'draft', 'pending')->get()->count(),
    'publish' => $baseQuery->clone()->whereStatus('publish')->get()->count(),
    'draft' => $baseQuery->clone()->whereStatus('draft')->get()->count(),
    'pending' => $baseQuery->clone()->whereStatus('pending')->get()->count(),
];
```

## Extending the query builder

The Lumberjack `QueryBuilder` class can be extended with custom functionality at runtime \(the class is "macroable"\). The following example adds a `search` method to the `QueryBuilder` class that can be used to filter results based on a keyword search:

```php
use Rareloop\Lumberjack\QueryBuilder;

// Add custom function
QueryBuilder::macro('search', function ($term) {
    $this->params['s'] = $term;
    
    return $this;
});

QueryBuilder::macro('writtenByCurrentUser', function () {
    $this->params['author'] = get_current_user_id();
    
    return $this;
});

// Use the functionality
$posts = (new QueryBuilder())
    ->search('Elephant');
    ->writtenByCurrentUser()
    ->get();
    
// Or on a post type
$posts = Post::builder()
    ->search('Elephant')
    ->writtenByCurrentUser()
    ->get();
```

