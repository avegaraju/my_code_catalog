---
layout: post
title: A WCF Chat Service & Client
date: 2015-12-15 17:33
author: avegaraju
comments: true
categories: [callback, chat, code challenge, Coding Challenge &amp; Competitions, coding competition, duplex, interview question, synchronizationcontext, wcf, WCF chat service]
---
<p align="justify">I attended a interview recently for which the pre-requisite was to develop a chat application using WCF as part of a <em>code challenge</em>. The requirement was quite simple, so instead of taking the usual route of creating a WCF service and adding its reference to a client application, I decided to hand code everything without any configuration or service references!</p>
<p align="justify">Of course the  assumption here is that the service and client applications are owned by a single company/team so that the contracts can be shared between the service and client.</p>
<p align="justify"><!--more--></p>
<p align="justify">The complete code is available in my <a href="https://github.com/avegaraju/WCFChatApp" target="_blank" rel="noopener">Github</a> account.</p>

<h6>The Problem Statement</h6>
<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:c247d9f7-42bd-467e-a56a-5bb236a92f5d" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="text" gutter="false" light="true"]
Goal - Create a Chat Server and Client application using 
.NET 4.0/4.5 with WCF as the communication mechanism. 
The Chat Server should be a standalone application and 
the Chat Client should be a standalone application with 
a basic UI for users to interact and exchange messages 
through the Chat server. Stability of the code is a 
must-have and it should be ensured that the 
Server/Client do not crash during operation to 
maintain quality user experience. 

The basic functionality to be supported is as follows: 

1. The chat server should open up a TCP port for
 listening to incoming client requests. 

2. The server should be able to handle at 
least 2 distinct clients’ communication in parallel. 

3. On successful connection with the server, 
the chat client should be able to send plain text 
messages to the server. 

4. The server should be able to respond to each 
message by a static message &quot;Hello!&quot; appended 
with the server's timestamp. 

5. Both chat clients should be able to communicate 
with the server without the messages of either being 
visible to the other one. 

[/sourcecode]

</div>
<h6 align="justify">The Approach</h6>
<p align="justify">Since the service contract has to be shared between the service and client the best place to create the service contract is in a common library project. I have created a “contract” folder which has few more sub folders having contracts for service, response, request,  fault and a contract for callback service (more on this later).</p>
<a href="http://ashishvegaraju.files.wordpress.com/2015/12/image.png"><img style="background-image:none;padding-top:0;padding-left:0;display:inline;padding-right:0;border-width:0;" title="image" src="http://ashishvegaraju.files.wordpress.com/2015/12/image_thumb.png" alt="image" width="299" height="370" border="0" /></a>
<h6>The Duplex Message Exchange Pattern via Callbacks</h6>
IEAChatService.cs is the service contract defined as shown below
<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:53f692e4-3830-4b21-bc23-0989a6240c9a" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="csharp"]
 [ServiceContract(
        SessionMode = SessionMode.Required, 
        CallbackContract= typeof(IChatCallback))]
 public interface IEAChatService
[/sourcecode]

</div>
<p align="justify">I have defined a CallbackContract for this service contract. This callback contract interface is defined in IChatCallback.cs and is implemented by the client application (EA.Client.EAChat.UI).</p>

<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:f5c8c95c-e1a9-4f44-83ac-bfef0512728e" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="csharp" padlinenumbers="true"]
    [ServiceContract]
    public interface IChatCallback
    {
        [OperationContract(IsOneWay=true)]
        void RefreshUserList(List&lt;User&gt; onlineUsers);

        [OperationContract]
        void ResponseMessage(ResponsePacket response);
    }
[/sourcecode]

</div>
<p align="justify">This contract is called as a duplex service contract. Duplex service contract is a message exchange pattern where service as well as client can send messages to other independently. Since this interface is implemented by client (EA.Client.EAChat.UI), the service can directly call the client methods. In this case, whenever a new user enters the chat room the <em>Register  </em>method of the EAChatService.cs calls the callback method <em>RefreshUserList </em>on the client, so that client can refresh the user interface and show the name of the newly entered user in the list of users of the chat room. Below is the relevant piece of code.</p>

<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:2389c65a-1bbb-4b08-ab09-c7210d0e443d" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="csharp"]
       public void Register(User userHandle)
        {
            try
            {
						//---irrelevant code removed ---------
                       foreach (KeyValuePair&lt;string, IChatCallback&gt; kvp in registeredUser)
                        {
                            IChatCallback callback = kvp.Value;
                            callback.RefreshUserList(onlineUsers);
                        }
                    }

                }
               // ---irrelevant code removed ---------

            }
            catch (UserHandleInUseException uEx)
            {
                log.Error(&quot;User handle is alread in use. Error Detail: &quot; + uEx.Message);
                throw;

            }

        }
[/sourcecode]

</div>
<h6 align="justify">The Service Side Code</h6>
EAChatService.cs inherits from ServiceBase.cs. ServiceBase class has some common properties used why the EAChatService class. For instance, the ServiceBase class has a protected getter property  which returns the channel to the client instance which called the operation currently. Below is the code snippet.
<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:5c2a9717-b10b-4593-8ad3-7fcf6c9cc15e" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="csharp"]
      protected IChatCallback Callback
       {
           get
           {
               return OperationContext.Current.GetCallbackChannel&lt;IChatCallback&gt;();
           }
       }
[/sourcecode]

</div>
&nbsp;
<p align="justify">ServiceBase class also has a protected dictionary for registeredUsers which stores the userID and the corresponding Callback as a keyvalue pair in the dictionary object. Since multiple users may call the <em>Register </em>method at once, the critical section of adding users in the dictionary is synchronized using a lock. below is the complete code of register method.</p>

<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:a2ece10f-4f44-4329-81ca-decd9b13b982" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="csharp"]
       public void Register(User userHandle)
        {
            try
            {
                if (!registeredUser.ContainsKey(userHandle.UserId) &amp;&amp;
                    !registeredUser.ContainsValue(Callback))
                {
                    lock (syncObject)
                    {
                        registeredUser.Add(userHandle.UserId, Callback);
                        onlineUsers.Add(userHandle);

                        foreach (KeyValuePair&lt;string, IChatCallback&gt; kvp in registeredUser)
                        {
                            IChatCallback callback = kvp.Value;
                            callback.RefreshUserList(onlineUsers);
                        }
                    }

                }
                else
                {
                    throw new UserHandleInUseException(ServiceFaultCode.HANDLE_IN_USE, &quot;User with this handle already exist. Please choose a different handle.&quot;);
                }
            }
            catch (UserHandleInUseException uEx)
            {
                log.Error(&quot;User handle is alread in use. Error Detail: &quot; + uEx.Message);
                throw;

            }

        }
[/sourcecode]

</div>
<h6>The Client Code</h6>
<p align="justify">The client code is implemented in the <em>ChatWindow</em> cliass. The <em>ChatWindow</em>  class has a SynchronizationContext which is used to marshal messages from one thread to another thread. You must be wondering, why do we need to marshal messages from one thread to another. The reason is that the client UI runs on a separate thread and the duplex message from the WCF service is delivered to the client application via another thread. Now the message sent from WCF service as part of duplex call cannot be directly used by the client UI. To enable the message transmission between the UI thread and callback thread, we use the UI thread’s SynchronizationContext and call its <em>Post</em> method to pass the message. I am using Unity container to hold and resolve instances of SynchronizationContext object. Below is the implementation of <em>RefreshUserList</em> callback method which demonstrates the SynchronizationContext usage.</p>

<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:935a3bc5-6dc3-412f-984e-6869b3b190f6" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="csharp" padlinenumbers="true"]
       public void RefreshUserList(List&lt;User&gt; onlineUsers)
        {
            ClearOnlineUserList();

            foreach (User user in onlineUsers)
            {

                SendOrPostCallback callback =
                delegate(object state)
                {
                    this.OnlineUserListBox.Items.Add(user.UserId);
                };

                _uiSyncContext = (SynchronizationContext)container.Resolve(typeof(SynchronizationContext));
                _uiSyncContext.Post(callback, user.UserId);
            }
        }
[/sourcecode]

</div>
<h6>The WCF endpoint &amp; Hosting</h6>
<p align="justify">The WCF endpoint is configured programmatically in the EA.Host.ServiceHostEngine project where we have the logic of hosting the WCF service as well.</p>
<p align="justify">I have defined an Interface IHostScheme.cs which has the declaration of  <em>Host </em>method and this interface is implemented by NetTcpHost.cs. Since one of the assumptions is that this chat application works in intranet, I choseto use NetTcp binding for the communication. In future if we decide to make to this chat application work in extranet then we can create another class implementing the IHostScheme interface with appropriate binding implementation.</p>
<p align="justify">Lets look at the NetTcpScheme.cs class.</p>

<div id="scid:C89E2BDB-ADD3-4f7a-9810-1B7EACF446C1:a2203c92-2db1-433f-b25b-1f9d25cc64d8" class="wlWriterEditableSmartContent" style="float:none;margin:0;display:inline;padding:0;">

[sourcecode language="csharp"]
    public class NetTcpHost : IHostScheme
    {
        public ServiceHost Host(string portNumber, bool enableMetaData)
        {
            Type contractType = typeof(IEAChatService);
            Type serviceType = typeof(EAChatService);

            Uri baseAddress = GetBaseAddress(portNumber);

            ServiceHost serviceHost = new ServiceHost(serviceType, baseAddress);
            serviceHost.AddServiceEndpoint(contractType, new NetTcpBinding(), baseAddress.AbsoluteUri);

            ServiceMetadataBehavior metaDataBehavior = new ServiceMetadataBehavior();
            metaDataBehavior.HttpGetEnabled = false;
            serviceHost.Description.Behaviors.Add(metaDataBehavior);

            if (enableMetaData)
            {
                serviceHost.AddServiceEndpoint(ServiceMetadataBehavior.MexContractName,
              MetadataExchangeBindings.CreateMexTcpBinding(), &quot;mex&quot;);
            }

            return serviceHost;
        }

        private Uri GetBaseAddress(string portNumber)
        {
            string prefix = ConfigurationManager.AppSettings[&quot;nettcp_prefix&quot;];
            string chatServicePath = ConfigurationManager.AppSettings[&quot;EAChatServicePath&quot;];

            return new Uri(prefix + portNumber + chatServicePath);

        }
    }
[/sourcecode]

</div>
<p align="justify">The code is pretty much self explanatory. This class is responsible for creating a ServiceHost and a ServiceEndpoint. To create a ServiceHost, we need the type of Service contract and to create a service endpoint which will be hosted by the ServiceHost, we need the contract’s type, address (along with the port) where we will host the service and binding is set to NetTcpBinding because that’s one of the requirement in the problem statement that communication has to take place over tcp port.</p>
<p align="justify">I have made HttpGetEnabled property of ServiceMetadataBehavior to false, as I don’t se any benefit of exposing the wsdl over http for our application. However, MEX endpoint is made configurable, so that if in future we wish to create a new client and create a proxy using svcutil for communication, we could do so easily.</p>
<p align="justify">That’s it! There are other utility projects and relatively less interesting code in the solution, so I will skip the remaining details.</p>
<p align="justify">Feel free to use the code and modify it according to your requirements. The code is available  in my <a href="https://github.com/avegaraju/WCFChatApp" target="_blank" rel="noopener">Github</a> account.</p>
If you have any queries, feel free to ask in the comments section.
