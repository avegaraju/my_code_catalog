---
layout: post
title: Test doubles ; Over the wire
date: 2019-03-01 00:09
author: avegaraju
comments: true
categories: [fakes, integration testing, Microservice testing, mocks, stubs, test doubles over the wire, test driven development]
---
<!-- wp:paragraph {"customFontSize":12} -->
<p style="font-size:12px;"><strong>Please note</strong>: If the code snippets in this article are not visible on your mobile browser then try to view this article with <em>Desktop site</em> option enabled.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph {"dropCap":true} -->
<p class="has-drop-cap">Developing microservices is complex and testing its integration is even  more complex. Assuring availability of all the inter-dependent services with up-to-date builds in the test environment is quite a tedious &amp; costly setup to maintain. Stubs and mocks on the other hand are cheap, easily maintainable and they make integration tests predictable and repeatable. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Almost two years ago, we faced a problem at work where we were desperately looking for a solution to test multiple Web API interactions with an external system over HTTP. In short, we wanted a hassle free mechanism to stub and verify invocation of an external service. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>We researched and came across <a href="http://www.mbtest.org/">Mountebank</a>, which was not only painful to configure, but was error prone too. To top it all off, configuring dynamic behavior required enabling javascript injection. No points for guessing, the security team of my company outrightly rejected the request to enable javascript injection on any of our servers. Not to mention, no clean way to deploy and integrate it with our CI/CD pipelines. Later, due to various constraints, we decided to spend our time on delivering new features instead of solving this problem. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Feeling dejected, I started <a href="https://github.com/avegaraju/Imposter">imposter</a> project in my free time with an intent to develop a simple, developer friendly stubbing and mocking mechanism. The project is now in a decent enough state to deserve a blog article.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Without further ado, in this article I will demonstrate how to create and configure stubs &amp; mocks using the <a href="https://github.com/avegaraju/Imposter">imposter</a> project. Let's begin with configuring and creating stubs.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3>Configuring &amp; creating stubs</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>To configure a stub, you need to reference <a href="https://www.nuget.org/packages/Appify.FluentImposter.AspnetCore/">FluentImposter.AspNetCore package</a>. This package contains an ASP.Net middleware <code>UseStubImposters</code>, to configure stubs. All you need to do is to configure it in the <code>Startup.cs</code> as shown below:</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist 2e0e4c12d21a0a15d706fc6b0a514161 %}</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p><code>ImpostersAsStubConfiguration</code>, as the name suggests, accepts a list of imposters-as-stubs. These imposters are hosted as REST endpoints by the middleware. To demonstrate, let's create an imposter which will stub the <code>Orders</code> REST endpoint. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist 505104b11cb45fda1bd0e7dca0b68f48 %}</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Being a fan of fluent APIs, I made the <code>ImposterDefinition</code> fluent. I like the idea of an API guiding me about the next steps. To begin with, it guides to define a resource to be stubbed, followed by a set of conditions that must match for the imposter to accept a request and respond with a canned response.  In this example, if the request body contains <code>Product:1234</code> then  this imposter will return a pre-configured response.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":4} -->
<h4>Creating Pre-configured responses.</h4>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The <code>ResponseBuilder</code> helps to build the canned response. The code is pretty straight forward. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist 9c71b7d3b946211332bc51220b67737b %}</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>As shown in the below POSTMan screenshot, the <code>Orders</code> imposter endpoint is responding with a preconfigured response and a <code>Created (201)</code> status code because the request body has the content <code>Product:1234</code>. </p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":676} -->
<figure class="wp-block-image"><img src="https://ashishvegaraju.files.wordpress.com/2019/02/image.png" alt="" class="wp-image-676" /><figcaption>The screen shot is of PostMan invoking Orders imposter and receiving a pre-configured response.</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>Let's move on to configure and create an imposter as a mock.</p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3>Configuring &amp; creating mocks</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The same package<a href="https://www.nuget.org/packages/Appify.FluentImposter.AspnetCore/">&nbsp;FluentImposter.AspNetCore package</a>  has another middleware <code>UseMockImposters</code>, which is capable of configure and hosting mocks. Example shown below:</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist 02cf07d5aa9fa748e009e99dbfc44bd6 %}</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Mocking requires invocation verification, hence a backing store is needed to keep the count of REST endpoint invocations. At the time of writing this article, I've only added support for <a href="https://aws.amazon.com/dynamodb/">Amazon DynamoDb</a> as a backing store which can be enable by adding   <br><a href="https://www.nuget.org/packages/Appify.FluentImposter.DataStore.AwsDynamoDb/">Appify.FluentImposter.DataStore.AwsDynamoDb</a> Nuget package in your project. In case you do not have an Amazon AWS account, you can work with <a href="https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBLocal.html">local DynamoDb installation</a> as well. But be aware that local DynamoDb, as the name suggests, is good for local testing and not recommended for production use. </p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Below is how you can create a MocksDatStore which connects to local DynamoDb installation.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist 8c050b7fafdd808e515be8355cc92c10 %}</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>I wouldn't go into the details of this datastore. All that the <code>MocksDataStore</code> should do is to return <code>AwsDynamoDbDataStore</code> instance which does all the magic of maintaining request invocations.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>The next step is to create Imposters for the REST endpoints. To demonstrate, lets create a mock for <code>api/Products</code> endpoint.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>{% gist 90378aac89406d5d6023a60635916969 %}</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>Let's spin up POSTMan and invoke <code>api/Products</code> endpoint.</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p>As shown in the screen shot, the imposter responded with a canned response.</p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":688} -->
<figure class="wp-block-image"><img src="https://ashishvegaraju.files.wordpress.com/2019/02/image-2.png" alt="" class="wp-image-688" /><figcaption>POSTMan screenshot of invoking api/Products endpoint.</figcaption></figure>
<!-- /wp:image -->

<!-- wp:heading {"level":4} -->
<h4>Mock verification</h4>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>The <code>UseMockImposters</code> middleware hosts a mock verification endpoint with resource path <code>mocks/verify</code> accepting Http GET requests. The request body should contain the details of the REST endpoint for which invocation verification has to be done.  Below is how the invocation verification for <code>api/Products</code> endpoint can be done using POSTMan</p>
<!-- /wp:paragraph -->

<!-- wp:paragraph -->
<p></p>
<!-- /wp:paragraph -->

<!-- wp:image {"id":689} -->
<figure class="wp-block-image"><img src="https://ashishvegaraju.files.wordpress.com/2019/02/image-3.png" alt="" class="wp-image-689" /><figcaption>Verification of invocation using POSTMan</figcaption></figure>
<!-- /wp:image -->

<!-- wp:paragraph -->
<p>The mock verification endpoint responds back with the resource invoked along with the total number of invocations so far. </p>
<!-- /wp:paragraph -->

<!-- wp:heading {"level":3} -->
<h3>Conclusion</h3>
<!-- /wp:heading -->

<!-- wp:paragraph -->
<p>Thanks for reading. I am actively working on this project from last few months and planning to support more protocols. If you have any suggestion or improvements for the project then do let me know via comments. If you want to contribute then feel free to create pull requests in <a href="https://github.com/avegaraju/Imposter">github</a>. </p>
<!-- /wp:paragraph -->
