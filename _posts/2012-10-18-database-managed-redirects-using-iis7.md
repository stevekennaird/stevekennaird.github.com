---
layout: post
title: "Database managed redirects using IIS7"
description: ""
category: Internet-Information-Services
tags: [iis, sql-server]
---
{% include JB/setup %}

## Handling database-managed redirects for a website in IIS7

So you've got a website that you've got hundreds of redirects for. Maybe you're replacing an old site that had a poor url structure, 
but don't want to lose any SEO juice attributed to the old pages, and you don't want everyone who comes to your site from a search engine seeing a 404.

As with coding in general, there are many ways to get to the same end result. My first thoughts were to use either a HttpModule (or if you're only fussed about requests for files with a
file extension, e.g. .html, then a HttpHandler) or to try and hack in to IIS7's UrlRewrite feature.

There'll be other options depending on how your website is built, but this is what first came to mind for my scenario. I needed to be able to easily add new redirects and their [http redirect type](http://en.wikipedia.org/wiki/URL_redirection#HTTP_status_codes_3xx)
from an admin area, so that anybody with access to the admin area could add a redirect, they wouldn't have to get a developer to do it for them. So, say if a content editor needed
to set up a permanent redirect for '/homepage' to just '/home', they could with ease. Be cautious who you let do this though, if they don't know what they're doing, they can cause infinite redirect loops and break the website!

I chose to build my redirect functionality on top of IIS. This suits my scenario but may not suit yours, if you can't access IIS on the server for example, it can be a bit more tricky (though these rules can be defined in the web.config file of your website).
My reason for this is that I thought it would be more simple, and more likely to get right first time as it built on top of functionality that IIS was designed to offer. It also kept the visual route I could see in my head of the request executing 
my code shorter and more clean. This is my opinion obviously, but when you see the amount of lines required to get this up and running, you'll see why I went for it.

First of all, you need an admin area where you can add/edit/delete redirects, which I won't go through as that isn't the point of this post. You need to store in a SQL database table:

* Original Url [nvarchar(256)] (The url you want to redirect somewhere else) 
* Destination Url [nvarchar(256)] (Where you want the request to be redirected to)
* Redirect Type [int] (An integer representation of a HttpStatusCode, e.g. 301 for a permanent redirect, 302 for found, or 307 for a temporary redirect)

*Important: Make sure you index the original url column, and the redirect type columns.*

For each redirect type you use, you then need to set up a stored procedure in the same database, like so:

`
CREATE PROCEDURE [dbo].[GetRewrittenUrl301]  
@input nvarchar(256)  
AS  
SELECT NewUrl   
FROM dbo.Redirect   
WHERE @input LIKE OriginalUrl AND StatusCode = 301  
`
Note the use of the like keyword in the stored procedure. This allows you to enter wildcard redirects, e.g. if you had a record in the Redirect table where the OriginalUrl was 'old-folder-name/%', and urls starting with
'old-folder-name/' would be redirected; you wouldn't need to create a redirect for every single child page for that folder.

Now, you can either use IIS7 to setup your UrlRewrite providers, or you can try and set it up via the web.config. Follow [this](http://www.iis.net/learn/extensions/url-rewrite-module/using-custom-rewrite-providers-with-url-rewrite-module) for instructions of how to do it in IIS7 manually, or amend the code
below to suit your scenario (it must go withing the system.webServer node). In my case, I'm only handling 301 and 302 redirects. You'll need to amend the connection string to connect to your database.

`
<rewrite>  
      <providers>  
        <provider name="DbProvider_Permanent" type="DbProvider, Microsoft.Web.Iis.Rewrite.Providers, Version=7.1.761.0, Culture=neutral, PublicKeyToken=0545b0627da60a5f">  
          <settings>  
            <add key="ConnectionString" value="Data Source=.\SQLExpress;Initial Catalog=HF-Live;Persist Security Info=True;User ID=my-db-username;Password=my-db-pass" />  
            <add key="StoredProcedure" value="GetRewrittenUrl301" />  
            <add key="CacheMinutesInterval" value="1" />  
          </settings>  
        </provider>  
        <provider name="DbProvider_Found" type="DbProvider, Microsoft.Web.Iis.Rewrite.Providers, Version=7.1.761.0, Culture=neutral, PublicKeyToken=0545b0627da60a5f">  
          <settings>  
            <add key="ConnectionString" value="Data Source=.\SQLExpress;Initial Catalog=HF-Live;Persist Security Info=True;User ID=my-db-username;Password=my-db-pass" />  
            <add key="StoredProcedure" value="GetRewrittenUrl302" />  
            <add key="CacheMinutesInterval" value="1" />  
          </settings>  
        </provider>  
      </providers>  
</rewrite>
`

You can see that I got my inspiration from [here](http://www.iis.net/learn/extensions/url-rewrite-module/using-custom-rewrite-providers-with-url-rewrite-module), I just extended it to handle different types of redirects.

It currently works really well, and has been rock solid to date (been in production for three months without issue).