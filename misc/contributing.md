# Contributing

Contributions are **welcome** and will be fully **credited**.

We accept contributions via Pull Requests on [Github](https://github.com/Rareloop/lumberjack).

## Pull Requests

* [**PSR-2 Coding Standard**](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-2-coding-style-guide.md) - The easiest way to apply the conventions is to install [PHP Code Sniffer](http://pear.php.net/package/PHP_CodeSniffer).
* **Add tests** - Your patch likely will not be accepted if it doesn't have tests.
* **Create feature branches** - Don't ask us to pull from your master branch.
* **One pull request per feature or fix** - If you want to do more than one thing, send multiple pull requests.
* **Send coherent history** - Make sure each individual commit in your pull request is meaningful. If you had to make multiple intermediate commits while developing, please [squash them](http://www.git-scm.com/book/en/v2/Git-Tools-Rewriting-History#Changing-Multiple-Commit-Messages) before submitting. We follow the advice of Chris Beams on [How to write good commit messages](https://chris.beams.io/posts/git-commit/).

## Running Tests

```bash
$ vendor/bin/phpunit
```

## Linting against PSR2 Coding Standards

```bash
$ vendor/bin/phpcs --standard=PSR2 ./src
```
