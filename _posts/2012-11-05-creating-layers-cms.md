---
layout: post
title: "Creating Layers CMS"
description: ""
category: Layers-CMS 
tags: [layers-cms, asp.net, open-source]
---
{% include JB/setup %}

# Building an(other) ASP.Net C# CMS

A little while ago, I was asked to build a website for a family member who only needed a basic Content Management System for the time being, but had ambitious
plans for 6 months - 1 year down the line. Not having a lot of time on my hands, I wanted to find an open source ASP.Net C# CMS that was simple, succint, well built,
employed the latest tech (MVC4, ASP.Net 4.5, Razor), and wasn't overkill or so complicated that building on top of it would be difficult and involve a steep learning curve.

## Wait, are you sure we really need *another* CMS?

Well, I looked into a range of ASP.Net open source CMSs and found a lot of them were overkill or used some slightly older tech ([Umbraco](http://umbraco.com/), [DotNetNuke](http://www.dotnetnuke.com/), [n2cms](http://n2cms.com/)),
and some were well built but were more complicated than I needed ([Orchard CMS](http://www.orchardproject.net/)) so extending them would involve a fair deal of learning and investigating the
source code to understand what was going on. There would also be a lot of "bloat" for my intended use (more features and therefore code involved than was necessary).

I looked at [Meek CMS](http://www.adventuretechgroup.com/labs-meek/) and enjoyed the simplicity and the fact it used an open source database (MongoDB), but the project
wasn't quite mature enough and the UI wasn't as good as I wanted it to be.

I really like .NET, so I wasn't going to look at any CMSs built in other languages such as PHP.

## I give to thee, Layers CMS

So although I lacked the time to build a CMS by myself, I felt that I'd found a little niche worth exploiting for an open source project.
Surely I'm not the only one who wants a simple CMS that would be easy to extend? I couldn't find any CMS out there worth using or extending that would enable
anybody with decent ASP.Net MVC (C#) skills to pick up and build on top of, without too much of a learning curve or having to spend large amounts of time
understanding key concepts behind the system and the way it has been built.

One of the main goals has been simplicity, and a hope that my passion for .NET and open source projects would encourage development of something which I feel is lacking in
the .NET open source community; a decent, capable yet simple CMS that wouldn't put up barriers to developers looking to build on top of a CMS with some bespoke functionality.

The name comes from a concept that I struggle to get out of my head sometimes when looking at code. Coding is all about creating layers, like bricks when building a house. Use too many layers
and things get more tricky. Don't add unnecessary layers of abstractions - use extra layers only when you need to and when it feels right, and this should help you create better
written code.

### Made with great open source software

There's so much great open source software out there, you'd be a fool to not use it and participate in the communities. Amongst others, Layers CMS uses Asp.NET MVC4, MvcMailer, Twitter Bootstrap (minimal use for form styling and buttons) and Attribute Routing.

[MvcMailer](https://github.com/smsohan/MvcMailer) is used to power email notifications from the contact forms.

[Twitter Bootstrap](http://twitter.github.com/bootstrap/) is used for some basic styling, mainly for the contact form.

[Attribute Routing](https://github.com/mccalltd/AttributeRouting/wiki) is used to make it easier to define routes within your controller, no need to faff about with the global.asax to define your routes (unless you really want to!).

## Getting involved

If you want an all singing, all dancing CMS, check out some of the links above. If you too have seen the state of open source .NET CMSs and have seen the need for a more simple solution,
feel free to take a look at Layers CMS. It's on [Github](https://github.com/stevekennaird/LayersCMS), and I would welcome any feedback/criticism/opinions/pull requests!

[https://github.com/stevekennaird/LayersCMS](https://github.com/stevekennaird/LayersCMS)

[<img src="/assets/img/layers-cms/layers-cms-logo.jpg" alt="Layers CMS Logo" />](https://github.com/stevekennaird/LayersCMS)