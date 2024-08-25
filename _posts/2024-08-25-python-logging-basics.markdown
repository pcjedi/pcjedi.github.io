---
layout: post
---
# Getting the basics of Logging in Python

In this post I will go through the basic components and concepts of the build-in logging module in python. But first, let's look at this very basic example:

```python
import logging

logging.warning(msg="you gonna learn something today!")
# WARNING:root:you gonna learn something today!
```

By this, we have used the `root`-logger, to log a message on the warning level.

## Logging Components

Here, i will quickly explain the 3 basic components of the Logging module. Other, more advanced concepts like Filters, Records, Arguments and Extras will be explained elsewhere.

### Logger

When using the logging functions available directly in the logging module, like in the example above, you are using the `root`-logger.
You can have more than one logger.
Each logger is defined by its name.
You would never instantiate a logger directly, but rather call the `logging.getLogger`-function, which does this for you. This function gets the logger and if not available, creates it for you.

### Handler

The logging-Handler define what to do with the log-Messages. These could be send to STDOUT/STDERR in case of a StreamHandler. Or a File in Case of a FileHandler. Another Handler maybe the HTTPHandler. Loggers and Handlers are in a Many-to-Many Relation. A Logger may has several Handlers, meaning that the Log-Message ends up in multiple places. Also a Handler may get added to multiple Loggers. The last case may have practical merit, but could probably be better solved using Ancestry and Propagation. More on that later.

### Formatter

The Formatter defines how the Message is displayed. In the example above, you can see the Level and the Name of the Logger within the created output.
[Here is a list of available Attributes](https://docs.python.org/3/library/logging.html#logrecord-attributes)


### Bringing those together

```python
import logging

logger = logging.getLogger(name="loggername")

handler = logging.StreamHandler()
handler.formatter = logging.Formatter(fmt="%(name)s %(lineno)d %(message)s")

logger.addHandler(handler)

logger.warning("you have been warned!")
# loggername 10 you have been warned!
```

## Logging Concepts

Knowing the logging components, you can already effectively use the Logging module. Getting the most of it and understanding further techniques requires the knowledge of the next three concepts, that can play hand in hand when mastering the python logging module.

### Level

This one is rather simple, due to its linear nature. The Log-Level is an Integer, that is attached to every Logger, Handler and Message. If the Level of the Message is lower than the Level of the Logger or Handler, it will be ignored. [Reference](https://docs.python.org/3/library/logging.html#logging-levels)

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

We created a Logger named `a`. After that we created a Logger `b` with `a` as its parent. After that, we created a Logger `c` with `b` as its Parent. We can make use of this by using `__name__` as the logger name. If we are inside a module and not `__main__`, the logger from the file/folder one hierarchy level above will become the ancestor. This will make it easy to configure the logging on a module basis, when executed.

> always initialize a logger like this `logger = logging.getLogger(__name__)`

### Propagation

The advantage of Ancestry is, that certain properties of loggers can be 'inherited' through loggers due to propagation. I am not talking about real Inheritance here like meant in OOP, but rather an implicit one, due to the Propagation of Log Messages through the Logger Hierarchy. Meaning, every Message will be handled by the Handlers of the Logger and afterwards be propagated to the Parent, where this loop continues, until the Root Logger is reached. The Propagation can be deactivated at any Logger. Consider the following Example:

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

in `debug.log` we see the line `a.b.c 12 log this`, but nothing more. The line 12 from logger `a.b.c` has been logged by the handler of the `root` logger. the next Message was not handled, because logger `a` did not propagate it anymore to the `root` logger.
