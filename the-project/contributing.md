---
description: You have just found a bug?
---

# Contributing

First of all, **thank you** for contributing.

Bugs or feature requests can be posted online on the GitHub issues section of the project.

Few rules to ease code reviews and merges:

* You MUST follow the [PSR-12](https://www.php-fig.org/psr/psr-12/) for coding standards.
* You MUST run the test suite (see below).
* You MUST write (or update) unit tests when bugs are fixed or features are added.
* You SHOULD write documentation.
* You MAY follow the [PSR-5](https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc.md) and [PSR-19](https://github.com/php-fig/fig-standards/blob/master/proposed/phpdoc-tags.md).

We use the following branching workflow:

* Each minor version has a dedicated branch (e.g. v1.1, v1.2, v2.0, v2.1…)
* The default branch is set to the last minor version (e.g. v2.1).

To contribute use [Pull Requests](https://help.github.com/articles/using-pull-requests), please, write commit messages that make sense, and rebase your branch before submitting your PR.

Your PR **should NOT** be submitted to the master branch but to the last minor version branch or to another minor version in case of bug fix.

### Run test suite

* install composer: `curl -s http://getcomposer.org/installer | php`
* install dependencies: `php composer.phar install`
* run tests: `vendor/bin/simple-phpunit`
