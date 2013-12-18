---
layout: post
title: Debugging PHP in Vim
categories:
    - blog
---
For a long time now vim has been my editor of choice. Configured well, and used
in conjunction with some other _unixy_ tools it makes for a great IDE. However,
for far too long I have debugged my PHP applications using a combination of
echo statements, var dumps and writing to log files. This madness has to stop!

Enter [Vdebug][vdebug]. Vdebug is an awesome plug-in for vim that acts as a
client for debuggers supporting the DBGP protocol. Xdebug for PHP, is one such
debugger.

### Installation

Installation is a simple process following the standard work flow for
installing a vim plug-in. I use [Vundle][vundle] to make this process simpler
to manage. Simply add:

    Bundle "gmarik/vundle"

to your ~/.vimrc file (or wherever you maintain your plug-ins), then execute:

    vim +BundleInstall +qall

from your terminal.

Vdebug is now installed.

### Configure Xdebug

I'm not going to go into the installation process for Xdebug as it differs
somewhat from distro to distro. I'm currently setting this all up on OSX, so
installing Xdebug was as simple as `brew install php5-xdebug`.

There are all sorts of different options which can effect the configuration of
Xdebug. I have:

    [xdebug]
    zend_extension="/usr/local/Cellar/php55-xdebug/2.2.3/xdebug.so"
    xdebug.remote_autostart=1
    xdebug.remote_enable=1
    xdebug.remote_handler=dbgp
    xdebug.remote_host=localhost
    xdebug.remote_port=9000

within my `/usr/local/etc/php/5.5/conf.d/ext-xdebug.ini` file. Note that
`xdebug.profiler_enable=1` configures the profiler to remain on always. I use
Vdebug that often now that it is just easier to do so. You can however
configure the Xdebug's profiler to switch on and off based on different flags
passed in via the querystring, or even via a cookie. See the Xdebug docs for
details.

### Basic Usage

First, an example using the default config that ships with Vdebug. I'm using a
stock standard Symfony2 app running in the dev environment to demonstrate. With
`app/console --dev=env server:run` running in the backgorund, open
`web/app_dev.php` within vim and hit `<F5>`. Now visit `http://localhost:8000` in
your browser. Coming back to vim you should see something like:

<a href="/images/2013-12-02-debugging-php-in-vim/initial_screen.png"><img src="/images/2013-12-02-debugging-php-in-vim/initial_screen.png" width="650" /></a>

We have ended up at the first line of the `router_dev.php` file within the main
`DebuggerSource` window. On the right hand side there are various other
windows:

* `DebuggerWatch` window displaying the variables at the current position in execution.
* `DebuggerStack` shows the current execution stack.
* `DebuggerStatus` the status of the debugger.

By default Vdebug will simply go to the first line of your program and halt.
This is why we ended up within the `router_dev.php` file. This doesn't seem
that intuitive to me so I have configured Vdebug to execute up to my first
breakpoint. Click `<F6>` (twice) to detach from the debugger and add the
following to your ~/.vimrc file.

    g:vdebug_options["break_on_open"]=0

Now we need to actually set a break point. Moving the cursor down to the
`$response = $kernel->handle($request);` line of the `app_dev.php` I hit <F10>
inserting a breakpoint. You should now see something like:

<a href="/images/2013-12-02-debugging-php-in-vim/breakpoint.png"><img src="/images/2013-12-02-debugging-php-in-vim/breakpoint.png" width="650" /></a>

Note the B> in the gutter next to the line you were on when you clicked <F10>?
This lets you know where your breakpoints were added. Now, clicking <F5> (to
start the debugger) and hitting `http://localhost:8000` again in your browser
should execute your application up to that breakpoint:

<a href="/images/2013-12-02-debugging-php-in-vim/breakpoint_2.png"><img src="/images/2013-12-02-debugging-php-in-vim/breakpoint_2.png" width="650" /></a>

Vdebug has all the options you would expect from a debugger.

* `<F5>` - Run - Tells the debugger engine to run, which it will do until it meets a
* `<F2>` - Step Over - Tells the debugger engine to step to the next statement.
* `<F3>` - Step in - Tells the debugger engine to step into a statement on the current line.
* `<F4>` - Step out - Tells the debugger engine to step out of the current statement into the calling statement.
* `<F9>` - Run to cursor - Tells the debugger engine to run until the point where the cursor is currently sitting.

Then there are `<F7>` and `<F6>` which essentially kill the debugger. I ended
up reconfiguring most of the key bindings to something that made more sense to
me:

    " Vdebug
    let g:vdebug_keymap = {
    \    "run" : "<Leader>/",
    \    "run_to_cursor" : "<Down>",
    \    "step_over" : "<Up>",
    \    "step_into" : "<Left>",
    \    "step_out" : "<Right>",
    \    "close" : "q",
    \    "detach" : "x",
    \    "set_breakpoint" : "<Leader>p",
    \    "eval_visual" : "<Leader>e"
    \}

    let g:vdebug_options = {
    \    "break_on_open" : 0,
    \}

Once you have some breakpoint set and are navigating through the execution of
your program, the `DebuggerWatch` window is probably one of the more
significant aspects of Vdebug. In here you can easily inspect its values,
clicking on any array or object also opens it for further inspection.

There also exists options for executing expressions, either manually via
`:VdebugEval $x + 2` or by visually highlighting the expression you wish to
evaluate and then hitting `<Leader>e`.

Anyway, thats it for a high level overview. Install and have a play with it
yourself to get a better feel for what it can do. It really will change your
workflow and improve your ability to track down issues.

[vdebug]:   https://github.com/joonty/vdebug
[vundle]:   https://github.com/gmarik/vundle
