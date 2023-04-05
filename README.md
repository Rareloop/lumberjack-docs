---
description: Supercharge your WordPress Development
---

# Introduction

Lumberjack is a powerful MVC framework for the modern WordPress developer. Write better, more expressive and easier to maintain code.

## Who is Lumberjack for?

Coming from another PHP framework such as Laravel, have experience using Timber with WordPress but want more, or are just getting started with modern WordPress? Then Lumberjack is for you.

Use as little or as much as you need, it's beginner friendly!

## Beautiful code. Easy to maintain

Object orientated and MVC patterns help keep your code structured and DRY.

**index.php**

```php
class IndexController
{
    public function handle()
    {
        $context = Timber::get_context();
        $context['posts'] = Post::whereStatus('publish')
            ->limit(5)
            ->get();

        return new TimberResponse('index.twig', $context);
    }
}
```

**index.twig**

```markup
{% raw %}
{% if posts is not empty %}
    <h4>Recent Articles</h4>
    <ul>
        {% for post in posts %}
            <li class="article">
                <h3>{{ $post->title }}</h3>
                {{ $post->preview }}
                <a href="{{ $post->link }}">Read the full story</a>
            </li>
        {% endfor %}
    </ul>
{% endif %}
{% endraw %}
```

## You're in good company

> Lumberjack is the deluxe version of what Modern WordPress should look like today. The team has done a great job of making it easy to build complicated custom applications while taking advantage of the best parts of WordPress.
>
> * _**Jared Novack - Timber creator**_

Standing on the shoulders of giants, Lumberjack proudly builds on the great work of other open source WordPress projects:

* [Timber](https://www.upstatement.com/timber/)
* [Bedrock](https://roots.io/bedrock/)

## Made by [Rareloop](https://rareloop.com)

We're a Digital Product Studio based in Southampton (UK) with many years experience working on modern WordPress websites. We design and build digital products for a range of clients, take a look at what else we can do.

[Find out more](https://www.rareloop.com/)
