# Pipelines

## Table of Contents
1. [Introduction](#introduction)
2. [Using Closures](#using-closures)
3. [Using Objects](#using-objects)
  1. [Using Classes](#using-classes)
4. [Specifying a Callback](#specifying-a-callback)

<h2 id="introduction">Introduction</h2>
In computer science, a pipeline refers to a series of stages where each stage's input is the previous one's output.  It can be used to implement a decorator, which adds functionality to an object from an outside object.  In Opulence, this is especially useful for [HTTP middleware](http-middleware).  Pipelines can accept a list of:

* `Closures`
* Objects and a method to run on them

... to act as pipeline stages.  You can even mix and match various types of stages.

> **Note:** If a stage does not specifically call the `$next` closure, no stages after it in the pipeline will be run.

<h2 id="using-closures">Using Closures</h2>
`Closure` stages must accept the input as their first parameter and the next pipe in the pipeline as the second parameter.  Let's take a look at a simple example:

```php
use Opulence\Pipelines\Pipeline;

$stages = [
    function ($input, $next) {
        $input .= '-pipe1';

        return $next($input);
    },
    function ($input, $next) {
        $input .= '-pipe2';

        return $next($input);
    }
];
echo (new Pipeline)
    ->send('foo')
    ->through($stages)
    ->execute();
```

This will output:

```
foo-pipe1-pipe2
```

<h2 id="using-objects">Using Objects</h2>
`Pipeline` can also accept an array of objects and a method to call on those objects.

> **Note:** The method MUST accept two parameters - the output from the previous stage and the next pipe in the pipeline.

```php
use Closure;
use Opulence\Pipelines\Pipeline;

interface IMyPipe
{
    public function filter($input, Closure $next);
}

class PipeA implements IMyPipe
{
    public function filter($input, Closure $next)
    {
        $input .= '-pipeA';

        return $next($input);
    }
}

class PipeB implements IMyPipe
{
    public function filter($input, Closure $next)
    {
        $input .= '-pipeB';

        return $next($input);
    }
}

$stages = [new PipeA(), new PipeB()];
// We must pass in the name of the method to call ("filter")
echo (new Pipeline)
    ->send('foo')
    ->through($stages, 'filter')
    ->execute();
```

This will output:

```
foo-pipeA-pipeB
```

<h4 id="using-classes">Using Classes</h4>
If you use the dependency injection container, you can also use class names as stages:

```php
$classes = ['PipeA', 'PipeB'];
$stages = [];

foreach ($classes as $class) {
    $stages[] = $container->resolve($class);
}

echo (new Pipeline)
    ->send('foo')
    ->through($stages, 'filter')
    ->execute();
```

This will output:

```
foo-pipeA-pipeB
```

<h2 id="specifying-a-callback">Specifying a Callback</h2>
To run a callback at the very end of the pipeline, pass in a `Closure` that accepts the pipeline's output as a parameter:

```php
use Opulence\Pipelines\Pipeline;

$stages = [
    function ($input, $next) {
        $input .= '-pipe1';

        return $next($input);
    },
    function ($input, $next) {
        $input .= '-pipe2';

        return $next($input);
    }
];
echo (new Pipeline)
    ->send('foo')
    ->through($stages)
    ->then(function ($output) {
        return strtoupper($output);
    })
    ->execute();
```

This will output:

```
FOO-PIPE1-PIPE2
```
