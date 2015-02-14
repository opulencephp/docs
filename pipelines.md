# Pipelines

## Table of Contents
1. [Introduction](#introduction)
2. [Using Closures](#using-closures)
3. [Using Objects](#using-objects)
  1. [Using Classes](#using-classes)
4. [Specifying a Callback](#specifying-a-callback)

<h2 id="introduction">Introduction</h2>
In computer science, a pipeline refers to a series of stages where each stage's input is the previous one's output.  It can be used to implement a decorator, which adds functionality to an object from an outside object.  In RDev, this is especially useful for route middleware.  Pipelines can accept a list of `Closure` pipes or class names and methods to run as pipes.  You can even mix and match `Closure` and class/method pipes.
  
<h2 id="using-callbacks">Using Callbacks</h2>
`Closure` pipes must accept the input as their first parameter and the next pipe in the pipeline as the second parameter.  Let's take a look at a simple example:

```php
use RDev\IoC;
use RDev\Pipelines;

$container = new IoC\Container();
$pipes = [
    function($input, $next)
    {
        $input .= "-pipe1";
        
        return $next($input);
    },
    function($input, $next)
    {
        $input .= "-pipe2";
        
        return $next($input);
    }
];
$pipeline = new Pipelines\Pipeline($container, $pipes);
echo $pipeline->send("foo");
```

This will output:

```
foo-pipe1-pipe2
```

<h2 id="using-objects">Using Objects</h2>
`Pipeline` can also accept an array of objects and a method to call on those classes.

> **Note:** The method MUST accept two parameters - the output from the previous stage and the next pipe in the pipeline.

```php
use RDev\IoC;
use RDev\Pipelines;

interface IMyPipe
{
    public function filter($input, \Closure $next);
}

class PipeA implements IMyPipe
{
    public function filter($input, \Closure $next)
    {
        $input .= "-pipeA";
        
        return $next($input);
    }
}

class PipeB implements IMyPipe
{
    public function filter($input, \Closure $next)
    {
        $input .= "-pipeB";
        
        return $next($input);
    }
}

$container = new IoC\Container();
$pipes = [new PipeA(), new PipeB()];
// We must pass in the name of the method to call ("filter")
$pipeline = new Pipelines\Pipeline($container, $pipes, "filter");
echo $pipeline->send("foo");
```

This will output:

```
foo-pipeA-pipeB
```

<h4 id="using-classes">Using Classes</h4>
Pipe class names are also supported.  They will automatically be resolved using the IoC container:
 
```php
$pipes = ["PipeA", "PipeB"];
$pipeline = new Pipelines\Pipeline($container, $pipes, "filter");
```

This will output:

```
foo-pipeA-pipeB
```

<h2 id="specifying-a-callback">Specifying a Callback</h2>
To run a callback at the very end of the pipeline, pass in a `Closure` that accepts the pipeline's output as a parameter:

```php
use RDev\IoC;
use RDev\Pipelines;

$container = new IoC\Container();
$pipes = [
    function($input, $next)
    {
        $input .= "-pipe1";
        
        return $next($input);
    },
    function($input, $next)
    {
        $input .= "-pipe2";
        
        return $next($input);
    }
];
$pipeline = new Pipelines\Pipeline($container, $pipes);
$callback = function($pipelineOutput)
{
    return strtoupper($pipelineOutput);
};
echo $pipeline->send("foo", $callback);
```

This will output:

```
FOO-PIPE1-PIPE2
```