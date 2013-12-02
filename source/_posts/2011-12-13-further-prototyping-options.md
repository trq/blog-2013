---
layout: post
title:  Further prototyping options
categories:
    - blog
---
Ok, it's getting pretty late here so this post wont be too detailed. I'm just further prototyping my Options trait and I think it's getting pretty close to being ready to implement into [Proem][proem].

You can see my original post [here][options]. There where a few things that bugged me about the original implementation. My main quibble was that it completely wiped out all the benefits provided by type hinting. Hence, I decided to try to implement something where I could actually validate the options as they come in.

This is what I have come up with. It should be pretty self explanatory.

    <?php

    class Something {}

    trait Options
    {
        private $options = array();

        private function setDefaults($options)
        {
            $this->options = $options;
        }

        private function setOptions(array $options)
        {
            foreach ($options as $key => $value) {
                if (isset($this->options[$key])) {
                    $this->options[$key]->setValue($value);
                } else {
                    $this->options[$key] = new Option($value);
                }
            }

            foreach ($this->options as $key => $value) {
                try {
                    $value->validate();
                    $this->options[$key] = $value->getValue();
                } catch (Exception $e) {
                    throw new \Exception($key . $e->getMessage());
                }
            }
        }

        private function getOption($key)
        {
            if (isset($this->options[$key])) {
                return $this->options[$key];
            }
        }
    }

    class Option
    {
        private $value;
        private $is_required = false;
        private $is_object = false;
        private $obj_type = null;

        public function __construct($value = null) {
            $this->value = $value;
        }

        public function setValue($value) {
            $this->value = $value;
            return $this;
        }

        public function getValue() {
            return $this->value;
        }

        public function required() {
            $this->is_required = true;
            return $this;
        }

        public function object($obj) {
            $this->is_object = true;
            $this->obj_type = $obj;
            return $this;
        }

        public function validate() {
            if ($this->is_required) {
                if (!isset($this->value)) {
                    throw new Exception(' is required');
                }
            }

            if ($this->is_object) {
                if (!is_object($this->value)) {
                    if (!is_a($this->value, $this->obj_type)) {
                        throw new Exception(' is required to be of type ' . $this->obj_type);
                    }
                }
            }
            return true;
        }
    }

    class Foo
    {
        use Options;

        public function __construct(array $options = array())
        {
            $this->setDefaults([
                'foo' => (new Option('foo')),
                'bar' => (new Option())->required(),
                'boo' => (new Option())->required()->object('Something')
            ]);

            $this->setOptions($options);
        }
    }

    $f = new Foo([
        'bar' => 'blahblahblah',
        'boo' => new Something,
        'bob' => 'this is bob'
    ]);
    var_dump($f);

It's still a little rough, but it works and can be fine tuned as it is used. This prototype only validates required options, and that they are of a specific object type. An actual implementation would validate for arrays, integers, strings and most likely regex pattern matches.

The entire validation process is currently really rough, but that will definitely and will definitely be refactored and extended. I'm also not overly happy with how the setOptions() method is designed within the Options trait, but again, that can and will be refactored.

It's the Foo class where the actual usage syntax takes place. I think it's reasonably tidy for the amount of work that is being done.

I'd be really keen to get some feedback on the concept so any comments are more than welcome.

edit:

Actually, thinking about this a bit more, I can do away with the extra dependency on some Option object _and_ tidy the end syntax up a little by using an array instead.

The end result will end up looking something like:

    <?php

    class Foo 
    {
        use Options;

        public function __construct(array $options = array())
        {   
            $this->setDefaults([
                'foo' => 'foo',
                'bar' => ['required'],
                'boo' => ['required', 'object' => 'Something']
            ]); 

            $this->setOptions($options);
        }   
    }

All the validation would the occur within the Options trait itself.

[proem]:         http://proemframework.org
[options]:       http://thorpesystems.com/2011/11/implementing-mootools-options-in-php
