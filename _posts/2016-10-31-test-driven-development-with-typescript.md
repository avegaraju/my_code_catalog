---
layout: post
title: Test Driven Development with TypeScript
date: 2016-10-31 00:05
author: avegaraju
comments: true
categories: [chutzpah, javascript, mocking, QUnit, stubbing, TDD, test driven development, typemoq, typescript, unit test]
---
<p align="justify">In this article, I will demonstrate how to write testable code using TypeScript. I assume that the reader is familiar with JavaScript, heard about Typescript and has done or secretly desires to do test driven development.</p>
<p align="justify">There is an excellent <a href="https://www.typescriptlang.org/docs/tutorial.html">quick start guide</a> on Typescript that I recommend you to go through before moving ahead.</p>
<p align="justify"><!--more--></p>

<h2><strong>Project Setup</strong></h2>
I am writing the code in Visual Studio 2015 Update 2 (which already has the necessary ingredients to author TypeScript applications). You can also use Visual Studio Code for authoring web applications with TypeScript.
<p align="justify">1. Start Visual Studio and Create a new project. From the installed templates, select <strong>Html Application With TypeScript</strong> template.</p>
<p align="justify"><a href="http://ashishvegaraju.files.wordpress.com/2016/10/project_template.png"><img style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" title="Project_template" src="http://ashishvegaraju.files.wordpress.com/2016/10/project_template_thumb.png" alt="Project_template" width="521" height="365" border="0" /></a></p>
<p align="justify">2. Notice how Visual Studio has done the required scaffolding for your first TypeScript application. In fact, if you press F5 on your keyboard, you will be able to run this application in your browser. Take some time here and see how closely TypeScript resembles an object oriented programming language. You’d see a class, a constructor method etc.</p>
<p align="justify">3. When you are done with admiring the beauty of TypeScript, you may select the code in <strong>app.ts</strong> and delete it. Since we are doing Test Driven Development, we will not be writing any Production code without accompanying tests!</p>
<p align="justify">4. In an enterprise project, you’d usually create a new Test Project in the solution. But for the purpose of this article and also to keep things simple in the beginning, I will just create a new folder called as “<strong>Tests</strong>” under the same project and add a test file in it. .</p>
<p align="justify">The goal of the application is to calculate area of different shapes. I usually choose simple applications and simple domains to demonstrate new stuff. This helps in focusing more on the subject rather than focusing on details of the domain.</p>
<p align="justify">At this point, I have renamed <strong>app.ts</strong> to <strong>Rectangle.ts</strong> and the test file name is <strong>RectangleTests.ts</strong></p>
<a href="http://ashishvegaraju.files.wordpress.com/2016/10/image2.png"><img style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" title="image" src="http://ashishvegaraju.files.wordpress.com/2016/10/image_thumb2.png" alt="image" width="409" height="271" border="0" /></a>
<h2>Set up for writing Tests</h2>
<p align="justify">To test TypeScript code, you’d need a testing framework. There are few popular frameworks available like Jasmine and QUnit. For the purpose of this article and also because at the time of writing this article, I couldn't find any example where typescript is used with QUnit and Chutzpah (more on that later), I decided to write this article using QUnit.</p>
<p align="justify">Go ahead and add Qunit-MVC via the package manager console.</p>

<div id="codeSnippetWrapper" class="csharpcode-wrapper">
<div id="codeSnippet" class="csharpcode">
<pre class="alteven"><span id="lnum1" class="lnum">   1:</span> Install-Package Qunit-MVC</pre>
<!--CRLF-->

</div>
</div>
<p align="justify">You’d also need a type definition file for Qunit. The purpose of type definition file (file with extension *.d.ts) is to let TypeScript know about the types that exist in an API. There is a github project called as <a href="https://github.com/DefinitelyTyped/DefinitelyTyped">Definitely Typed</a>,  which provides high quality type definitions for almost all JavaScript APIs.</p>
<p align="justify">Run below command to add the type definition file for QUnit.</p>

<div id="codeSnippetWrapper" class="csharpcode-wrapper">

<!--CRLF-->

</div>
<blockquote>
<p align="justify">Install-Package qunit.TypeScript.DefinitelyTyped</p>
</blockquote>
<p align="justify">Apart from that, you’d need a test runner, where you can see your passing/failing tests. For that I am going to use <a href="https://github.com/mmanela/chutzpah">Chutzpah</a>. To use Chutzpah in Visual studio you need to add an extension. Select <strong>Tools</strong>&gt;<strong>Extensions and Updates </strong>and search for Chutzpah. (Since I already have Chutzpah extension added in my VS, I see the option to disable or uninstall it.)</p>
<a href="http://ashishvegaraju.files.wordpress.com/2016/10/image.png"><img style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" title="image" src="http://ashishvegaraju.files.wordpress.com/2016/10/image_thumb.png" alt="image" width="526" height="371" border="0" /></a>
<p align="justify">Add chutzpah.json file in the root of your project. This json file provides a range of configuration for Chutzpah test runner. For the sake of simplicity, I am configuring the json file with very basic settings:</p>

<p>{% gist 39c4b175b706121f1a7fef34eab791e1 %}</p>

<p align="justify">Compile Mode <strong>External</strong> denotes that Chutzpah test runner can expect compiled .JS file in the default output directory. If you select mode as <strong>Executable </strong>then you need to provide a powershell or bat file which will convert .ts files to .js file. In this project, I will use tsconfig.json file and configure it in such a way that visual studio will take care of generating a .js file while building (ctrl + shift + b) the project.</p>
<p align="justify">Add a file with name <strong>tsconfig.json</strong> in the root of the project with below settings.</p>

<p>{% gist 7abfdf6818481183519962ce7c846e6e %}</p>

With that in place, we are all set to write the tests.
<h2>The <span style="color:#ff0000;">Red</span> <span style="color:#008000;">Green</span> <span style="color:#0000ff;">Refactor</span> Cycle</h2>
Open <strong>RectangleTests.ts</strong> file and write you first failing test. Below is my first failing test.

<p>{% gist 857c540752b3708614a7e47a807dd727 %}</p>

There is no need to run tests. The project will not compile because I have not created the Rectangle class yet. A non-compiling class is equivalent to a failing test. So lets write just enough code to make this test pass.

Open <strong>Rectangle.ts</strong> file and type below code:
<p>{% gist d80aba54cec4112de1ab75e811c74f00 %}</p>

Perfect! This is exactly what we need to make the failing test pass. Lets run the test and confirm if our test is passing.

<a href="http://ashishvegaraju.files.wordpress.com/2016/10/image3.png"><img style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" title="image" src="http://ashishvegaraju.files.wordpress.com/2016/10/image_thumb3.png" alt="image" width="419" height="182" border="0" /></a>

So that’s our Red and Green cycle in TDD. But, what about Refactor? As of now, I do not see any need to refactor the code.
<p style="text-align:justify;">Let’s write the next test. I would like the length and breadth of the rectangle to be passed as constructor argument to the Rectangle class, so lets add another test which passes length and breadth to the Rectangle class.</p>
<p style="text-align:justify;">You'd notice that the project is not compiling because there is no constructor in the Rectangle class which accepts two arguments. To make the project compile, lets add a constructor to the Rectangle class:</p>

<p>{% gist 8006adf0eaf30604a3d7bf2a6e7b4bf6 %}</p>
 
Now lets run the tests again.

But hey! Our first test is now failing! Because we do not have any blank constructor in the Rectangle class. At this point we need to do some <span style="color:#1a1a1a;">refactoring</span>!

Do we really need the first test? I would say <strong>No</strong>, because with the new constructor, we are enforcing a contract that to calculate area of a Rectangle you need to provide length and breadth of the rectangle before hand. So lets <strong>delete </strong>the first test.
<blockquote><span style="color:#1a1a1a;">During the refactoring cycle, not only the production code, but the test code can and should be refactored. </span>
<p align="justify">Tests deserve to be maintained to the same level of quality as the production code. Indeed, perhaps the tests deserve even more attention than the production code since the quality of the production code depends on the tests;</p>
<p align="right">-Bob Martin</p>
</blockquote>
At this point, I would skip demonstrating all the Red Green and refactor cycles that I went through to add the function which calculates the area of a rectangle. The reason I want to skip that is because I want to demonstrate an interesting concept called as “Stubbing”!
<h2>Stubbing</h2>
For those who are familiar with test driven development would know that unit tests without the ability to stub external dependencies are not of much use. To demonstrate stubbing in our example, I am introducing an external dependency, a webservice which helps in calculating the area of a triangle.
<blockquote>Area of Triangle = 1/2 (height * base)</blockquote>
In our hypothetical world, there exists a service, which if called, provides the height of the triangle. The requirement is that we call a web service to get the height value.

To enable stubbing, we need to add typeMoq framework to the project. Using typeMoq, we can stub external dependencies and also use mocking to verify expectations on method calls.
<blockquote>
<p align="justify">Install-Package typemoq</p>
</blockquote>
Let’s create a class with name TriangleTests.ts and write the first failing test.

<p>{% gist 665ea1805d5ef5a2055d99a972e693d8 %}</p>
This class fails to compile, so lets quickly add Triangle.ts class with a constructor accepting base value:
<p>{% gist 9af785f45ad02aabad7a256d9e0954f9 %}</p>
Now, the tests should pass. Next, we need to somehow call a service from this class to get height of the triangle. This means, we need to add a new class which does the Ajax call to a hypothetical service. With that information, lets write our next test.

<p>{% gist b0cc4d4a6c38406abb006840cdf57d60 %}</p>

Notice the usage of mock.object. At this point the project will not compile, so lets fix it and make the tests run. Add an interface in Triangle.ts file.
<p>{% gist 140f24a92ce52eb0fa8f85442d14e58f %}</p>

and an empty class which implements this interface
<p>{% gist 40f0de21969bb8bc445205cd12e12e48 %}</p>

Notice that we are not writing any implementation for this class. In fact, any attempt to invoke the performAjaxCall method on this class will throw an exception. At this point, we just want our test to pass which is using the mock object to stub an external dependency.

Lets modify the Triangle class and add a new constructor argument:
<p>{% gist aeadc011c987d21f9c473568492fd134 %}</p>
Notice the usage of IAjaxWrapper interface in the constructor argument, just like how its done in any object oriented language. Add below references in the test class. With these references, we are telling Chutzpah test runner where to find the typemoq library.
<p>{% gist 294bb8f0958321a380adbe17abd2e234 %}</p>
With typemoq referenced in the test class, lets define our mock object.

<p>{% gist 1c720bd0d64763f1cdb0f550c45690d8 %}</p>

Below is the complete code:
<p>{% gist ef8bd15ad5854a7cf3ec4cffc6425919 %}</p>
This test will pass!

Let’s write the next test to actually calculate the area of triangle.
<p>{% gist ae4c98ec4387da063db5398ad97ca6a3 %}</p>

<p align="justify">Notice how the performAjaxCall method is being set up to return a predefined height value. At this point, we do not care how AjaxWrapper is going to fetch the value of triangle’s height, all we know is that the performAjaxCall method is setup to provide that value. Notice we just need an interface, so that we can handle the dependency with fake implementation and move ahead with our tests.</p>
<p align="justify">Of course, this test will fail, because calculateArea method does not exist. Let’s fix that and make the test pass. Create a calculateArea method in the Triangle.ts class. Below is the code:</p>

<p>{% gist 97b608c380d51888cd808cc4c571d8bb %}</p>

This should be enough for the tests to pass.

With that, I’d conclude this article. Hope the article was helpful.
<h2>Quick Tips</h2>
1. Chutzpah can show test results on a web page. To view your tests in a browser, right click on the test file and from the context menu Select <strong>Run Chutzpah with &gt; Debugger. </strong>You can also put breakpoints and debug your tests.
<p align="justify"><a href="http://ashishvegaraju.files.wordpress.com/2016/10/image4.png"><img style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" title="image" src="http://ashishvegaraju.files.wordpress.com/2016/10/image_thumb4.png" alt="image" width="268" height="136" border="0" /></a></p>
2. You can check the code coverage of your TypeScript code. Below is the code coverage of TriangleTests.ts

<a href="http://ashishvegaraju.files.wordpress.com/2016/10/image5.png"><img style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" title="image" src="http://ashishvegaraju.files.wordpress.com/2016/10/image_thumb5.png" alt="image" width="218" height="218" border="0" /></a>

The code for this demonstration can be found at my <a href="https://github.com/avegaraju/TDDWithTypeScript" target="_blank" rel="noopener">github repository</a>.
