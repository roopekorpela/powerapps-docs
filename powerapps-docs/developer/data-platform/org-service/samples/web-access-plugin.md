---
title: "Sample: Web access from a plug-in (Microsoft Dataverse) | Microsoft Docs" # Intent and product brand in a unique string of 43-59 chars including spaces
description: "Learn how to write a plug-in that can access resources on the World Wide Web." # 115-145 characters including spaces. This abstract displays in the search result.
ms.date: 04/10/2023
author: divkamath
ms.author: dikamath
ms.reviewer: pehecke
ms.topic: sample
search.audienceType:
  - developer
contributors:
  - JimDaly
  - phecke
---

# Sample: Web access from a plug-in

[!INCLUDE[cc-terminology](../../includes/cc-terminology.md)]

This sample shows how to write a plug-in that can access web (network) resources like a web service or feed. It also demonstrates how to limit the time duration allowed for this call. You can download the sample from [here](https://github.com/microsoft/PowerApps-Samples/tree/master/dataverse/orgsvc/C%23/WebAccessPlugin).

## How to run this sample

1. Download or clone the [Samples](https://github.com/Microsoft/PowerApps-Samples) repo so that you have a local copy. This sample is located under PowerApps-Samples-master\dataverse\orgsvc\C#\WebAccessPlugin.
1. There are two different plug-in class examples:
   - WebClientPlugin uses [WebClient Class](/dotnet/api/system.net.webclient)
   - HttpClientPlugin uses [HttpClient Class](/dotnet/api/system.net.http.httpclient)
1. Open the sample solution in Visual Studio, navigate to the project's properties, and verify the assembly will be signed during the build. Press F6 to build the sample's assembly (WebAccessPlugin.dll).
1. Run the Plug-in Registration tool and register the assembly in the Microsoft Dataverse server's sandbox and database.
1. For either plug-in type, when registering a step, specify a web URI string (i.e., `https://www.microsoft.com`) in the unsecure configuration column.
   - The default value `https://www.bing.com` will be used if none is provided.
1. Using an app or write code to perform the appropriate operation to invoke the message and table request that you registered the plug-in on.
1. When the plug-in runs, if the duration of the call exceeds the 15 second limit, it will throw an error. Otherwise it should succeed.
1. When you are done testing, unregister the assembly and step.

## What this sample does

When executed, the plug-in downloads web page data from the specified web service address (or the default address).
If the request exceeds the 15 second limit it will throw an [InvalidPluginExecutionException](/dotnet/api/microsoft.xrm.sdk.invalidpluginexecutionexception)
and write details to the Plug-in Trace Log.

- If the `WebClientPlugin` plugin fails, it will write something like the following to the Plug-in Trace log:

  ```
  Downloading the target URI: https://www.bing.com
  Exception: Microsoft.Xrm.Sdk.InvalidPluginExecutionException: The timeout elapsed while attempting to issue the request. ---> System.Net.WebException: The operation has timed out
    at System.Net.WebClient.DownloadDataInternal(Uri address, WebRequest& request)
    at System.Net.WebClient.DownloadData(Uri address)
    at PowerApps.Samples.WebClientPlugin.Execute(IServiceProvider serviceProvider)
    --- End of inner exception stack trace ---
    at PowerApps.Samples.WebClientPlugin.Execute(IServiceProvider serviceProvider)
  ```

- If the `HttpClientPlugin` plug-in fails, it will write something like the following to the Plug-in Trace log:
  ```
  Downloading the target URI: https://www.bing.com
  Inner Exceptions:
    Exception: System.Threading.Tasks.TaskCanceledException: A task was canceled.
  Exception: Microsoft.Xrm.Sdk.InvalidPluginExecutionException: An exception occurred while attempting to issue the request.
     at PowerApps.Samples.HttpClientPlugin.Execute(IServiceProvider serviceProvider)
  ```
  The [TaskCanceledException](/dotnet/api/system.threading.tasks.taskcanceledexception) is somewhat ambigous about the cause of the task being cancelled. For a more complete solution showing how to explicitly detect errors due to time outs, see this blog post: [Better timeout handling with HttpClient](https://thomaslevesque.com/2018/02/25/better-timeout-handling-with-httpclient/).

## How this sample works

In order to simulate the scenario described in [What this sample does](#what-this-sample-does), the sample will do the following:

### Setup

1. Checks the constructor's unsecure configuration parameter for a web address value; otherwise, the default value is used.
2. Depending on which plug-in is registered, either the [WebClient Class](/dotnet/api/system.net.webclient) or [HttpClient Class](/dotnet/api/system.net.http.httpclient) class is used by the plug-in's `Execute` method to download web page data.
3. If the call exceeds the 15 second duration specified, an [InvalidPluginExecutionException](/dotnet/api/microsoft.xrm.sdk.invalidpluginexecutionexception) will be thrown and details about the error will be written to the Plug-in Trace Log.

### Demonstrate

#### WebClientPlugin plugin

1. Uses a derived `CustomWebClient` class to set the [WebRequest.Timeout Property](/dotnet/api/system.net.webrequest.timeout) that is not available in the `WebClient` class.

   ```
    /// <summary>
    /// A class derived from WebClient with 15 second timeout and KeepAlive disabled
    /// </summary>
    public class CustomWebClient : WebClient
    {
      protected override WebRequest GetWebRequest(Uri address)
      {
        HttpWebRequest request = (HttpWebRequest)base.GetWebRequest(address);
        if (request != null)
        {
          request.Timeout = 15000; //15 Seconds
          request.KeepAlive = false;

        }
        return request;
      }
    }
   ```

1. Uses the [WebClient.DownloadData Method](/dotnet/api/system.net.webclient.downloaddata) to download the data from the resource.
1. Shows how to parse the expected [WebException Class](/dotnet/api/system.net.webexception) and use the [Status Property](/dotnet/api/system.net.webexception.status) to determine whether the cause of the failure was due to a timeout.

#### HttpClientPlugin plugin

1. Uses the [HttpClient Class](/dotnet/api/system.net.http.httpclient) and sets the [Timeout Property](/dotnet/api/system.net.http.httpclient.timeout) to limit the allowed time for the operation to complete.
1. Catches the expected [AggregateException Class](/dotnet/api/system.aggregateexception) and examines the inner exceptions for the expected [TaskCanceledException](/dotnet/api/system.threading.tasks.taskcanceledexception)

### See also

[Access external web resources](../../access-web-services.md)<br/>
[Register a plug-in](../../register-plug-in.md)

[!INCLUDE[footer-include](../../../../includes/footer-banner.md)]
