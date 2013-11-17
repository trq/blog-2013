---
layout: post
title:  Very simple events in php.
categories:
    - blog
---
I have spent most of the morning going over some ideas I would like to introduce within the [Proem][proem] including Dependency Injection Containers, and simple Events.

I fell in love with the idea of events while I was freelancing using [wordpress][Wordpress]. Although I think most of Wordpress's code sux, it's 'Hook' system makes it easy to add functionality without ever having to hack [wordpress][Wordpress] itself. This is something I would like to build into [proem][Proem] from the ground up.

Triggering _events_ and having _listeners_ listen out for these events would add allot of flexibility to Proem. Both within the framework itself, and also within applications built on top of it.

The basics of it is quite simple to implement, so here is a small working prototype:

```
<?php
class Event
{
    private $name;
    private $params;

    public function __construct($name, $params = array()) {
        $this->name = $name;
        $this->params = $params;
    }

    public function getName() {
        return $this->name;
    }

    public function getParams() {
        return $this->params;
    }
}

class EventManager
{
    private $events = array();

    public function attach($name, $callback) {
        $this->events[$name][] = $callback;
    }

    public function trigger($name, $params = array()) {
        foreach ($this->events[$name] as $event => $callback) {
            $e = new Event($name, $params);
            $callback($e);
        }
    }
}

$events = new EventManager;

$events->attach('do', function($e) {
    echo $e->getName() . "\n";
    print_r($e->getParams());
});

$events->trigger('do', array('a', 'b', 'c'));
```

Like I said, this is just a simple implementation. The first thing that we would want to do with this code would be to implement a static event manager of sorts which implemented a singleton. This would enable events to be triggered in a global manner. Having implemented that, we would likely also want to add the ability to namespace events somewhat so that listeners could concentrate on only listening to events triggered by specific objects or namespaces for example.

Another great idea I have seen kicking around would be to be able to trigger events along with a priority. This priority would then be responsible for sorting what order they where executed with.

Of course, on top of this, it would also be of great value to be able to halt the executing process if a certain listener determines this should happen. You could then use events for things like ACL checks, and if a user doesn't pass the ACL, the listener should be able to deny access.

Anyway, it's just something I'm playing with. At this point in time, there is allot of stuff catching my eye, and it's starting to look like [Proem][proem] could do with a rewrite before it's even at an initial beta. Damn.

Coming back to this post less than an hour later, and I have now implemented priorities and the ability for the Event to return data back to a callback within the trigger.

```
<?php
class Event
{
    private $name;
    private $params;

    public function __construct($name, $params = array()) {
        $this->name = $name;
        $this->params = $params;
    }

    public function getName() {
        return $this->name;
    }

    public function getParams() {
        return $this->params;
    }
}

class EventManager
{
    private $events;

    public function __construct() {
        $this->events = new SplPriorityQueue;
    }

    public function attach($name, $callback, $priority = 0) {
        $this->events->insert(array($name, $callback), $priority);
    }

    public function trigger($name, $params = array(), $callback = null) {
        foreach ($this->events as $event) {
            if ($event[0] = $name) {
                $e = new Event($name, $params);
                if ($r = $event[1]($e)) {
                    if (is_callable($callback)) {
                        $callback($r);
                    }
                }
            }
        }
    }
}

$events = new EventManager;

$events->attach('do', function($e) {
    echo "Registered first\n";
    return "Hello";
});

$events->attach('do', function($e) {
    echo "Registered second\n";
}, 100);

$events->trigger('do', array('a', 'b', 'c'), function($r) {
    echo "$r\n";
});
```

The idea of halting execution might need a bit more work. I don't want to kill the entire application, it would be best to actually just stop any more events from being propagated instead. I guess I'll leave that for another post,

[proem]:        http://proemframework.org
[wordpress]:    http://wordpress.org
