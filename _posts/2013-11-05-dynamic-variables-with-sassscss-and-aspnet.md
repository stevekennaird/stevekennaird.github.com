---
layout: post
title: "Dynamic Variables with SASS SCSS and ASP.Net"
description: ""
category:  "ASP.Net"
tags: [sass, scss, css, design]
---
{% include JB/setup %}

At work we're in the process of building a new SAAS product which needs to be
themed/skinned per customer (where the customer is a company, each with many
employees).

The designer we're working with sold me on the merits of using Compass with
SCSS. The variables used in SASS are perfect for writing minimal maintainable
css with the ability to apply colour changes in many places without a manual
find'n'replace process through multiple css files. The challenge for me and my
team is to popuplate the SASS variables in ASP.Net from the server side. This
wasn't as easy as I expected.

I checked out [Nuget][1] and [StackOverflow][2] in the hope of finding some assistance.
After looking into a few options that come up through Nuget, I couldn't find
something that made this easy. And on StackOverflow, there wasn't really any
help. So it was time to get my hands dirty!

One Nuget package that I did like was [SassAndCoffee][3]. This package took care
of all the Ruby-related stuff for me. It transforms .sass/.scss files as they're
requested, compiles them to plain ol' css (and caches them). All very cool, but
it relies on the .sass/.scss files being on disk. No support for dynamically
generated files (for if you want to pull values out of the database/session to
use as SASS variable values). I downloaded the source from the repo, and got
something to work, but not in a way that I was happy with. I considered
re-writing and contributing to the project on [GitHub][4], but the repository
wasn't very active, and had open pull requests that hadn't been looked into for
over a year.

I was trying to use the @import command to import a client-specific .scss file
into the main .scss file for the website. This failed, as the ruby tool which
compiles .scss to css required any imported files to be saved on the disk.

But, this package was still useful. I just had to think from a completely
different angle.

The solution ended up being pretty simple.

1.  Based on a value in the user's session, write the path to the css file into
    the view, with the path being something like /content/scss/client-123.css
    (where 123 is the id of the company from the database)

2.  Before the page is rendered, look for a .scss file with the same
    filepath/name (just a different file extension). If one doesn't exist.
    Create it. This is where the magic happens. You can do this part how you
    want. Essentially I just created a string listing a few SASS variables,
    pulling in values from the database. I then opened the main .scss file on
    disk, prepended by variables to it, then wrote that string to a new .scss
    file on disk, with the correct filename (client-123.scss)

3.  That's it. Let the request continue, the .scss file now exists on disk, and
    SassAndCoffee will be able to read it and convert it to css, ready for
    rendering by the browser.

4.  The final part is the clear up. These variable values can be changed in the
    database, so when they change, we need to remove the .scss and .css files
    (client-123.scss and client-123.css) so they can be rebuilt.



I knocked together a quick proof of concept. Not the most tidy (or best placed),
but it works and might give you some clues as to how you can achieve something
similar.


using System;
using System.IO;
using System.Linq;
using System.Web.Mvc;

namespace Attempt2.Controllers
{
    public class HomeController : Controller
    {
        private const string BaseScssDirectory = "~/content/scss/";
        private const string OutputDirectory = "clients/";
        private const string MainScssFilename = "main.scss";
        private const string OutputFilenameFormat = "client-{0}.scss";
        
        public ActionResult Index(int? clientId)
        {
            int useClientId = clientId.GetValueOrDefault(1);
            Session["ClientId"] = useClientId;

            if (!ClientScssExists(useClientId))
            {
                SaveClientScssFile(useClientId);
            }


            return View();
        }

        bool ClientScssExists(int clientId)
        {
            string filename = string.Format(OutputFilenameFormat, clientId);
            string path = Server.MapPath(String.Format("{0}{1}{2}", BaseScssDirectory, OutputDirectory, filename));
            return System.IO.File.Exists(path);
        }

        private string GenerateClientScss(int clientId)
        {
            string mainCss = GetMainScss();
            string themeCss = GenerateThemeScss(clientId);

            return string.Format("{0}{1}{2}", themeCss, string.Concat(Enumerable.Repeat(Environment.NewLine, 5)), mainCss);
        }

        private string GenerateThemeScss(int clientId)
        {
            switch (clientId)
            {
                case 1:
                    return @"$PrimaryColour: red;";
                case 2:
                    return @"$PrimaryColour: green;";
                case 3:
                    return @"$PrimaryColour: blue;";
                default:
                    return @"$PrimaryColour: black;";
            }
        }

        private string GetMainScss()
        {
            string filePath = string.Format("{0}{1}", BaseScssDirectory, MainScssFilename);
            return System.IO.File.ReadAllText(Server.MapPath(filePath));
        }

        private void SaveClientScssFile(int clientId)
        {
            string filename = string.Format(OutputFilenameFormat, clientId);
            string filePath = string.Format("{0}{1}{2}", BaseScssDirectory, OutputDirectory, filename);
            filePath = Server.MapPath(filePath);

            string clientScss = GenerateClientScss(clientId);

            using (var writer = new StreamWriter(filePath, false))
            {
                writer.WriteLine(clientScss);
            }

        }


    }
}


[1]: <http://www.nuget.org/>

[2]: <http://www.stackoverflow.com/>

[3]: <https://github.com/paulcbetts/SassAndCoffee>

[4]: <http://www.github.com/>