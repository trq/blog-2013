---
layout: post
title:  Namespacing twitter bootstrap for ease of interpolation
categories:
    - blog
---
At work we have a dozen or so apps that share and extend a common CSS base that we have written and implemented ourselves. Over the last week or so however, I have been slowly implementing bits and pieces of the awesome [Bootstrap][bs] framework into one of those apps. In general, it's gone pretty well. I have taken my time and only pulled in the bits I needed as I needed them. Mostly just buttons, icons and a few drop down menus.

We have our own version of [bootstrap.less][bsless] which pulls in the bootstrap code, and also defines any stuff we have overridden. Most of the @import calls have been commented out, and this is how I had been managing what was being compiled by [Less][less].

Before I left work this afternoon however, chatting with one of the other devs, we decided we wanted to start making use of the scaffolding [Bootstrap][bs] provides. To get this working (and looking good) we would need to pull in [Bootstrap][bs]'s [scaffolding][bsscaf], [grid][bsgrid], [layouts][bslayouts] and [bsreset][reset]. I knew this would break stuff. We needed a solution that would make this breakage containable.

During development we have [Bootstrap][bs] compiled via [lessphp][leafo] on the fly. Looking at the [less][less] documentation we found something that looked promising, [namespaces][less-ns].

Putting [Bootstrap][bs] into it's own namespace would allow us to easily pull in whatever we need at a document, div or element level. We would also have complete control of this right where we need it, in our actual templates.

So, how do you namespace [Bootstrap][bs] easily? Simple. Open your [bootstrap.less][bsless] file and wrap all the @import calls within a .bootstrap {} block. eg;

    .bootstrap {

    // CSS Reset
    @import "reset.less";

    // Core variables and mixins
    @import "variables.less"; // Modify this for custom colors, font-sizes, etc
    @import "mixins.less";

    // Grid system and page structure
    @import "scaffolding.less";
    @import "grid.less";
    @import "layouts.less";

    // more less ...

    }

That's it. Now, you can use [Bootstrap][bs] by simply applying the _bootstrap_ class to the element surrounding the area you wish to use [Bootstrap][bs].

Now just to covert the rest of the [Bootstrap][bs] JavaScript to not rely on [jQuery][jquery].

[bsless]:       https://github.com/twitter/bootstrap/blob/master/less/bootstrap.less
[bsscaf]:       https://github.com/twitter/bootstrap/blob/master/less/scaffolding.less
[bsgrid]:       https://github.com/twitter/bootstrap/blob/master/less/grid.less
[bslayouts]:    https://github.com/twitter/bootstrap/blob/master/less/layouts.less
[bsreset]:      https://github.com/twitter/bootstrap/blob/master/less/reset.less
[bs]:           http://twitter.github.com/bootstrap
[jquery]:       http://jquery.org
[less]:         http://lesscss.org/
[leafo]:        http://leafo.net/lessphp/
[less-ns]:      http://lesscss.org/#-namespaces
