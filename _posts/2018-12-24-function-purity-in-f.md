---
layout: post
title: Function Purity in F#
date: 2018-12-24 23:44
author: Ashish Vegaraju
comments: true
categories: []
---
<!-- wp:paragraph -->
<p>In my last article about <a href="https://ashishvegaraju.com/2018/09/24/functional-domain-modeling/">Functional domain modeling</a>, I explored the expressiveness of F# in modeling a domain. I fell in love with the simplicity and expressiveness of the language. In this article I will attempt to explore function purity in F#. In F#, functions are first class citizens because it allows to pass function as an argument to other function, return a function or assign function to a variable. Initially I found it a bit hard to wrap my head around the concept of treating functions as first class citizens. In fact one of the biggest challenge for me was surprisingly not the weird syntax of F#, but to think in terms of functions.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Coming from an OO background and not very comfortable with F#, below was my first attempt to write a use case. I basically tried to mimic a use case in a typical ports and adapters project from one of the C# projects. This use case serves a basic purpose which is to update the dimensions of a product, if it exists, of course.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist 18a503c2351100ee5c3568efc8426f49 %}</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Pretty straight forward. The use case accepts a <code>DataStore</code> dependency using which the <code>update</code> method can query the <code>Products</code> table for existence of a product. If the product is found then the product is fetched from the data store and dimensions are updated. Finally, the product with updated dimensions is persisted in the database using the data store.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Though this use case works, there are several problems with this way of writing code. Lets take a look at the problems:</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->
<h4><strong>Testability</strong></h4>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The code above is not easily testable. Why you ask? Lets look at how many ways things can go wrong with this code.</p>
<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->
<ol><li>What if the constructor arguments are null?</li><li>What if the <code>productExists</code> data store method returns an exception.</li><li>What if <code>updateDimensions</code>, which in turn calls <code>getProduct</code> and <code>updateProduct</code> methods on the data store returns exception?</li><li>What if the product does not exist?</li><li>What if product dimensions does not exist?</li><li>what if ....</li></ol>
<!-- /wp:list -->

<!-- wp:paragraph -->
<p>and the list will go on ..</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>So many scenarios to test with just one dependency. What if I add <code>ExternalService</code> as another dependency to this use case? Imagine the number of ways this third party service call can go wrong. The point is that whenever we deal with external systems, be it database or an external service, we are entering a world of uncertainty and we do not have much control over the behavior of those external systems. It would be nice if the use case and the domain model could completely avoid any type of IO operations or side affects. Sounds like a nice idea, but a use case which does not perform IO is next to useless. Stay with me and we will explore a way to minimize or avoid IO and side affects in the use case and domain model by the end of this article.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->
<h4>Hidden dependencies</h4>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>One of the beautiful things about F# is its expressiveness. The <code>update</code> method in our example is all but expressive about its intent. It has a dependency on <code>DataStore</code> which is not evident when we look at the method signature. This is what I call as a hidden dependency. What if in future someone modifies the <code>update</code> method and calls a completely different method on the datastore? Since the <code>update</code> method is not expressive enough, developers can assume a lot of things. Is there a way to avoid these hidden dependencies? Can we make the update method a little more expressive to avoid mistakes by future developers? Again, by the end of this article, we will try to re-write the <code>update</code> method to make it a little bit more expressive.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->
<h4>Referential Transparency</h4>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Methods in our use case are not referentially transparent. Especially the <code>update</code> method is not by any means. What is referential transparency? A function is referentially transparent if the function can be replaced with its corresponding value without affecting the behavior of the system. For example,</p>
<!-- /wp:paragraph -->

<!-- wp:preformatted -->
<pre class="wp-block-preformatted">if, x + y = 10<br><br>then, the expression x + y + z = 20 can be written as 10 + z = 20</pre>
<!-- /wp:preformatted -->

<!-- wp:paragraph -->
<p>In our use case the method <code>update</code>, depending on whether one of the data store methods return an exception, may or may not update the product dimensions. We can never assume that a call to <code>update</code> method always updates product dimensions. Due to this uncertainty, I will not be able to include <code>update</code> method in a method chain like this and assume that warehouse system will be always notified after the product dimensions update.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist 53dd6dc264b22ae4664983445e567312 %}</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>There are other smaller problems with this code. But for now, lets focus on fixing the problems that we identified above and making this use case a little bit better.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Here is an attempt to fix some of the problems we discussed above.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist 1e90438e3ae2c7aa8905977a751d74aa %}</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>First thing worth noticing is that these functions are not inside a <code>type</code> because there is no common dependency between these two functions. These functions exists on its own (under a module) and only accepts parameters that they can work with. The function name and signature are self documenting and they clearly communicate the intent. There is no scope of nasty exceptions bubbling up because of an unstable dependency. <strong><i>These are pure functions</i></strong>.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Lets see how easy it is to test these functions.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist d8c27b1a3b8ef64cb5549e89115e5fca %}</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Simple and straightforward. There is no need to setup mocks for different scenarios here. You can create test data to your hearts content and test this function out. I have used <a href="https://github.com/AutoFixture/AutoFixture">Autofixture</a>, but <a href="https://fsharpforfunandprofit.com/posts/property-based-testing/">Property based tests</a> are better suited for testing these functions.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Coming back to the referential transparency. The <code>find</code> and <code>update</code> functions are referentially transparent. For instance, if the <code>Product</code> exists in the list of products then <code>find</code> will always return the Product. Otherwise, it always returns <code>None</code>, consistently.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>So, we have seen how getting rid of impure operations tremendously simplifies the program. But wait, this new shiny pure functional program does nothing useful. The end goal of this program is to update dimensions in the <i>database.</i> How do we make sure that we stick to function purity but also be able to perform impure operations like persisting data in a database or calling a third party service etc? You need to do the following:</p>
<!-- /wp:paragraph -->

<!-- wp:list {"ordered":true} -->
<ol><li>Push IO (impure) functions at the boundary of your domain.</li><li>Call pure functions from the impure functions and not the other way round.</li></ol>
<!-- /wp:list -->

<!-- wp:quote -->
<blockquote class="wp-block-quote"><p>As long as an impure function calls a Pure function you are good. When a pure function calls an impure function then the whole method chain becomes impure.</p></blockquote>
<!-- /wp:quote -->

<!-- wp:paragraph -->
<p>You would have guessed where we are going with this kind of separation between pure and impure functions. OK, no points for guessing, moving impure functions at the boundary and calling pure functions from there naturally leads you to <a href="http://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html">clean architecture</a>.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Let's see how does the call to pure functions from an impure function looks like.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist e27a7abe2645023078015666fdf07b49 %}</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><br><code>tryUpdateProductDimensions</code>is an impure function because it calls <code>getAllProducts</code> which in turn calls <code>allProducts</code> from the <code>DB</code> module. <code>DB</code> module performs a bunch of impure operations. We can call it as the core of impurity (pun intended). For the sake of completeness, here is how I have implemented the DB module.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist 03d60370b3d794575affc1e505975c0c %}</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->
<h4>Conclusion</h4>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>For a developer who is not fully familiar with F# or the functional programming paradigm may quite easily fall into the pit of writing OO style code with F# just like me in the beginning of this article. In F#, separating impure functions from pure functions requires discipline. As I understand, F# has no in-built magic to prevent you from mixing pure and impure functions. Haskell in that respect is pretty strict because it forces you to wrap impure functions using an <a href="https://wiki.haskell.org/Introduction_to_IO">IO monadic system</a>. In simple terms an IO monad is an abstract data structure which elevates a value. Once the value is elevated you can no longer access the original value. To access the original value, you must use one of the mapping functions of the monad. I can write a similar IO monad in F#, but is it really worth the effort? The F# compiler would not honor such an IO monad. Though I think that it may be a good idea to enforce it as a coding convention for your projects.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Let me know in the comments section below what you think about the IO system in Haskell. Should F# support something similar? If yes, why and if no why not?</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->
