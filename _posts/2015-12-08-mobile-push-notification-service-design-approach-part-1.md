---
layout: post
title: Mobile Push Notification Service - Design Approach (Part 1)
date: 2015-12-08 18:00
author: avegaraju
comments: true
categories: [APN, apple push notification service, architecure, cloud messaging services, design, Design &amp; Architecture, GCM, google cloud messaging service, mobile, push notification]
---
<h6><span style="font-weight:normal;">Overview of Push Notifications</span></h6>
<p align="justify">In this article, I am going to design the architecture for implementing push notifications for you mobile application. This part only has design and architecture information, code will be covered in Part 2.</p>

<blockquote>If you are new to push notifications, please read <a href="https://en.wikipedia.org/wiki/Notification_service">wiki </a>article about the technology and also about <a href="https://en.wikipedia.org/wiki/Google_Cloud_Messaging">GCM </a>and <a href="https://en.wikipedia.org/wiki/Apple_Push_Notification_Service">APN </a>services.</blockquote>
<!--more-->

Below diagram depicts a high level overview of device registration process.

<img src="https://ashishvegaraju.files.wordpress.com/2015/11/devicereg.png?w=559" alt="DeviceReg" width="1312" height="758" />
<ol>
	<li>
<div align="justify">Device sends sender ID, application ID to GCM server for registration.</div></li>
	<li>
<div align="justify">Upon registration Cloud Service (GCM/APNs) sends a unique registration ID to the device.</div></li>
	<li>
<div align="justify">After receiving the registration ID from the cloud service the device will forward the registration ID to the Push Notification Web Service.</div></li>
	<li>
<div align="justify">The Push Notification web service will store the registration ID in a local database for later use.</div></li>
</ol>
<p align="justify">a. Whenever Push Notification service needs to send the notification, it has to call GCM/APN service with registration ID of the device which is stored in the database.</p>
<p align="justify">b. GCM/APN service will deliver the notification to respective mobile device based on the registration ID.</p>

<blockquote>Push Notification service would require an outbound internet connection to send messages to cloud services. Make sure to open two outbound ports on your push notification server.</blockquote>
<h4><strong>Push Notification Components</strong></h4>
You would need to implement two components:
<ol>
	<li>
<h5><em><strong>Push Notification Web Service</strong></em></h5>
</li>
</ol>
<p align="justify">Push notification web service could be a rest service. Bare minimum the web service should have two functionalities. One, to register new device and Second, to unregister a device.</p>

<ol>
	<li>
<h5><strong><em>Push Notification Windows Service.</em></strong></h5>
</li>
</ol>
<p align="justify">You need to implement a windows service which could spawn multiple threads to listen to a messaging queue at a specified address. This service should be capable to pick up messages as and when it arrives to the messaging queue. The service will then query the local database and get a list of eligible devices which should receive the notification message and communicate with the cloud messaging services to get the notifications delivered.</p>

<blockquote>
<p align="justify">Push Notification Windows service is a Microsoft Windows<sup>® </sup>terminology. On non-windows platform this windows service will be a process which will be initiated with a start-up batch file which could be hooked up with tomcat initialization process.</p>
</blockquote>
<h4><strong>Push Notification Architecture</strong></h4>
Diagram below depicts the push notification architecture at high level.

<img src="https://ashishvegaraju.files.wordpress.com/2015/11/hld.png?w=559" alt="HLD" width="1313" height="924" />
<ol>
	<li>
<div align="justify">As depicted in the diagram above, the first step is get the unique device ID from the cloud messaging service. The mobile application should make a call to the cloud messaging service every time it is launched. It is important to make this call every time as it may so happen that the unique device ID assigned earlier has now expired.</div></li>
	<li>
<div align="justify">

Second step is to call the Push Notification web service from the mobile application and include the unique device ID (received from cloud messaging service) in the payload. You may want to pass some additional information like the device type, device operating system etc in the payload. A sample payload JSON is shown below:
<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:8deaa2ad-61ae-43d6-8179-78575cbdfd0d" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="javascript"]
DevicePayload:{

deviceId:'APA9sdsdA',

deviceType:'mobile/tablet',

deviceOS:'android/apple',

userID:'xyz@gmail.com',

}

[/sourcecode]

</div>
</div></li>
	<li>
<div align="justify">Push notification web service will persist the payload in a local database.</div></li>
	<li>
<div align="justify">Core product which is responsible for generating events per user will create notification messages and publish it to the Messaging Queue.</div></li>
	<li>
<div align="justify">Push notification windows service’s functionality is to constantly poll the Messaging Queue and pull any new message that arrives to the queue. The service should then query the local database to get the device id associated with the user id for which the notification was generated.</div></li>
	<li>
<div align="justify">With the fetched device id, the windows service will then communicate with cloud messaging services (GCM/APN).</div></li>
</ol>
<p align="justify">This is the architecture at high level to design push notification service for your mobile application. In the next article (Part-2), I will cover the code required to communicate with the cloud messaging services.</p>
