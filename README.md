# Queueable actions in Laravel

[![Latest Version on Packagist](https://img.shields.io/packagist/v/spatie/laravel-queueable-action.svg?style=flat-square)](https://packagist.org/packages/spatie/laravel-queueable-action)
![GitHub Workflow Status](https://img.shields.io/github/workflow/status/spatie/laravel-queueable-action/run-tests?label=tests)
![Check & fix styling](https://github.com/spatie/laravel-queueable-action/workflows/Check%20&%20fix%20styling/badge.svg)
[![Total Downloads](https://img.shields.io/packagist/dt/spatie/laravel-queueable-action.svg?style=flat-square)](https://packagist.org/packages/spatie/laravel-queueable-action)

Actions are a way of structuring your business logic in Laravel. 
This package adds easy support to make them queueable.

```php
$myAction->onQueue()->execute();
```

You can specify a queue name.

```php
$myAction->onQueue('my-favorite-queue')->execute();
```

## Support us

[<img src="https://github-ads.s3.eu-central-1.amazonaws.com/laravel-queueable-action.jpg?t=1" width="419px" />](https://spatie.be/github-ad-click/laravel-queueable-action)

We invest a lot of resources into creating [best in class open source packages](https://spatie.be/open-source). You can support us by [buying one of our paid products](https://spatie.be/open-source/support-us).

We highly appreciate you sending us a postcard from your hometown, mentioning which of our package(s) you are using. You'll find our address on [our contact page](https://spatie.be/about-us). We publish all received postcards on [our virtual postcard wall](https://spatie.be/open-source/postcards).

## Installation

You can install the package via composer:

```bash
composer require spatie/laravel-queueable-action
```

## Usage

If you want to know about the reasoning behind actions and their asynchronous usage, 
you should read the dedicated blog post: [https://stitcher.io/blog/laravel-queueable-actions](https://stitcher.io/blog/laravel-queueable-actions).

You can use the following Artisan command to generate queueable and synchronous action classes on the fly.

```
php artisan make:action MyAction [--sync]
```

Here's an example of queueable actions in use:

``` php
class MyAction
{
    use QueueableAction;

    public function __construct(
        OtherAction $otherAction,
        ServiceFromTheContainer $service
    ) {
        // Constructor arguments can come from the container.
    
        $this->otherAction = $otherAction;
        $this->service = $service;
    }

    public function execute(
        MyModel $model,
        RequestData $requestData
    ) {
        // The business logic goes here, this can be executed in an async job.
    }
}
```

```php
class MyController
{
    public function store(
        MyRequest $request,
        MyModel $model,
        MyAction $action
    ) {
        $requestData = RequestData::fromRequest($myRequest);
        
        // Execute the action on the queue:
        $action->onQueue()->execute($model, $requestData);
        
        // Or right now:
        $action->execute($model, $requestData);
    }
}
```

### Testing queued actions

The package provides some test assertions in the `Spatie\QueueableAction\Testing\QueueableActionFake` class. You can use them in a PhpUnit test like this:

```php

/** @test */
public function it_queues_an_action()
{
    Queue::fake();
    
    (new DoSomethingAction)->onQueue()->execute();

    QueueableActionFake::assertPushed(DoSomethingAction::class);
}
```

Don't forget to use `Queue::fake()` to mock Laravel's queues before using the `QueueableActionFake` assertions.

The following assertions are available:

```php
QueueableActionFake::assertPushed(string $actionClass);
QueueableActionFake::assertPushedTimes(string $actionClass, int $times = 1);
QueueableActionFake::assertNotPushed(string $actionClass);
```

Feel free to send a PR if you feel any of the other `QueueFake` assertions are missing.

### Chaining actions

You can chain actions by wrapping them in the `ActionJob`. 

Here's an example of two actions with the same arguments:

```php
use Spatie\QueueableAction\ActionJob;

$args = [$userId, $data];

app(MyAction::class)
    ->onQueue()
    ->execute(...$args)
    ->chain([
        new ActionJob(AnotherAction::class, $args),
    ]);
```

The `ActionJob` takes the action class *or* instance as the first argument followed by an array of the action's own arguments.

### Custom Tags

If you want to change what tags show up in Horizon for your custom actions you can override the `tags()` function.

``` php
class CustomTagsAction
{
    use QueueableAction;

    // ...

    public function tags() {
        return ['action', 'custom_tags'];
    }
}
```

### Job Middleware

Middleware where action job passes through can be added by overriding the `middleware()` function.

``` php
class CustomTagsAction
{
    use QueueableAction;

    // ...

    public function middleware() {
        return [new RateLimited()];
    }
}
```

### What is the difference between actions and jobs?

In short: constructor injection allows for much more flexibility. 
You can read an in-depth explanation here: [https://stitcher.io/blog/laravel-queueable-actions](https://stitcher.io/blog/laravel-queueable-actions#what's-the-difference-with-jobs?!?).

### Testing the package

``` bash
composer test
```

### Changelog

Please see [CHANGELOG](CHANGELOG.md) for more information on what has changed recently.

## Contributing

Please see [CONTRIBUTING](CONTRIBUTING.md) for details.

### Security

If you discover any security related issues, please email freek@spatie.be instead of using the issue tracker.

## Credits

- [Brent Roose](https://github.com/brendt)
- [Alex Vanderbist](https://github.com/alexvanderbist)
- [Sebastian De Deyne](https://github.com/sebdedeyne)
- [Freek Van der Herten](https://github.com/freekmurze)
- [All Contributors](../../contributors)

## License

The MIT License (MIT). Please see [License File](LICENSE.md) for more information.
