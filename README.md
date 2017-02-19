<p align="center">
    <img src="./assets/caporal.png" width="500" height="177">
</p>

[![Travis](https://img.shields.io/travis/mattallty/Caporal.js.svg)](https://travis-ci.org/mattallty/Caporal.js)
[![Codacy grade](https://img.shields.io/codacy/grade/6e5459fd36e341d1bd27414cf6b06e5c.svg)](https://www.codacy.com/app/matthiasetienne/Caporal-js/dashboard)
[![Codacy coverage](https://img.shields.io/codacy/coverage/6e5459fd36e341d1bd27414cf6b06e5c.svg)]()
[![npm](https://img.shields.io/npm/v/caporal.svg)](https://www.npmjs.com/package/caporal)
[![npm](https://img.shields.io/npm/dm/caporal.svg)](https://www.npmjs.com/package/caporal)
[![npm](https://img.shields.io/npm/l/caporal.svg)](https://www.npmjs.com/package/caporal)
[![David](https://img.shields.io/david/mattallty/Caporal.js.svg)](https://david-dm.org/mattallty/Caporal.js)
[![GitHub stars](https://img.shields.io/github/stars/mattallty/Caporal.js.svg?style=social&label=Star)](https://github.com/mattallty/Caporal.js/stargazers)
[![GitHub forks](https://img.shields.io/github/forks/mattallty/Caporal.js.svg?style=social&label=Fork)](https://github.com/mattallty/Caporal.js/network)

# Caporal

> A full-featured framework for building command line applications (cli) with node.js,
> including help generation, colored output, verbosity control, custom logger, coercion
> and casting, typos suggestions, and auto-complete for bash/zsh/fish.
 

## Glossary

* **Program**: a cli app that you can build using Caporal
* **Command**: a command within your program. A program may have multiple commands.
* **Argument**: a command may have one or more arguments passed after the command. 
* **Options**: a command may have one or more options passed after (or before) arguments.

Angled brackets (e.g. `<item>`) indicate required input. Square brackets (e.g. `[env]`) indicate optional input.

## Examples

```javascript
#!/usr/bin/env node
const prog = require('caporal');
prog
  .version('1.0.0')
  // you specify arguments in .command()
  // 'app' is required, 'env' is optional
  .command('deploy', 'Deploy an application') 
  .argument('<app>', 'App to deploy', /^myapp|their-app$/)
  .argument('[env]', 'Environment to deploy on', /^dev|staging|prod$/, 'local')
  // you specify options using .option()
  // if --tail is passed, its value is required
  .option('--tail <lines>', 'Tail <lines> lines of logs after deploy', prog.INT) 
  .action(function(args, options, logger) {
    // args and options are objects
    // args = {"app": "myapp", "env": "production"}
    // options = {"tail" : 100}
  });

// ./myprog deploy myapp production --tail 100
```

### Variadic arguments

You can use `...` to indicate variadic arguments. In that case, the resulted value will be an array.

```javascript
#!/usr/bin/env node
const prog = require('caporal');
prog
  .version('1.0.0')
  // you specify arguments in .command()
  // 'app' and 'env' are required, and you can pass additional environments through not required
  .command('deploy <app> <env> [other-env...]', 'Deploy an application to one or more environments') 
  .action(function(args, options, logger) {
    console.log(args);
    // {
    //   "app": "myapp", 
    //   "env": "production",
    //   "otherEnv": ["google", "azure"]
    // }
  });

// ./myprog deploy myapp production aws google azure
```

### Simple program (single command)

For a very simple program with just one command, you can omit the .command() call:

```javascript
#!/usr/bin/env node
const prog = require('caporal');
prog
  .version('1.0.0')
  .description('A simple program that says "biiiip"')
  .action(function(args, options, logger) {
    logger.info("biiiip")
  });
```

## API

#### `require('caporal)`

Returns a `Program` instance.

### Program API

#### `.version(version) : Program`

Set the version of your program. You may want to use your `package.json` version to fill it:

```javascript
const myProgVersion = require('./package.json').version;
const prog = require('caporal');
prog
  .version(myProgVersion)
// [...]
```

Your program will then automaticaly handle `-V` and `--version` options:

    matt@mb:~$ ./my-program --version
    1.0.0

#### `.command(name, description) -> Command`

Set up a new command with name and description. Multiple commands can be added to one program.
Returns a {Command}.

```javascript
const prog = require('caporal');
prog
  .version('1.0.0')
  // one command
  .command('walk', 'Make the player walk')
  .action((args, options, logger) => { logger.log("I'm walking !")}) // you must attach an action for your command
  // a second command
  .command('run', 'Make the player run')
  .action((args, options, logger) => { logger.log("I'm running !")})
  // a command may have multiple words
  .command('cook pizza', 'Make the player cook a pizza')
  .argument('<kind>', 'Kind of pizza')
  .action((args, options, logger) => { logger.log("I'm cooking a pizza !")})
// [...]
```

#### `.logger([logger]) -> Program | winston`

Get or set the logger instance. Without argument, it returns the logger instance (*winston* by default).
With the *logger* argument, it sets a new logger.

### Command API

#### .argument(synopsis, description, [validator, [defaultValue]]) -> *Command*

Add an argument to the command. Can be called multiple times to add several arguments.

* **synopsis** (*String*): something like `<my-required-arg>` or `<my-optional-arg>`
* **description** (*String*): argument description
* **validator** (*Caporal Flag | Function | RegExp*): optional validator, see [Coercion and casting ](#coercion-and-casting)
* **defaultValue** (*): optional default value

#### .option(synopsis, description, [validator, [defaultValue, [required]]) -> *Command*

Add an option to the command. Can be called multiple times to add several options.

* **synopsis** (*String*): You can pass short or long notation here, or both. See examples.
* **description** (*String*): option description
* **validator** (*Caporal Flag | Function | RegExp*): optional validator, see [Coercion and casting ](#coercion-and-casting)
* **defaultValue** (*): optional default value
* **required** (*Bool*): Is the option itself required ? Default to `false`

#### .action(action) -> *Command*

Define the action, e.g a *Function*, for the current command. The *action* callback will be called with
3 arguments: *args*, *options*, and *logger*.

#### .alias(alias) -> *Command*

Define an alias for the current command. A command can only have one alias.

## Logging

Inside your action(), use the logger argument (third one) to log informations.

```javascript
#!/usr/bin/env node
const prog = require('caporal');
prog
  .version('1.0.0')
  .command('deploy <app> [env]', 'Deploy an application') 
  .option('--restart', 'Make the application restart after deploy') 
  .action((args, options, logger) => {
    // Available methods: 
    // - logger.debug()
    // - logger.info() or logger.log()
    // - logger.warn()
    // - logger.error()
    logger.info("Application deployed !");
  });
```

### Logging levels

The default logging level is 'info'. The predifined options can be used to change the logging level:

* `-v, --verbose`: Set the logging level to 'debug' so debug() logs will be output.
* `--quiet, --silent`: Set the logging level to 'warn' so only warn() and error() logs will be output. 

### Custom logger

Caporal uses `winston` for logging. You can provide your own winston-compatible logger using `.logger()`
 the following way:

```javascript
#!/usr/bin/env node
const prog = require('caporal');
const myLogger = require('/path/to/my/logger.js');
prog
  .version('1.0.0')
  .logger(myLogger)
  .command('foo', 'Foo command description') 
  .action((args, options, logger) => {
    logger.info("Foo !!");
  });

```

* `-v, --verbose`: Set the logging level to 'debug' so debug() logs will be output.
* `--quiet, --silent`: Set the logging level to 'warn' so only warn() and error() logs will be output. 


## Coercion and casting

You can apply coercion and casting using either:
 * Caporal flags
 * Functions
 * RegExp

### Using Caporal flags

* `INT` (or `INTEGER`): Check option looks like an int and cast it with `parseInt()`  
* `FLOAT`: Will Check option looks like a float and cast it with `parseFloat()`
* `BOOL` (or `BOOLEAN`): Check for string like `0`, `1`, `true`, `false`, `on`, `off` and cast it
* `LIST` (or `ARRAY`): Transform input to array by spliting it on comma  
* `REPEATABLE`: Make the option repeatable, eg `./mycli -f foo -f bar -f joe`
* `REQUIRED`: Make the option required in the command line

```javascript
#!/usr/bin/env node
const prog = require('caporal');
prog
  .version('1.0.0')
  .command('order pizza')
  .option('--number <num>', 'Number of pizza', prog.INT, 1)
  .option('--kind <kind>', 'Kind of pizza', /^margherita|hawaiian$/)
  .option('--discount <amount>', 'Discount offer', prog.FLOAT)
  .option('--add-ingredients <ingredients>', prog.LIST)
  .action(function(args, options) {
    // options.kind = 'margherita'
    // options.number = 1
    // options.addIngredients = ['pepperoni', 'onion']
    // options.discount = 1.25
  });

// ./myprog order pizza --kind margherita --discount=1.25 --add-ingredients=pepperoni,onion
```

```javascript
#!/usr/bin/env node
const prog = require('caporal');
prog
  .version('1.0.0')
  .command('concat') // concat files
  .option('-f <file>', 'File to concat', prog.REPEATABLE)
  .action(function(args, options) {

  });

// Usage:
// ./myprog concat -f file1.txt -f file2.txt -f file3.txt
```

### Using a function

Using this method, you can check and cast user input. Make the check fail by throwing an error,
and cast input by returning a new value from your function. 


```javascript
#!/usr/bin/env node
const prog = require('caporal');
prog
  .version('1.0.0')
  .command('order pizza')
  .option('--kind <kind>', 'Kind of pizza', function(opt) {
    if (['margherita', 'hawaiian'].includes(opt) === false) {
      throw new Error("You can only order margherita or hawaiian pizza!");
    }
    return opt.toUpperCase();
  })
  .action(function(args, options) {
    // options = { "kind" : "MARGHERITA" }
  });

// ./myprog order pizza --kind margherita
```

### Using RegExp

Simply pass a RegExp object in the third argument to test against it.
**Note**: It is not possible to cast user input with this method, only check it, 
so it's basicaly only interesting for strings.

```javascript
#!/usr/bin/env node
const prog = require('caporal');
prog
  .version('1.0.0')
  .command('order pizza') // concat files
  .option('--kind <kind>', 'Kind of pizza', /^margherita|hawaiian$/)
  .action(function(args, options) {
    
  });

// ./myprog order pizza --kind margherita
```

## Colors

By default, Caporal will output colors for help and errors. 
This behaviour can be disabled by passing `--no-colors`.


## Auto-generated help

Caporal automaticaly generates help/usage instructions for you.
Help can be displayed using `-h` or `--help` options, or with the default `help` command.
 

## Typo suggestions

Caporal will automaticaly make suggestions for option typos.
If set up `--foot` you pass `--foo`, Caporal will suggest you `--foot`.


## Credits

Caporal is strongly inspired by [commander.js](https://github.com/tj/commander.js) and [Symfony Console](http://symfony.com/doc/current/components/console.html).
Caporal make use of the following npm packages:
* [chalk](https://www.npmjs.com/package/chalk) for colors
* [cli-table2](https://www.npmjs.com/package/cli-table2) for cli tables
* [fast-levenshtein](https://www.npmjs.com/package/fast-levenshtein) for suggestions
* [minimist](https://www.npmjs.com/package/minimist) for argument parsing
* [prettyjson](https://www.npmjs.com/package/prettyjson) to output json 
* [winston](https://www.npmjs.com/package/winston) for logging 
