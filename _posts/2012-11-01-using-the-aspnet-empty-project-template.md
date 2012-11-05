---
layout: post
title: "Using the ASP.Net Empty Project Template"
description: ""
category: ASP.Net
tags: [asp.net, visual-studio, asp.net-mvc]
---
{% include JB/setup %}

#Getting yourself out of a jam once you've built on top of the ASP.Net Empty Project Template

With the launch of ASP.Net v4.5 and MVC4/WebAPI etc, the ASP.Net team took the admirable step to making a "single ASP.Net". The theory being that
the different project types available to users in Visual Studio rail-roaded the user to making big choices about the application they were building
before they'd even really started. Whilst the different template types were great in terms of saving the user some initial configuration work to do
when creating a project (e.g. referncing the relevant DLLs, setting up handlers in the web.config file), it became difficult to mix project types.

By creating a single ASP.Net, Visual Studio could offer users the ability to have a "plain" ASP.Net website project, but they could then add (amongst other things) MVC4 capabilities
by installing the [MVC Nuget package](http://nuget.org/packages/Microsoft.AspNet.Mvc). When tasked with a new website build, I thought I could put this to the test,
as initially the website only needed to handle requests for PDF files, which I knew could be served using an HttpHandler. I knew there was no need for the extra bloat
and code involved in using an MVC controller, so decided to opt for the ASP.Net Empty Project template, knowing that I should be able to get MVC added later into the same 
project if/when needed. So far so good, the website was launched, PDFs were served to users, everybody was happy.

## But the requirements changed... now I need MVC!

Developers should never be surprised when the requirements change, even the best planned projects can have shifting requirements. In my case, the project I had built
that was intended to only serve PDFs from a download link on other websites now needed the ability to browse the complete catalog of available PDFs in a category structure.
I now wanted MVC in the project, as that's my tool of choice for quickly developing dynamic pages.

So, step one, download and install the ASP.Net MVC package via the Package Manager console (or via the Nuget packages UI).

<pre>
PM&gt; Install-Package Microsoft.AspNet.Mvc
</pre>

Next, I was expecting the project to take on the role of an MVC project; I'd be able to easily add Controllers via the context menus on the 'Controllers' folder in the Solution Explorer,
I'd be able to easily add missing Views for ActionResult methods where Visual Studio had detected the view was missing. But no, that wasn't the case at all!

It turns out, it doesn't magically work like that. Once you've installed the MVC Nuget package, there are a few more steps you need to do to get your controllers firing for requested urls.

### Step One: Set up your routing

So you've added your controller with one or more actions. Now, as you would with a standard MVC project, you need to map your controller actions to urls using [Routing](http://www.asp.net/mvc/tutorials/older-versions/controllers-and-routing/asp-net-mvc-routing-overview-cs).
To do this, you'll need to ensure your routing is being managed either within the Global.asax file or handled via [WebActivator](https://github.com/davidebbo/WebActivator).

For example, in the Global.asax file:

<pre>
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;
using System.Web.Routing;

namespace MyPlainAspNetApplication
{
    public class MvcApplication : System.Web.HttpApplication
    {
        public static void RegisterRoutes(RouteCollection routes)
        {
            routes.MapRoute(
                "BrowseControllerRoute",						// Route name
                "browse/{action}",								// URL with parameters
                new { controller = "Browse", action = "Index" } // Parameter defaults
            );

        }

        protected void Application_Start()
        {
            RegisterRoutes(RouteTable.Routes);
        }
    }
}

</pre>

### Step Two: Setup the standard handlers in the root-level web.config file

Within the system.webServer node in the web.config, the following needs to be added to ensure that ASP.Net intercepts requests for extensionless urls.
Without this, request urls that you've just set up routing for will return a 404 error.

<pre>
&lt;modules runAllManagedModulesForAllRequests="true" /&gt;
&lt;handlers&gt;
	&lt;remove name="ExtensionlessUrlHandler-ISAPI-4.0_32bit" /&gt;
    &lt;remove name="ExtensionlessUrlHandler-ISAPI-4.0_64bit" /&gt;
    &lt;remove name="ExtensionlessUrlHandler-Integrated-4.0" /&gt;
    &lt;add name="ExtensionlessUrlHandler-ISAPI-4.0_32bit" path="*." verb="GET,HEAD,POST,DEBUG,PUT,DELETE,PATCH,OPTIONS" modules="IsapiModule" scriptProcessor="%windir%\Microsoft.NET\Framework\v4.0.30319\aspnet_isapi.dll" preCondition="classicMode,runtimeVersionv4.0,bitness32" responseBufferLimit="0" /&gt;
    &lt;add name="ExtensionlessUrlHandler-ISAPI-4.0_64bit" path="*." verb="GET,HEAD,POST,DEBUG,PUT,DELETE,PATCH,OPTIONS" modules="IsapiModule" scriptProcessor="%windir%\Microsoft.NET\Framework64\v4.0.30319\aspnet_isapi.dll" preCondition="classicMode,runtimeVersionv4.0,bitness64" responseBufferLimit="0" /&gt;
    &lt;add name="ExtensionlessUrlHandler-Integrated-4.0" path="*." verb="GET,HEAD,POST,DEBUG,PUT,DELETE,PATCH,OPTIONS" type="System.Web.Handlers.TransferRequestHandler" preCondition="integratedMode,runtimeVersionv4.0" /&gt;
&lt;/handlers&gt;
</pre>

### Step Three: Other necessary web.config changes

There are two required appSettings values that need to be added for MVC to work properly. Ensure these are added to the web.config.

<pre>
&lt;appSettings&gt;
	&lt;add key="webpages:Version" value="2.0.0.0" /&gt;
	&lt;add key="webpages:Enabled" value="false" /&gt;
&lt;/appSettings&gt;
</pre>

Next, the standard MVC pages node needs to be added to the web.config system.web node like so:

<pre>
&lt;pages&gt;
	&lt;namespaces&gt;
		&lt;add namespace="System.Web.Helpers" /&gt;
		&lt;add namespace="System.Web.Mvc" /&gt;
		&lt;add namespace="System.Web.Mvc.Ajax" /&gt;
		&lt;add namespace="System.Web.Mvc.Html" /&gt;
		&lt;add namespace="System.Web.Routing" /&gt;
		&lt;add namespace="System.Web.WebPages" /&gt;
	&lt;/namespaces&gt;
&lt;/pages&gt;
</pre>

### Step Four: Hacking the project file to behave like an MVC project

Steps 1 to 3 could be handled as the MVC package is installed from Nuget - perhaps they will be in the future with some web.config transforms and the insertion of some routing code.
Next is the main part that I feel the Visual Studio/MVC teams could do better with. When you create a new MVC project (as opposed to an empty ASP.Net project), the whole Visual Studio IDE gears itself towards MVC to 
make common tasks quick and painless, such as adding a new Controller, where the Controller class gets added, set to inherit from the default Controller class, and adds an empty ActionResult
ready for the user to modify. But, when you add MVC via Nuget to a project, you don't get this, and it isn't easy to do. You have to hack the project file to tell Visual Studio that this
is now an MVC project. To do this, you need to change the value for ProjectTypeGuids within the csproj/vbproj file for your application.

In my case, I had to close down Visual Studio, *backup* the csproj I was about to change, then search inside the csproj file using a text editor for the ProjectTypeGuids node, and make the following change:

From:
<pre>
&lt;ProjectTypeGuids&gt;
	{349c5851-65df-11da-9384-00065b846f21};{fae04ec0-301f-11d3-bf4b-00c04f79efbc}
&lt;/ProjectTypeGuids&gt;
</pre>

To:
<pre>
&lt;ProjectTypeGuids&gt;
	{E3E379DF-F4C6-4180-9B81-6769533ABE47};{349c5851-65df-11da-9384-00065b846f21};{fae04ec0-301f-11d3-bf4b-00c04f79efbc}
&lt;/ProjectTypeGuids&gt;
</pre>

You can see that the node originally contained two GUIDs. The first one is the project type, and the second is the language used. So in my case, the first is Web Application and C#, and the second
is MVC 4, Web Application and C#.

Also, in the same PropertyGroup node as the ProjectTypeGuids is a child of, add the following node:
<pre>
	    &lt;MvcBuildViews&gt;false&lt;/MvcBuildViews&gt;
</pre>

Once that has been done, you can open your solution file again, and you should notice that your project is an MVC project. You get the "Add Controller..." context menu option once you click on
your "Controllers" folder in the Solution Explorer, adding View files becomes easy again.

Problem solved.


#### Useful information

I worked out how to do this by comparing my empty ASP.Net project with a new MVC project. I also gained help relating to ProjectTypeGuids from the following two pages:

* [ASP.Net MVC 4 release notes](http://www.asp.net/whitepapers/mvc4-release-notes#_Toc303253806) - specifically the part about upgrading MVC3 to MVC4
* [StackOverflow question about ProjectTypeGuids](http://stackoverflow.com/questions/2911565/what-is-the-significance-of-projecttypeguids-tag-in-the-visual-studio-project-fi)