---
layout: post
title: Testing Multiple Interface Implementations
categories:
    - blog
---
Recently I created a *Message Queue Component* as part of a larger system I work
on for my employer. Part of the requirement was to ensure the *Queue* would
work with many different providers. (RabbitMq, Beanstalk etc). I achieved this
quite easily using the *Adapter* pattern and creating what I called *Drivers*.

Along the way, I discovered a pretty cool way to test multiple implementations
of a common interface.

To keep this example simple, we are just going to implement a couple of methods
of a simplified *Driver* interface across two different drivers.

>>> The point of this article is to show you how I tested multiple implementations of an interface, not how to build a queue.

Our incomplete interface;

```
<?php

namespace Trq\Queue;

interface Driver extends \Countable
{
    /**
     * Push a message onto the queue
     */
    public function push(string $msg);
}
```

Now for an initial test of the *InMemory* driver.

```
<?php

namespace Test\Trq\Queue;

use Trq\Queue\Driver\InMemory;

class InMemoryDriverTest extends \PHPUnit_Framework_TestCase
{
    public function testCanPushAndGetCount()
    {
        $driver = new InMemory();

        $driver->push('message 1');
        $driver->push('message 2');
        $driver->push('message 3');

        $this->assertCount(3, $driver);
    }
}
```

Pretend for a second that we now have a complete and working *InMemory* driver that
has been fully tested. We are next tasked with creating a *RabbitMq* driver.

Instead of repeating our tests, lets refactor what we have to test so that it
can easily test multiple implementations.

```
<?php

namespace Test\Trq\Queue;

use Trq\Queue\Driver;

class AbstractDriverTestCase extends \PHPUnit_Framework_TestCase
{
    protected $driver;

    public function setUp()
    {
        $this->driver = static::getDriver();
    }

    abstract public function getDriver() : Driver;

    public function testCanPushAndGetCount()
    {
        $this->driver->push('message 1');
        $this->driver->push('message 2');
        $this->driver->push('message 3');

        $this->assertCount(3, $this->driver);
    }
}
```

Our *InMemoryDriverTest* now simply becomes;

```
<?php

namespace Test\Trq\Queue;

use Trq\Queue\Driver\InMemory;

class InMemoryDriverTest extends AbstractDriverTestCase
{
    public function getDriver()
    {
        return new InMemory();
    }
}
```

When we create a new driver, we simply create another test extending the
`AbstractDriverTestCase` and implement the `getDriver()` method.
```
<?php

namespace Test\Trq\Queue;

use Trq\Queue\Driver\RabbitMq;

class RabbitMqDriverTest extends AbstractDriverTestCase
{
    public function getDriver()
    {
        return new RabbitMq(
            require __DIR__ . '/rabbit-test-config.php'
        );
    }
}
```

This isn't rocket science, but it's not immediately obvious that you can
actually have tests in your abstract test cases.

Organising things this way allows us to easily test that all implementations of our
interface behave the same way.
