# Introduction

ASP.NET was first released in 2002. Over the last decade, it has undergone significant changes and investment by Microsoft. From Web Forms to MVC and Web API, from ASP.NET AJAX to SignalR, ASP.NET has never stood still. And yet the architecture that underpins the platform never really changed: ASP.NET was coupled to IIS, IIS was coupled to Windows. The .NET runtime is installed globally and new versions were only released once every couple of years. 

ASP.NET 5 isn't just a few new features with a major version stamp. It's a significant architectural rebuild of the platform underneath ASP.NET. ASP.NET 5 is all about choice:

 - **Choice of operating systems**  
 ASP.NET 5 applications can target .NET Core, a cross-platform .NET runtime. .NET Core is supported on Windows, OSX and Linux, all by Microsoft. 
 - **Choice of web servers**  
 ASP.NET 5 is no longer coupled to IIS. Any [OWIN-compatible](http://owin.org/) server can host ASP.NET. You can run ASP.NET under IIS, or under a new, cross-platform HTTP server called Kestrel, or embedded in your own server applications. 
 - **Choice of framework versions**  
 .NET Core is portable. If you target .NET Core, the entire runtime can be bundled and deployed side by side with your application. When Microsoft make improvements to the framework, your application can use them immediately, without affecting other applications. 

## Goal of this book

Have you ever had this experience? You decide to learn a new programming language, and you set out to build an application with it. You learn the language, write the code, and slowly your application takes shape. It runs locally in your development environment just fine. You want to deploy it somewhere for others to try, or maybe put it into production, and then you realize: you have no idea how to go about deploying it. Plenty of tutorials will teach you the frameorks and development environments, but finding good information on how to deploy and run the application in a production-like environment is really difficult. 

This book won't teach you how to *develop* ASP.NET 5 applications. There are plenty of great resources that can do that, from the [official ASP.NET 5 documentation](http://docs.asp.net) to books and blog posts and presentations. The purpose of this book is to teach you how to *deploy* them. As you learn ASP.NET 5 and prepare to deploy your applications, you'll find that many things have changed:

 - The way ASP.NET is hosted has changed
 - The way you compile and publish applications has changed
 - The way you configure applications for production has changed

The goal of this book is to walk you through these changes, and to give you everything you need to successfully deploy your first ASP.NET 5 application to production. 

I> #### Work in progress
I> This book is a work in progress. It's based on ASP.NET 5 RC1, and it's being written as we learn more about the platform. Follow the progress on the book by adding it to your cart in Leanpub:  
I>
I> https://leanpub.com/aspnetdeployment
