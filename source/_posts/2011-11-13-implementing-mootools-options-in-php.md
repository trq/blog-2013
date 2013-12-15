---
layout: post
title:  Implementing Mootools options in PHP
categories:
    - blog
---
[php5.4][php] has many new features that I can't wait to get into. Two of the
features that stand out for me at the moment are the new short array syntax and
[traits][traits].

The short array syntax means that you can define an array using:

    $arr = ['foo' => 'bar'];

Now, while this alone doesn't seem like a big deal. It does make passing arrays
of arguments to a function or method look allot cleaner.

    somefunction(['foo' => 'bar']);

In turn, this basically gives us a decent syntax that we can use to pass named
arguments to php functions.

With the introduction of traits we have the ability to define methods (within a
[traits][traits]) that can be shared across multiple classes without actually
extending the class. I'm not going to go into too much detail about
[traits][traits], as it's a pretty simply concept, and the following code
should demonstrate it pretty well.

Now, [mootools][mootools] has a cool little class Options that has a single
method setOptions. This enables you to merge your default options within a
[mootools][mootools] class, with some user defined overwrites.

Something like (taken directly from the manual [here][options]):

    var Widget = new Class({
        Implements: Options,
        options: {
            color: '#fff',
            size: {
                width: 100,
                height: 100
            }
        },
        initialize: function(options){
            this.setOptions(options);
        }
    });
     
    var myWidget = new Widget({
        color: '#f00',
        size: {
            width: 200
        }
    });
     
    //myWidget.options is now: {color: #f00, size: {width: 200, height: 100}}

We can implement something similar using [php][php]'s new [traits][traits] by
doing the following:

    <?php
    trait Options
    {
        private function setOptions(array $default_options, array $options)
        {
            return array_replace_recursive($default_options, $options);
        }

    }

    class Foo
    {
        use Options;

        private $options = [
            'foo' => 'bar',
            'boo' => 'bob',
        ];

        public function __construct(array $options = array())
        {
            $this->options = $this->setOptions($this->options, $options);
        }

        public function debug()
        {
            print_r($this->options);
        }

    }

    $foo = new Foo;
    $foo->debug();

    $foo = new Foo(['boo' => 'new']);
    $foo->debug();

This produces:

<pre>
Array
(
    [foo] => bar
    [boo] => bob
)
Array
(
    [foo] => bar
    [boo] => new
)
</pre>

Now, while this works and is kinda nice you can see it's not quite as clean as
the [mootools][mootools] implementation. The main reason being we've had to
pass the $this->options array into setOptions() is the first argument. The
reason for this is that at the time the [traits][traits] is defined, we are not
sure what array our default options are stored in. [traits][traits] can define
properties, but defining $this->options within the trait itself at this point
produces STRICT error something along the lines of:

<pre>
PHP Strict Standards:  Foo and Options define the same property ($options) in
the composition of Foo.
</pre>

Because we are then trying to overwrite it within the Foo object itself.

So, I'm going to tidy this up so that we don't need to define $this->options
within the class at all:

    trait Options
    {
        private $options = array();

        private function setDefaults($options)
        {
            $this->options = $options;
        }

        private function setOptions(array $options)
        {
            $this->options = array_replace_recursive($this->options, $options);
        }

        private function getOption($key)
        {
            if (isset($this->options[$key])) {
                return $this->options[$key];
            }

        }

    }

    class Foo
    {
        use Options;

        public function __construct(array $options = array())
        {
            $this->setDefaults([
                'foo' => 'bar',
                'boo' => 'bob',
            ]);

            $this->setOptions($options);
        }

        public function debug()
        {
            echo $this->getOption('boo') . "\n";
        }

    }

    $foo = new Foo;
    $foo->debug();

    $foo = new Foo(['boo' => 'new']);
    $foo->debug();

This results in:

<pre>
bob
new
</pre>

Ok, as you can see we are still using the private property $options. However,
we no longer need to define this within our class itself. This could of course
leed to trouble if our class overwrites $options, but that is another issue.

I've tidied up our classes __construct so that it can now firstly defines it's
defaults, then merges (or actually replaces) those defaults with a users
options passed in as arguments. For completeness, I have added a simple,
getOption() method which can be used throughout the class to retrieve an
option.

Sure, we could have done all this in earlier version of [php][php] but it would
have meant a) Implementing all this functionality in each class that required
it or worse b) extending some base <em>Options</em> object which would in turn
mean that our object could no longer extend any other objects.

Some people are against the whole idea of named arguments because it pretty
much blows away the idea of type hinting. However, with the approach that I
have taken here, it would be easy enough to extend the Options [traits][traits]
to be able to accept some form of 'type' argument that could potentially do
something similar. I might delve into that in a future post.

[php]:      http://php.net
[traits]:   http://au.php.net/traits
[mootools]: http://mootools.net
[options]:  http://mootools.net/docs/core/Class/Class.Extras#Options:setOptions
