---
description: You have just found a bug?
---

# Contributing

First of all, **thank you** for contributing.

Bugs or feature requests can be posted online on the GitHub issues section of the project.

Few rules to ease code reviews and merges:

* You MUST follow the [PSR-12](https://www.php-fig.org/psr/psr-12/) for coding standards.
* You MUST use the [PSR-20](https://www.php-fig.org/psr/psr-20/) to get the time.
* You MUST run the test suite (see below).
* You MUST write (or update) tests when bugs are fixed or features are added.
* You SHOULD write documentation.
* You MAY follow the [PSR-5](https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc.md) and [PSR-19](https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc-tags.md).

We use the following branching workflow:

* Each minor version has a dedicated branch (e.g. `1.1.x`, `1.2.x`, `2.0.x`, `2.1.x`…)
* The default branch is set to the last minor version (e.g. v2.1.x).
* Please select the correct branch when submitting a PR
  * If it is a bug fix, please use the version first major release (`1.0.x`, `2.0.x`, `3.0.x`...)
  * If it is a new feature, please use the last minor release

To contribute use [Pull Requests](https://help.github.com/articles/using-pull-requests), please, write commit messages that make sense, and rebase your branch before submitting your PR.

### Run test suite

* install composer: `curl -s http://getcomposer.org/installer | php`
* install dependencies: `php composer.phar install`
* run tests: `vendor/bin/phpunit`
