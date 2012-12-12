# How to start with Backbone.js: A simple skeleton app

<aside style="float: right; width: 350px; padding: 0 0 20px 20px;">
<nav id="outline">
  <h3>Outline</h3>
  <ol>
    <li><a href="#philosophy">Philosophy</a></li>
    <li><a href="#backstory">Backstory</a></li>
    <li><a href="#tools">Tools</a></li>
    <li><a href="#usage">Using the skeleton</a></li>
    <li><a href="#resources">Useful resources</a></li>
  </ol>
</nav>
<section id="tldr">
<h3>TL;DR</h3>

<p>It took me some time to get an optimal code/directory layout for Backbone.js apps.</p>
<p>Because I think this is a major pain for beginners, I prepared a well commented <a href="https://github.com/mihar/backbone-skeleton">sample skeleton app</a>.</p>
<p>Get it from Github while it's hot, pull requests/feedback are welcome.</p>
</section>
</aside>

## Prerequisites

You need a solid knowledge of JavaScript, familiarity with Backbone, Ruby, HAML, SASS and CoffeeScript to find this writeup useful.

Also, I'm developing on a Mac and have not tested this on other platforms. Although, I do not see any reason why it shouldn't work.

<a name="philosophy"></a>
## 1. Philosophy

Part of the success of [Rails](http://rubyonrails.com) was the conventions and it's predefined directory tree. While looking overwhelming and
maybe annoying to a beginner at first, it soon becomes liberating. With experience things fall into place, and soon you feel
feel like every tiny bit of code has it's dedicated home.

[Backbone](http://backbonejs.org), being the nimble, does not prescribe any particular code or directory structure. Until I read enough material 
and settled on this particular layout, I was feeling very confused and disoriented.

[This skeleton app](https://github.com/mihar/backbone-skeleton) was extracted from a production app and then extensively annotated, to explain certain decisions and choices.

<a name="backstory"></a>
## 2. Backstory

When I first started to play with Backbone I was already heavily entrenched in the Ruby and Rails world. 
So naturally I thought, yeah, MVC, I know that. It turned out to be a bit farther from the truth than I 
wanted or cared to admit.

Disclaimer: *I rarely developed pure client-side software, though I was using JavaScript extensively to make 
things faster and more responsive.*

Thing is that MVC on the server-side is quite a bit different from MVC on the client. It has something to
do with wiping the state clean each time you reload the page. Statelessness.

The client on the other hand is stateful and thus keeps all your bad practices in memory until they start
to slow things down and eventually stop working.

The hardest part of learning to develop MVC on the client and using Backbone was seeing the big picture.
Seeing where all the little parts fit in and how this all works together in the grand scale.

So for the most part, my journey with Backbone consisted of finding out best practices for file and code 
organization, setting up the environment and directory structures.

Using backbone.js as a library was "easy". *(Not really, but this isn't what this article's about.)*

One of the biggest mistakes I was making when starting out was **trying to use Backbone constructs for everything**.

Backbone is intentionally kept simple, because it's supposed to be a complement to your own JavaScript. So just create your
own `App` class, and populate it with the stuff and initialization your app really needs.

<a name="tools"></a>
## 3. Tools

So a good workflow needs good tools. Here I'll describe the tools that I found indispensable when developing in Backbone.

<img src="http://media.tumblr.com/tumblr_meu8mprgfy1qahol6.png" style="float: right; padding: 0 0 20px 20px;">

### CoffeeScript, HAML and SASS

Because I resent cruft and redundancy, I'm a big fan of abstraction languages. Whenever I can, I opt for **[HAML](http://haml.info)**, 
**[SASS](http://sass-lang.com)** and **[CoffeeScript](http://coffeescript.org)**. 

The brevity they bring is paramount to me.

### HAML Coffee

In Backbone, you usually need a template engine. Templates provide the markup for views. There's a lot of
solutions for this, but because I like to be consistent, the best choice was to use HAML.

Fortunately, there's a library for this: [haml-coffee](https://github.com/netzpirat/haml-coffee), which enables you
to use HAML intertwined with snippets of CoffeeScript.

<img src="http://media.tumblr.com/tumblr_meu8c0jkq71qahol6.png" style="float: right; padding: 0 0 20px 20px;">

### Guard

To be able to use these languages seamlessly, you need some sort of a on-demand compiler.
Turns out a Ruby gem called [Guard](https://github.com/guard/guard) does exactly this.

Guard is extremely flexible. It watches for file system changes and then doing something to files that changed.


### Jammit

<img src="http://media.tumblr.com/tumblr_meu8cvNwwk1qahol6.png" style="float: right; padding: 0 0 20px 20px;">

[Jammit](http://documentcloud.github.com/jammit/) is an asset packaging library. It concatenates
and compresses CSS and Javascript. It's easy to use, but needs a [configuration file](http://documentcloud.github.com/jammit/#configuration), that defines which files to work on.

### Sinatra with Isolate

Backbone apps are static files and you can run them directly off your hard drive. But to do proper paths and even maybe
some API, we need a server.

<img src="http://media.tumblr.com/tumblr_meu8eq6m0U1qahol6.png" style="float: right; padding: 0 0 20px 20px;">

[Sinatra](http://www.sinatrarb.com), a mini Ruby web framework, forms the base of the server. This enables some quick server-side magic as well as making an API for persistence.

To make this part as easy as possible to use, I packaged the server with [Isolate](https://github.com/jbarnette/isolate), a small Ruby library for sand-boxing, which is like a mini-[Bundler](http://gembundler.com). When launching the server with `rake server` for the first time, it will check and auto-install it's dependencies. *It just works.*

<a name="usage"></a>
## 4. Using the skeleton

Getting started with a new app using my skeleton is trivial. It uses Ruby in several critical places, so be sure you have a working installation of Ruby, preferably of the 1.9 kind.

All of the files, directories and their meaning is described with more detail in the [README](https://github.com/mihar/backbone-skeleton#readme) file of the skeleton.

A [working example](http://examples.breakthebit.org/backbone-skeleton/) of this app is available online. This way you can check if the console output is the same on your local setup and here.

Start by cloning my [backbone-skeleton repo](https://github.com/mihar/backbone-skeleton).

    $ git clone https://github.com/mihar/backbone-skeleton.git my-new-backbone-app

Then use the `bundle` command that comes with [Ruby Bundler](http://gembundler.com/v1.2/index.html) to install the necessary dependencies for guard. Guard will compile our HAML, SASS and CoffeeScript to their native counterparts.

    $ cd my-new-backbone-app
    $ bundle

Once Bundler completes the installation, we can try starting Guard, to immediately start watching files for changes.

    $ bundle exec guard

While leaving guard running, go to another terminal and let's fire up a simple, bundled Ruby web server, that we'll use for development. The server will install all of it's dependencies by itself.

    $ rake server

    [1/1] Isolating sinatra (>= 0).
    Fetching: rack-1.4.1.gem (100%)
    Fetching: rack-protection-1.2.0.gem (100%)
    Fetching: tilt-1.3.3.gem (100%)
    Fetching: sinatra-1.3.3.gem (100%)
    [2012-12-05 18:17:05] INFO  WEBrick 1.3.1
    [2012-12-05 18:17:05] INFO  ruby 1.9.3 (2012-02-16) [x86_64-darwin11.3.0]
    [2012-12-05 18:17:05] INFO  WEBrick::HTTPServer#start: pid=39675 port=9292

Now our server is listening on [http://localhost:9292](http://localhost:9292), so go ahead, and open that.

If you see "Skeleton closet", everything is go.

![](http://media.tumblr.com/tumblr_meu8pymBnq1qahol6.png)

Go check out the JavaScript console for more information.

![](http://media.tumblr.com/tumblr_meu8kjjYVI1qahol6.png)

<a name="resources"></a>
## 5. Resources

### [The missing CDN](http://cdnjs.com)

They host *all* the libraries, including Backbone and underscore.

### [Backbone Peepcode tutorials](https://peepcode.com/products/backbone-js)

Peepcode has been my friend since my Ruby and Rails days. They produce high-quality screencasts on a variety of topics.

They have a series of 3 videos on Backbone, going from the basics to some pretty advanced stuff. 

Be prepared to shell out $12 per video, though.

### [Derick Bailey backbone posts](http://lostechies.com/derickbailey/category/backbone/)

I've learned so much about the correct ways to do things on Derick's blog. He's a seasoned Backbone developer
that has overcome many problems and written up on the progress. Wealth of resources.

### [Derick Bailey's 4 part screencast](http://pragprog.com/screencasts/v-dback/hands-on-backbone-js)

Haven't seen this one yet, but if I'm judging by his blog, this should be very worth the money. 4 videos, $12 a pop.

### [Backbone Patterns](http://ricostacruz.com/backbone-patterns/)

Documented patterns extracted from building many Backbone apps.

### [Organizing Backbone apps with modules](http://weblog.bocoup.com/organizing-your-backbone-js-application-with-modules/)

Article exploring similar problems of code/directory structure and organizations.

### [Backbone Boilerplate](http://weblog.bocoup.com/introducing-the-backbone-boilerplate/)

A much bigger project with a similar goal as mine.

### [Backbone for absolute beginners](http://adrianmejia.com/blog/2012/09/11/backbone-dot-js-for-absolute-beginners-getting-started/)