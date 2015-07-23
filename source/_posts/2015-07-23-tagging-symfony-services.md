---
layout: post
title: Tagging Symfony Services
categories:
    - blog
---
It's been a long long time since I have written a blog post on any subject, but
it's time to get started again. Today, I bring you, _tagging Symfony services_.

Why would you want to tag services? You might ask. One reason is that tagging
services is one way to easily identify a particular set of services as a
specific type, strategy or implementation, allowing you to create a
simple modular plugin type system.

As an example, I'm going to build a very simple business rules manager which
will be able to apply numerous different business rules to some data via
different plugged in rules.

I start with a fresh install of the [Symfony Standard Edition][sse].

`symfony new tagged-services-article`

Open the DefaultController and edit it so that it looks like the following
example:

    <?php

    namespace AppBundle\Controller;

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class DefaultController extends Controller
    {
        /**
         * @Route("/", name="homepage")
         */
        public function indexAction()
        {
            $data = [2000,4000,8000,10000, 'abc', 20000, 40000, 'xyz'];

            return $this->render('default/index.html.twig', ['data' => $data]);
        }
    }

> Note I have also edited the route to match /

Now open the template file and edit it like so:

    {\% extends 'base.html.twig' %}

    {\% block body %}
        <ul>
        {\% for item in data %}
            <li>{{ item }}</li>
        {\% endfor %}
        </ul>
    {\% endblock %}

Now if you start the server `php app/console server:run` and visit
`http://127.0.0.1:8000` you should see something like:

![](/images/2015-07-23-tagging-symfony-services/1.png)

Ok, now we're going to get down to business. The first thing we need is going
to be a single point of entry. A RuleManager. Lets create it.

In AppBundle create a new directory called Service and create a new class
called RuleManager.php within it.

    <?php

    namespace AppBundle\Service;

    use AppBundle\RuleManager\Rule;

    class RuleManager
    {
        private $rules = [];

        /**
         * @param Rule $rule
         */
        public function addRule(Rule $rule)
        {
            $this->rules[] = $rule;
        }

        public function applyRules(array $data)
        {
            foreach ($this->rules as $rule) {
                $data = array_filter($data, function($value) use ($rule) {
                    return $rule->apply($value);
                });
            }

            return $data;
        }
    }

The RuleManager is responsible for taking all our data and filtering it by a
set of rules that we have not yet provided. Looking at the code, we can see
that each Rule needs to implement an interface Rule. Lets define that
simple interface within the `AppBundle\RuleManager` namespace.

    <?php

    namespace AppBundle\RuleManager;

    interface Rule
    {
        /**
         * @param mixed $value
         *
         * @return bool
         */
        public function apply($value);
    }

Now, lets build our first Rule. The first thing we need to do is filter out
any data that is not numeric.

    <?php

    namespace AppBundle\RuleManager;

    class IsNumericRule implements Rule
    {
        /**
         * @param mixed $value
         *
         * @return bool
         */
        public function apply($value)
        {
            return is_int($value);
        }
    }

Don't worry to much about how correct the implementation might be, the point of
the article isn't how to filter numeric data.

At this stage we can easily test the code in a browser by simply putting it all
together within the controller. Edit the indexAction.

    <?php

    namespace AppBundle\Controller;

    use AppBundle\RuleManager\IsNumericRule;
    use AppBundle\Service\RuleManager;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class DefaultController extends Controller
    {
        /**
         * @Route("/", name="homepage")
         */
        public function indexAction()
        {
            $data = [2000,4000,8000,10000, 'abc', 20000, 40000, 'xyz'];

            $ruleManager = new RuleManager();
            $ruleManager->addRule(new IsNumericRule());

            $data = $ruleManager->applyRules($data);

            return $this->render('default/index.html.twig', ['data' => $data]);
        }
    }

Visiting `http://127.0.0.1:8000` you should now see something like:

![](/images/2015-07-23-tagging-symfony-services/2.png)

Success! Our data is being filtered. Let's add some more rules. One to make
sure all values in our data are greater than 4000, and another to ensure our
values are less than 20000.

    <?php

    namespace AppBundle\RuleManager;

    class GreaterThanRule implements Rule
    {
        /**
         * @param mixed $value
         *
         * @return bool
         */
        public function apply($value)
        {
            return $value > 4000;
        }
    }

and:

    <?php

    namespace AppBundle\RuleManager;

    class LessThanRule implements Rule
    {
        /**
         * @param mixed $value
         *
         * @return bool
         */
        public function apply($value)
        {
            return $value < 20000:
        }
    }

Again we can go to our controller and add our new rules making sure to import their use statements.

    $ruleManager->addRule(new IsNumericRule());
    $ruleManager->addRule(new GreaterThanRule());
    $ruleManager->addRule(new LessThanRule());

Now we will only see two results. Perfect, our logic is working.

![](/images/2015-07-23-tagging-symfony-services/3.png)

It's pretty clear though that this isn't going to be at all easy to maintain.

We all know we should be using services and getting this stuff out of the
controllers, but how are we going to do that in a way that allows us to easily
add new rules?

Firstly, lets configure the RuleManager to be a service available from the container.

    # app/config/services.yml
    parameters:

    services:
      app.rule_manager:
          class: AppBundle\Service\RuleManager

Our controller should now be:

    <?php

    namespace AppBundle\Controller;

    use AppBundle\RuleManager\IsNumericRule;
    use AppBundle\RuleManager\LessThanRule;
    use AppBundle\RuleManager\GreaterThanRule;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class DefaultController extends Controller
    {
        /**
         * @Route("/", name="homepage")
         */
        public function indexAction()
        {
            $data = [2000,4000,8000,10000, 'abc', 20000, 40000, 'xyz'];

            $ruleManager = $this->get('app.rule_manager');
            $ruleManager->addRule(new IsNumericRule());
            $ruleManager->addRule(new GreaterThanRule());
            $ruleManager->addRule(new LessThanRule());

            $data = $ruleManager->applyRules($data);

            return $this->render('default/index.html.twig', ['data' => $data]);
        }
    }

Nothing fancy here. What did it buy us? Not much, we can swap out the
RuleManager for some other implementation, but really, we should be using
dependency injection anyway to get around that. Without getting side tracked
though, lets get to the point.

Finally we are getting to the meat of this article. Tagging services.

What we really want to be able to do is add new Rules to the RuleManager
easily without changing any code.

Let's make our rules services, and tag them so that we can easily identify
them.

    app.rule_manager.is_numeric:
      class: AppBundle\RuleManager\IsNumericRule
      tags:
        - { name: rule_manager.rule }

    app.rule_manager.greater_than:
      class: AppBundle\RuleManager\GreaterThanRule
      tags:
        - { name: rule_manager.rule }

    app.rule_manager.less_then:
      class: AppBundle\RuleManager\LessThanRule
      tags:
        - { name: rule_manager.rule }

All we have done here is made some normal services, then used the tags
attribute to identify these as being rule_manager.rule.

So, what does this buy us? Lots. By tagging them, we are now able to easily
identify them as the container is compiled and inject them into our existing
RuleManager service. Lets do just that.

Firstly, within the AppBundle directory create a DependencyInjection
directory. Then within that, create a new Compiler Pass class,
RuleManagerCompilerPass.php.

    <?php

    namespace AppBundle\DependencyInjection;

    use Symfony\Component\DependencyInjection\ContainerBuilder;
    use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
    use Symfony\Component\DependencyInjection\Reference;

    class RuleManagerCompilerPass implements CompilerPassInterface
    {
        public function process(ContainerBuilder $container)
        {
            if (!$container->has('app.rule_manager')) {
                return;
            }

            $definition = $container->findDefinition(
                'app.rule_manager'
            );

            $taggedServices = $container->findTaggedServiceIds(
                'rule_manager.rule'
            );

            foreach ($taggedServices as $id => $tags) {
                $definition->addMethodCall(
                    'addRule',
                    array(new Reference($id))
                );
            }
        }
    }

This code simply checks to see that a service exists named app.rule_manager,
retrieves it's definition, then finds all services tagged with
rule_manager.rule loops through them calling the RuleManager service's
addRule() method and passing a reference to each rule into it.

All that is left to do now is to register this CompilerPass. Edit the
AppBundle.php file:

    <?php

    namespace AppBundle;

    use AppBundle\DependencyInjection\RuleManagerCompilerPass;
    use Symfony\Component\DependencyInjection\ContainerBuilder;
    use Symfony\Component\HttpKernel\Bundle\Bundle;

    class AppBundle extends Bundle
    {
        public function build(ContainerBuilder $container)
        {
            parent::build($container);

            $container->addCompilerPass(new RuleManagerCompilerPass());
        }
    }

That's it. Our controller can now be simplified to:

    <?php

    namespace AppBundle\Controller;

    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class DefaultController extends Controller
    {
        /**
         * @Route("/", name="homepage")
         */
        public function indexAction()
        {
            $data = [2000,4000,8000,10000, 'abc', 20000, 40000, 'xyz'];

            $data = $this->get('app.rule_manager')->applyRules($data);

            return $this->render('default/index.html.twig', ['data' => $data]);
        }
    }

One of the major benefits now is that we can easily add more rules without ever
touching any existing code. It's all done via configuration.

For more info on tagging services see [here][doc].

I also have a git repository containing the code used in this article available [here][repo].

[sse]: https://github.com/symfony/symfony-standard
[doc]: http://symfony.com/doc/current/components/dependency_injection/tags.html
[repo]: https://github.com/trq/tagged-services-article
