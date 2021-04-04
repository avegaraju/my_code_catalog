---
layout: post
title: A Thread Safe Singleton Class in .NET
date: 2015-12-08 17:32
author: avegaraju
comments: true
categories: [.net, c#, csharp, Design &amp; Architecture, double check pattern, Singleton, thread safe]
---
<p align="justify">In this post I will demonstrate creating a singleton class in .Net that is thread safe. As usual the singleton class should have a private constructor as shown below so that any other class cannot “directly” create an instance of our singleton class.</p>
<p align="justify"><!--more--></p>
<p align="justify">Apart from that, I have created two global variables, one to hold the instance of the Singleton class and another one of type object which will be used later for synchronization between threads (more on that later).</p>

<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:fa80e900-39bf-485c-b945-fc9e5bb5939c" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="csharp" padlinenumbers="true" light="true" wraplines="true"]
    static SingletonSample _instance;
    static readonly object _synchronizer = new object();
    /// &amp;amp;lt;summary&amp;amp;gt;
    /// Private C'tor, so that this class cannot be instantited by other classes directly
    /// &amp;amp;lt;/summary&amp;amp;gt;
    private SingletonSample()
    {

    }

[/sourcecode]

</div>
<h6>Method 1 – Using synchronization lock</h6>
&nbsp;
<p align="justify">GetInstance is a public static method, this method is the only way by which any other class can get instance of the singleton class.</p>
<p align="justify">Below code snippet uses double check pattern before creating the instance of the singleton class. The first If statement, if true, will give the instance of Singleton class instantly. However, if _instance variable is null then program will enter a critical section (the lock statement) synchronized using a global object (_synchronizer). Inside the critical section, I am checking the _instance  variable again if it is null (double check), if it is, then we finally create the instance of the class.</p>
<p align="justify">One interesting point to note here is, I am not creating the instance directly and returning it back. I am first assigning the class reference into a temporary object of type SingletonClass (tempInstance) and then doing a  <a href="https://msdn.microsoft.com/en-us/library/system.threading.volatile(v=vs.110).aspx">Volatile</a> write on the actual instance variable (_instance).</p>
&nbsp;

&nbsp;
<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:2559210c-f730-46f3-821d-6bb73ce1632b" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="csharp"]
        /// &amp;amp;lt;summary&amp;amp;gt;
        /// Method is responsible for providing instance of this class.
        /// &amp;amp;lt;/summary&amp;amp;gt;
        /// &amp;amp;lt;returns&amp;amp;gt;Instance of SingleTonSample class&amp;amp;lt;/returns&amp;amp;gt;
        public static SingletonSample GetInstance()
        {
            //If instance is available return that.
            if (_instance != null)
                return _instance;

            //Critical section. Ony one thread can enter at a time.
            lock(_synchronizer)
            {
                //Check again if the instance in null
                if(_instance == null)
                {
                    SingletonSample tempInstance = new SingletonSample();
                    //Volatile write to make sure that _instance is 
                    //populated with tempInstance reference without any compiler optimization.
                    Volatile.Write(ref _instance, tempInstance);
                }
            }

            return _instance;
        }
[/sourcecode]

</div>
<p align="justify">The problem with this approach is that if this method is used in a highly parallel system, the critical section will cause all threads to do nothing but wait for an instance of Singleton class to be created. However, this will happen only once when the first instance is getting created, but still, this is wasteful of the resources.</p>
<p align="justify">Below is another, more efficient way of creating a singleton which neither uses double check pattern nor it has a critical section for thread synchronization.</p>

<h6>Method 2 – Without Critical Section or Double Check</h6>
<p align="justify">Below method will allow multiple threads to create instance of Singleton class, but <a href="https://msdn.microsoft.com/en-us/library/system.threading.interlocked(v=vs.110).aspx">Interlocked</a> class’s CompareExchange will make sure that only one instance is assigned to _instance variable. Objects created by other  threads and not assigned to _instance variable will soon become Orphan and  will be garbage collected at a later point of time by the CLR.</p>

<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:3c6dd858-1ab2-41bc-9bfa-fc04938077a4" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="csharp"]
        /// &amp;amp;lt;summary&amp;amp;gt;
        /// Method is responsible for providing instance of this class without critical section
        /// or double check pattern.
        /// &amp;amp;lt;/summary&amp;amp;gt;
        /// &amp;amp;lt;returns&amp;amp;gt;Instance of SingletonSample class&amp;amp;lt;/returns&amp;amp;gt;
        public static SingletonSample GetInstance()
        {
            //If instance is available return that.
            if (_instance != null)
                return _instance;

            SingletonSample tempInstance = new SingletonSample();
            /*Multiple threads will create singleton object, but only 
            *one will be asssgined to _instance variable. Rest others will be 
            *GC'd (Garbage Collected) later at some point of time. 
            */
            Interlocked.CompareExchange(ref _instance, tempInstance,null);

            return _instance;
        }
[/sourcecode]

</div>
Both methods are thread safe. However, method 2 is marginally more efficient and lean as compared to the first one.
