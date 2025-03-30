---
layout: post
---
# Getting the basics of Logging in Python

In this post I will go through the basic components and concepts of the build-in logging module in python. First, let's look at this very basic example:

```python
import logging

logging.warning(msg="you gonna learn something today!")
# WARNING:root:you gonna learn something today!
```

With this, we have used the `root`-logger, to log a message on the warning level.

## Logging Components

I will quickly explain the 3 basic components of the Logging module. Other, more advanced concepts like Filters, Records, Arguments and Extras will be explained in another article to come.
See this article as a supplement to the [official documentation](https://docs.python.org/3/library/logging.html), extended with my own insights and learnings.

### Logger

When using the logging functions available directly in the logging module, like in the example above, you are using the `root`-logger.
This will work in a very simple script. As soon as you are dealing with multiple modules, a more fine-grained interface is needed to configure what message should go where and keeping track of the messages origins afterwards. A Key Part of this is, to use different Loggers. Loggers are defined by their names in a one-to-one relationship. This means, that you can get any Logger from anywhere by calling its name. To do that, you call the `logging.getLogger(name=__name__)`-function. If no logger of that name exists, it will be created. In fact, this is the recommended way to create a logger in the [basic tutorial](https://docs.python.org/3/library/logging.html#logger-objects) and the [advanced usage](https://docs.python.org/3/howto/logging.html#advanced-logging-tutorial).

### Handler

The logging-Handler defines what to do with the log-Messages. These could be send to STDOUT/STDERR in case of a StreamHandler. Or a File in Case of a FileHandler. Another Handler maybe the HTTPHandler. Loggers and Handlers are in a Many-to-Many Relation. A Logger may has several Handlers, meaning that the Log-Message ends up in multiple places. Also a Handler may get added to multiple Loggers. The last case may have practical merit, but could probably be better solved using Ancestry and Propagation. More on that later.

### Formatter

The Formatter defines how the Message is displayed. In the example above, you can see the Level and the Name of the Logger within the created output.
[Here is a list of available Attributes](https://docs.python.org/3/library/logging.html#logrecord-attributes)


### Bringing it together

```python
import logging
import sys

logger = logging.getLogger(name=__name__)

if __name__=="__main__":
    handler = logging.StreamHandler(stream=sys.stderr) #  stderr is the default
    handler.formatter = logging.Formatter(fmt="%(name)s %(lineno)d %(message)s")

    logger.addHandler(handler)

    logger.warning("you have been warned!")
# __main__ 10 you have been warned!
```

## Logging Concepts

Now that you know the logging components, you can already effectively use the Logging module. Getting the most of it and understanding further techniques requires the knowledge of the next three concepts, that can play hand in hand when mastering the python logging module.

### Level

This one is rather simple, due to its [linear nature](https://docs.python.org/3/library/logging.html#logging-levels). The Log-Level is an Integer, that is attached to every Logger, Handler and Message. If the Level of the Message is equal or higher than the Level of the Logger and Handler, it will be handled. 

### Ancestry

Every Logger (except the Root Logger) has a Parent. You define the Parent using a `.`-syntax for the name, where the parent(s) come before the `.`. With this, you can construct a logger hierarchy. Let's consider the following example:

```python
import logging


logging.getLogger("a")
logging.getLogger("a.b").parent
# <Logger a (WARNING)>
logging.getLogger("a.b.c").parent
# <Logger a.b (WARNING)>
```

We created a Logger named `a`. After that we created a Logger `b` with `a` as its parent. After that, we created a Logger `c` with `b` as its Parent. We can make use of this by using `__name__` as the logger name. If we are inside a module and not `__main__`, the logger from the folder one hierarchy level above will become the ancestor. This will make it easy to configure the logging on a module basis, when executed.

> if initializing a logger like this:
>
> `logger = logging.getLogger(__name__)` 
> 
> it becomes easy to configure it

### Propagation

The advantage of Ancestry is, that certain properties of loggers can be 'inherited' through loggers due to propagation. I am not talking about real Inheritance here, like meant in OOP, but rather an implicit one. Every Message will be handled by the Handlers of the Logger and afterwards be propagated to the Parent, where this loop continues, until the Root Logger is reached. A [flow chart of this is displayed in the official documentation](https://docs.python.org/3/howto/logging.html#logging-flow) This Propagation can be deactivated at any Logger. Consider the following Example:

```python
import logging


fh = logging.FileHandler(filename="debug.log")
fh.setFormatter(logging.Formatter("{name} {lineno} {message}", style="{"))
fh.setLevel(0)

logging.getLogger("root").addHandler(fh)
logging.getLogger("root").setLevel(0)


logging.getLogger("a.b.c").debug("log this")

logging.getLogger("a").propagate = False

logging.getLogger("a.b.c").debug("don't log this")
# `debug.log`: a.b.c 12 log this
```

In `debug.log` we see the line `a.b.c 12 log this`, but nothing more. The line 12 from logger `a.b.c` has been logged by the handler of the `root` logger. the next Message was not handled, because logger `a` did not propagate it anymore to the `root` logger.

### Configuration for a single File

After all this theory I want to give you a little pratical shortcut for the configuration, that will probably solve 90% of all usecases. Writing the logs to a single file is this [typical usecase](https://docs.python.org/3/howto/logging.html#logging-to-a-file). Gladly, this can be easily configured, using `logging.basicConfig`:

```python
import logging

logger = logging.getLogger(__name__)

if __name__=="__main__":
    logging.basicConfig(filename='debug.log', encoding='utf-8', level=logging.DEBUG)
    logger.info("logging configured!")
```

> in order to avoid multiple configurations, make sure to place your configuration in a `if __name__=="__main__"`-block.

A more advanced configuration example will be shown in a later article, using `logging.config.dictConfig`. 

### Key Takeaways

1. Use logging if you need to save debug information to somewhere else than the console output

2. Initialize a logger with `logger = logging.getLogger(__name__)` at the beginning of every python-file for optimal usage and configurability

3. Within the `__name__=="__main__"`-block, use `logging.basicConfig` if you want to save your logs to a single file, otherwise use `logging.config.dictConfig`, which will be covered in another article