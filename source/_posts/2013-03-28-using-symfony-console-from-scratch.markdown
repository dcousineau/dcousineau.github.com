---
layout: post
title: "Using Symfony Console From Scratch"
date: 2013-03-28 22:20
comments: true
categories: 
- PHP
- Symfony
- Console
- CLI

---

CLI applications are extremely useful for many, if not most web projects. The [Symfony framework](http://symfony.com/) even goes so 
far as to include an extensible CLI console used for everything from running cache cleanup/warmup tasks, to user account management.

Many CLI scripts for web projects consist of just a static `.php` file which works fine but grow unweildy over time. Thankfully, the 
aforementioned Symfony Console component is released as a decoupled standalone that can be installed and setup easily and provide us
with structure and organization (and some powerful features).

Setup
===

The Symfony Console component can be found on GitHub at [github.com/symfony/console](https://github.com/symfony/console). While 
installing by hand is doable, I much prefer using [Composer](http://getcomposer.org/) to handle my dependencies. If you haven't 
used Composer before, I suggest following the [Getting Started](http://getcomposer.org/doc/00-intro.md#installation-nix) documentation.

To start things off, we'll need to setup, or modify, our `composer.json` file:

``` javascript composer.json
{
    "require": {
        "symfony/console": "2.1.*"
    },
    "autoload": {
        "psr-0": { "": "src/" }
    }
}
```

And then update our dependencies:

``` text
$ php composer.phar update
```

Now that we have Symfony Console and all it's dependencies downloaded and ready, we'll need to setup our actual console executable:

``` php bin/console
#!/usr/bin/env php
<?php

date_default_timezone_set('UTC');

set_time_limit(0);

(@include_once __DIR__ . '/../vendor/autoload.php') || @include_once __DIR__ . '/../../../autoload.php';

use Symfony\Component\Console\Application;

$app = new Application('My CLI Application', '0.1.0');
$app->run();
```

Afterwards a simple `chmod +x bin/console` will make it executable and you'll be ready to begin. You'll notice we placed this in a 
file named simply `console` in the `bin` directory, this is purely a stylistic choice, name it whatever you want. Giving it a test
whirl, you should see the output:

``` text
$ bin/console
My CLI Application version 0.1.0

Usage:
  [options] command [arguments]

Options:
  --help           -h Display this help message.
  --quiet          -q Do not output any message.
  --verbose        -v Increase verbosity of messages.
  --version        -V Display this application version.
  --ansi              Force ANSI output.
  --no-ansi           Disable ANSI output.
  --no-interaction -n Do not ask any interactive question.

Available commands:
  help   Displays help for a command
  list   Lists commands
```

Commands
===

Syfmony Console commands are simply classes you override and inject into the application instance. The skeleton for any given command
looks something like this:

``` php src/MyApp/Console/Command/TestCommand.php
<?php
namespace MyApp\Console\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputOption;

class TestCommand extends Command {
    protected function configure() {
        $this->setName("test")
             ->setDescription("Sample description for our command named test")
             ->setDefinition(array(
             ))
             ->setHelp(<<<EOT
The <info>test</info> command does things and stuff
EOT
             );
    }

    protected function execute(InputInterface $input, OutputInterface $output) {
        //...
    }
}
```

While we have created the class, we haven't attached it to our console application. To do this we edit our `bin/console` file
and add the follwoing code between the `new Application(...)` and the `$app->run()`:

``` php bin/console
<?php
//...
$app = new Application('My CLI Application', '0.1.0');
$app->addCommands(array(
    new MyApp\Console\Command\TestCommand(),
));
$app->run();
//...
```

When we run our `bin/console` we'll notice the output is now midly different:

``` text
$ bin/console
My CLI Application version 0.1.0

Usage:
  [options] command [arguments]

Options:
  --help           -h Display this help message.
  --quiet          -q Do not output any message.
  --verbose        -v Increase verbosity of messages.
  --version        -V Display this application version.
  --ansi              Force ANSI output.
  --no-ansi           Disable ANSI output.
  --no-interaction -n Do not ask any interactive question.

Available commands:
  help   Displays help for a command
  list   Lists commands
  test   Sample description for our command named test
```

And we can even use the help command to describe our newly created test command:
``` text
$ bin/console help test
Usage:
 test


Help:
 The test command does things and stuff
```

Output
---

Syfmony Console applications direct all their output through the `$output` variable provided. What these output objects gives
us is easy ANSI coloring utilizing XML-like tags. For example, a `$output->writeln("<info>This will be green</info> This will be white");` 
will print the ecapsulated text in the "info" style which is green. You can add and override styles to your hearts content, just
peruse the [official documentation](http://symfony.com/doc/2.0/components/console/introduction.html#coloring-the-output).

Input
---

CLI input comes in 2 flavors: *Arguments* and *Options*. In essence, Options are flags using the - or -- operators and Arguments are your 
classic space separated values. To put it visually:

``` text
$ bin/consle --option=value argument
```

Symfony Console requires you to be strict and actively provide your definition in the `configure()` method. For example, we can
setup an option and an argument:

``` php src/MyApp/Console/Command/TestCommand.php
<?php
//...
    protected function configure() {
        $this->setName("test")
             ->setDescription("Sample description for our command named test")
             ->setDefinition(array(
                new InputOption('flag', 'f', InputOption::VALUE_NONE, 'Raise a flag'),
                new InputArgument('activities', InputArgument::IS_ARRAY, 'Space-separated activities to perform', null),
             ))
             ->setHelp(<<<EOT
The <info>test</info> command does things and stuff
EOT
             );
    }
//...
```

Here we have defined a `--flag` option which has no value expected (meaning --flag=value would be technically be illegal) and an
array argument (which means an infinite series, e.g. `bin/console activities1 activities2`, etc). When we run our help command, 
you'll notice our output has changed to reflect our definition:

``` text
$ bin/console help test
Usage:
 test [-f|--flag] [activities1] ... [activitiesN]

Arguments:
 activities  Space-separated activities to perform

Options:
 --flag (-f) Raise a flag

Help:
 The test command does things and stuff
```

Now when our command is actually running we can check whether or not a flag was issued with a simple `$input->getOption('flag');`
as well as we can get all of our activities with a `$input->getArgument('activities');`.

There are many modifiers to be used with both Arguments and Options. For example, you can have required Arguments, Options that
require a value to be set, so on and so forth. You can view the [official documentation](http://symfony.com/doc/2.0/components/console/introduction.html#using-command-arguments)
to get an idea of the power available to you.

Integrating With Your Application
===

The bulk of power you gain from writing CLI apps comes from the ability to share code and resources with your main application.

Symfony Console does not provide an "out of the box" way to integrate with anything, however it is fairly trivial to roll in support
on your own. 

For example, say you were building a CLI app to complement your Silex web application. You have configured services and database connections
you would like to share via the Silex DI container/application object.

A best practice would be to create a base command class that can accept your DI container and have your commands inherit from it:

``` php src/MyApp/Console/Command/ContainerAwareCommand.php
<?php
namespace MyApp\Console\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

use Silex\Application

abstract class ContainerAwareCommand extends Command {
    /**
     * @var Silex\Application
     */
    protected $app;
    
    public function __construct(Application $app, $name = null) {
        parent::__construct($name);
        $this->app = $app;
    }
}
```

And update your `bin/console` file to include the PHP file responsible for creating and configuring the Application object and pass
said Application object into the commands:

``` php bin/console
#!/usr/bin/env php
<?php

date_default_timezone_set('UTC');

set_time_limit(0);

(@include_once __DIR__ . '/../vendor/autoload.php') || @include_once __DIR__ . '/../../../autoload.php';

//Provides $app, which is an instance of Silex\Application
require_once 'path/to/app/setup.php';

use Symfony\Component\Console\Application;

$cli_app = new Application('My CLI Application', '0.1.0');

$cli_app->addCommands(array(
    new MyApp\Console\Command\TestCommand($app),
));

$cli_app->run();
```

Another path is to contemplate overloading the actualy Symfony Console Application class. Doing this affords you the ability to
override the help message as well as any global option definitions:

``` php src/MyApp/Console/Application.php
<?php
namespace MyApp\Console;

use MyApp\Console\Command;
use Symfony\Component\Console\Application as BaseApplication;

class Application extends BaseApplication {
    /**
     * @var Silex\Application;
     */
    protected $app;
    
    public function __construct(Application $app) {
        $this->app = $app;
        parent::__construct('My CLI Application', '0.1.0');
    }
    
    public function getDefaultCommands() {
        $commands = parent::getDefaultCommands();

        $commands[] = new Command\TestCommand($this->app);

        return $commands;
    }
}
```

Which has another added benefit of keeping your `bin/console` file simple:

``` php bin/console
#!/usr/bin/env php
<?php

date_default_timezone_set('UTC');

set_time_limit(0);

(@include_once __DIR__ . '/../vendor/autoload.php') || @include_once __DIR__ . '/../../../autoload.php';
require_once 'path/to/app/setup.php';

use MyApp\Console\Application;

$app = new Application($app);
$app->run();
```

Environment
===

One thing to be aware of, most web applications these days are aware of environments when it comes to what configurations to use.
The Symfony Console Application object allows you to manually create your `$input` and `$output` options, which allows us to 
perform a neat little trick and gain early access to the option parsing.

If you look at lines 19 and 20 below you can see us checking the `--env` option to see if it's set, if not using a default environment
variable `MYAPP_ENV` from the shell or our fallback 'dev' environment designation. We then just go ahead and forward our newly
created `$input` and `$output` objects through our `run()` method as shown on line 23.

``` php bin/console
#!/usr/bin/env php
<?php

date_default_timezone_set('UTC');

set_time_limit(0);

(@include_once __DIR__ . '/../vendor/autoload.php') || @include_once __DIR__ . '/../../../autoload.php';

//Provides $app, which is an instance of Silex\Application
require_once 'path/to/app/setup.php';

use MyApp\Console\Application;
use Symfony\Component\Console\Input\ArgvInput;
use Symfony\Component\Console\Output\ConsoleOutput;

$input = new ArgvInput();
$output = new ConsoleOutput();

//Determine Environment
$env = $input->getParameterOption(array('--env', '-e'), getenv('MYAPP_ENV') ?: 'prod');
$app['environment'] = $env;

$cli_app_ = new Application('My CLI Application', '0.1.0');
$cli_app_->run($input, $output);
```

As an added bonus, this same concept can be applied to creating your own Output class. The Composer project actively makes use
of this to extend in the ability to do overwrites in the console (the technique used for it's fancy progress indication).