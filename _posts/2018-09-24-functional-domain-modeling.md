---
layout: post
title: Functional Domain Modeling
date: 2018-09-24 23:02
author: Ashish Vegaraju
comments: true
categories: []
---
<p style="text-align:justify;" data-mce-style="text-align: justify;">Lately, I've been reading about functional paradigm and related programming languages. More I read about it, more I feel that modeling domains in pure C# (or Java for that matter) is <em>unnatural</em> and does not communicate the intent of the domain clearly.</p>
<p style="text-align:justify;" data-mce-style="text-align: justify;">After so many years of writing C# code, that's quite a statement to make. But, I am serious. Lets model a simple subdomain <strong>Order</strong> for a hypothetical e-commerce company using F# then later using C# and see which of the models looks more expressive.</p>
<p style="text-align:justify;" data-mce-style="text-align: justify;">Lets start with C# first. Below is a typical <code>Order</code> entity with some typical properties.</p>
{% gist a0738e33f449f4fda4c11328293b5c95 %}
<p style="text-align:justify;" data-mce-style="text-align: justify;"><em>(For brevity, I have removed other properties &amp; methods which add behavior to this model.)</em></p>
<p style="text-align:justify;" data-mce-style="text-align: justify;">At first glance, this model looks decent. <em>Technically,</em> it has all the properties required to represent an&nbsp;<code>​​Order</code>. It uses <a href="https://martinfowler.com/bliki/UbiquitousLanguage.html" data-mce-href="https://martinfowler.com/bliki/UbiquitousLanguage.html">ubiquitous language</a> for modeling the domain, it uses strong types to represent attributes of an order (so, the smell of <a href="http://wiki.c2.com/?PrimitiveObsession" target="_blank" rel="noopener" data-mce-href="http://wiki.c2.com/?PrimitiveObsession">primitive obsession</a> is also taken care of). The properties expose only public getters, so state change (if we choose to do so) is only possible via public methods. But, does this model document the design of Order subdomain? Can a developer look at this model and understand what are the business constraints in this sub-domain? The answer is No. Let's examine this model closely to see why.</p>
<p style="text-align:justify;" data-mce-style="text-align: justify;"><strong>Lack of <em>Choice</em></strong></p>
<p style="text-align:justify;" data-mce-style="text-align: justify;">The <code>PaymentMethod</code> property is of type <code>IPaymentMethod</code>. Typically, this interface would be implemented by concrete payment method classes like <code>​CashOnDelivery</code>, <code>CreditCard</code> <code>Paypal</code> etc. But, this property does not <em>scream</em> about all the supported payment methods of the order sub-domain.</p>
<p style="text-align:justify;" data-mce-style="text-align: justify;">Of course, I can always see what are the types implementing <code>IPaymentMethod</code> interface, but, its well.. ehm... <em>unnatural</em> ...(you will see why when we examine the F# model). In short, the payments methods are not explicitly documented <em>in the</em> <em>model</em>. Another problem with having an interface is that it does not allow the domain modeler to make the property hold different <em>types (or combination of types)</em> of values depending on a particular payment method. For instance, in case of CreditCard, I would like the property to hold Card number <em>and</em> CardType information, if the payment method is PayPal then the property should hold the paypal Id etc.</p>
<p style="text-align:justify;" data-mce-style="text-align: justify;">Another problem, though minor, is that we do not know if we are dealing with a closed set of payment methods. What if an innocent developer makes an <em>innocent</em> mistake and adds <code>MyAwesomePaymentMethod</code> as another implementation of <code>IPaymentMethod</code>. So, now <code>Order</code> is secretly supporting a new payment method and no one (include the innocent developer) has a clue about it.</p>
<p style="text-align:justify;" data-mce-style="text-align: justify;">Here comes F# for the rescue. F# has discriminated unions. In other words a <em><strong>choice type,</strong></em> which, if used, unambiguously documents about the only three supported payment methods available. In other words, its a <em>closed set</em> of payment methods. Below is how the <code>PaymentMethod</code> type would be modeled in F#. Looking at this file, one can see the complete spec of <code>PaymentMethod</code> type.</p>
<em>
</em>

{% gist bfae189f33aad1cd24498df4230eb521 %}

If I want to print a payment method (Of course, I wouldn't from a domain model, perhaps I would execute a function for each <em>matched</em> payment method), I can <em>pattern match</em> and print (or do something more useful) in here.

{% gist f5c6040682597f7535eba60962a62ea8 %}
<p style="text-align:justify;" data-mce-style="text-align: justify;">Notice how pattern matching allows us to return different information depending on&nbsp; the <em>choice</em> of payment method. In case of Paypal, I can use the PaypalId. Similarly, in case of <code>Card</code> payment, I can use the card number and card type.</p>
<p style="text-align:justify;" data-mce-style="text-align: justify;">Coming back to C#, I can use pattern matching and try to mimic the F# <em>choice type</em> behavior. But its ugly and convoluted (IDK why I am even mentioning it here :D ).</p>
{% gist d53bccf85e229197c021c854b9b52437 %}

<strong>Stuck without <em>Option</em></strong>
<p style="text-align:justify;" data-mce-style="text-align: justify;">Let's take a good look at the C# domain model once again. Can you infer which properties are required and which are optional? Typically, an order may or may not have a discount code which makes the property <code>DiscountCode</code> optional. But, the C# model fails to document that constraint. You may argue that <code>DiscountCode</code> could be made <code>nullable</code> and then all is good. But, is <code>null</code> equivalent to <em>not having</em> a discount code?</p>
In F#, we can use <code>​Option&lt;'a&gt;</code> to denote an optional field as shown here:

{% gist b296ebfa02ef75274e9409eeca9c4a3a %}
<p style="text-align:justify;" data-mce-style="text-align: justify;">This is a self documenting domain model. By looking at it I know that an order may or may not have <code>DiscountCode</code>.</p>
<p style="text-align:justify;" data-mce-style="text-align: justify;">Luckily, <a href="https://github.com/louthy/language-ext" target="_blank" rel="noopener" data-mce-href="https://github.com/louthy/language-ext">Language Ext</a> package allows us to do this in C#. The <code>Order</code> model in C# (using Language Ext) now looks like this:</p>
{%gist 252c9d48dbb459594054edc01420786b %}
<p style="text-align:justify;" data-mce-style="text-align: justify;">Much better!</p>
As mentioned, an <code>​Option</code> may or may not have a value. This is denoted with <code>​Some</code> or​ <code>None</code> respectively. Below is how one can inspect / match an <code>Option</code> to see if it contains a <code>Some</code> or a <code>None</code><em>.</em>

<em>
</em>

{% gist eef9b73057ef1a9cdfc9708dfbf9ae5f %}
<p style="text-align:justify;" data-mce-style="text-align: justify;">That's a short one on why I feel that functional languages are naturally good for modeling domains as they make domain models expressive and self documenting. I am still experimenting with a couple of functional languages and so far I can see clear benefits of functional languages over imperative languages when it comes to developing a project with domain driven design.</p>
<p style="text-align:justify;" data-mce-style="text-align: justify;">I am not sure how easy or painful it is to access database or perform I/O with a functional language. I would probably run into it at some point. Do share your experiences (good and /or painful) with functional languages under comments and Thanks for reading!</p>
