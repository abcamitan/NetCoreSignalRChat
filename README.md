# Chat Web App
Simple Chat app using ASP.Net Core MVC and  SignalR Core

## Resource

This is based on docs from Asp Core website:
https://docs.microsoft.com/en-us/aspnet/core/signalr/?view=aspnetcore-2.1

Please use below reference, for doing the JavaScript connection from your web page:
https://docs.microsoft.com/en-us/aspnet/core/signalr/javascript-client?view=aspnetcore-2.1

## Getting Started
Download the latest stable version of .Net Core, in this project we used .Net Core 2.0.7 version. We are going to use SignalR (1.0.0-rc1-final).


## Below are the steps to do this
1) Run below from command prompt:
dotnet new razor -au Individual --name NetCoreSignalRChat

2) Go to NetCoreSignalRChat directory
cd NetCoreSignalRChat

3) Modify the NetCoreSignalRChat.csproj project file, then add the following lines under ItemGroup.PackageReference section:

<PackageReference Include="Microsoft.AspNetCore.SignalR" Version="1.0.0-rc1-final" />
<PackageReference Include="Microsoft.AspNetCore.SignalR.Client" Version="1.0.0-rc1-final" />

Then, run below command.

dotnet restore

4) Go to Pages directory to create the page for your chat app.
cd Pages

5) Run below to create the page automatically.
dotnet new page --name Chat

6) Open and update the Chat.cshtml.cs file.

using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace NetCoreSignalRChat.Pages
{
    [Authorize]
    public class ChatModel : PageModel
    {
        public void OnGet()
        {
        }
    }
}

7) Open and update the Chat.cshtml file.

@page
@model ChatModel
@{
}

<h1>Chat</h1>

<form id="send-form" action="#">
    Name:
    <input type="text" id="name" disabled />
    Send a message: 
    <input type="text" id="message-textbox" disabled /> 
    <button id="send-button" type="submit" disabled>Send</button>
</form>

<ul id="messages-list">
</ul>

@section Scripts {
    <script src="~/lib/signalr/signalr.js"></script>
    <script type="text/javascript">
        // Bind DOM elements
        var sendForm = document.getElementById("send-form");
        var sendButton = document.getElementById("send-button");
        var messagesList = document.getElementById("messages-list");
        var messageTextBox = document.getElementById("message-textbox");

        function appendMessage(content) {
            var li = document.createElement("li");
            li.innerText = content;
            messagesList.appendChild(li);
        }

        const connection = new signalR.HubConnectionBuilder()
            .withUrl("/hubs/chat")
            .configureLogging(signalR.LogLevel.Information)
            .build();

        sendForm.addEventListener("submit", function(evt) { 
            var message = messageTextBox.value;
            messageTextBox.value = "";
            connection.send("Send", message);
            evt.preventDefault();
        });

        connection.on("SendMessage", function (sender, message) {
            appendMessage(sender + ': ' + message);
        });

        connection.on("SendAction", function (sender, action) {
            appendMessage(sender + ' ' + action);
        });

        connection.start().then(function() {
            messageTextBox.disabled = false;
            sendButton.disabled = false;
        })
        .catch(err => console.error(err.toString()));
    </script>
}

8) Go back to the main folder, run below to go back:
cd ..

9) Create new Folder name in Hubs. 
Run below code on Linux or Mac:
mkdir Hubs

Run below code on Windows:
md Hubs

10) Go to Hubs folder, by running below:
cd Hubs

11) Create new file called ChatHub.cs using your favorite file editor. Then, copy and paste below code then save:

using System;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Authorization;
using Microsoft.AspNetCore.SignalR;

namespace NetCoreSignalRChat.Hubs
{
    [Authorize]
    public class ChatHub : Hub
    {
        public override async Task OnConnectedAsync()
        {
            await Clients.All.SendAsync("SendAction", Context.User.Identity.Name, "joined");
        }

        public override async Task OnDisconnectedAsync(Exception ex)
        {
            await Clients.All.SendAsync("SendAction", Context.User.Identity.Name, "left");
        }

        public async Task Send(string message)
        {
            await Clients.All.SendAsync("SendMessage", Context.User.Identity.Name, message);
        }
    }
}

12) Go back to the main folder. Find and update startup.cs. Using below codes.
Under the "using" section add below code:

using NetCoreSignalRChat.Hubs;

Find the ConfigureServices method, then add the codes below:

services.AddCors(options => options.AddPolicy("CorsPolicy", 
builder => 
{
    builder.AllowAnyMethod().AllowAnyHeader()
            .WithOrigins("http://localhost:5000")
            .AllowCredentials();
}));

services.AddSignalR();

Find the Configure method, then add the following code:

app.UseCookiePolicy();

app.UseSignalR(routes =>
{
    routes.MapHub<ChatHub>("/hubs/chat");
});

Lastly, save the file.

13) Run below command to install the signalr javascript file for the client app.

npm install @aspnet/signalr

14) Copy the signalr.js file under node_modules/@aspnet/signalr/dist/browser folder. Then, paste it under wwwroot/lib/signalr folder.

15) To test if it running properly, run below command:

dotnet run