# ASP.NET 5.0 in Production

This chapter will cover:

 - What does an ASP.NET 5.0 app look like in production? What files are needed, what is the directory structure?
 - Does it run on Windows Server, on Linux? In EC2 or Azure websites? In Docker? 
 - Is it self-hosted with Kestrel or will it be hosted inside IIS? 
 - Where will production configuration data come from? 
 
It will drill into the `dnu publish` command as a way to prepare the app. 

At the end of this chapter, readers will have a good understanding of exactly what ASP.NET apps would ideally look like when deployed in production. 
They'll understand that the Roslyn thing is OK for local development or Azure websites, but not so suited to repeatable production deployments. If necessary, they could manually deploy an ASP.NET 5.0 app to a production IIS box. 

-------

## Platforms and Web Servers

In the ASP.NET world, "Production" for an application always meant running on a version of Windows Server, and running under IIS. As we discussed in chapter 1, ASP.NET 5.0 is about choice. And that means:

 - **A choice of operating systems**  
   Production servers might run Windows Server 2012 R2, or they might equally run a Linux distribution like Ubuntu Server. While ASP.NET 5.0 applications that target the desktop CLR will still require Windows, the CoreCLR is cross platform. 

 - **A choice of web servers**  
 IIS has been the dominant Windows web server, and is still supported for ASP.NET 5.0. Kestrel is a new cross-platform web server that is designed specifically for ASP.NET 5.0.

### Kestrel

Kestrel is a cross-platform HTTP server, and it allows ASP.NET 5.0 applications to run consistently across Windows, Linux, and OSX. 

Where web servers like IIS and Apache are designed to be general-purpose web servers, with support for many languages and features like directory browsing and static content serving, Kestrel is designed specifically for hosting ASP.NET 5.0. 

Architectually, Kestrel builds on top of:

 - **libuv**, the open source asynchronous event library used by Node.js. This provides asynchronous TCP sockets to Kestrel in an OS-agnostic manner. 
 - `SslStream`, a .NET framework class to turn an ordinary stream into a TLS stream. This allows Kestrel to support HTTPS, including client-side certificates. On Windows, `SslStream` builds on SChannel, the standard Windows TLS components also used by IIS. 
 
In addition to TCP, Kestrel can also listen on named pipes or UNIX pipes. This is something we'll bump into later. 

Kestrel can be used to host your web application, or embedded inside your own process. This is handy when building long-running services that sometimes need to present a web interface. 

Digram showing DNX and user mode. 

### IIS

IIS is a general purpose web server which can also host ASP.NET 5.0. However, the way IIS hosts ASP.NET has changed quite substantially with ASP.NET 5.0. Let's first look at the architecture of IIS and how ASP.NET was traditionally hosted, and how it now works with ASP.NET 5.0. 

#### IIS Architecture

Windows ships with a kernel-mode device driver called HTTP.sys, which listens for HTTP connections and hands them over to an appropriate application. It allows multiple applications to effectively share the same IP/port combinations by performing some HTTP parsing in the kernel before dispatching to the application. For example: 

    http://server:80/app1 -> hosted by IIS
    http://server:80/app2 -> hosted by a different process

IIS builds on top of HTTP.sys - the request is initially received by HTTP.sys (which also performs some security filtering, like Windows Authentication or client certificates), which in turn hands it to IIS. It then sends the response from IIS back to the client. 

Each application running on IIS belongs to an application pool, and a worker process is created to serve requests to that application pool. This is the w3wp process you'll sometimes see in task manager. If the process runs for significant time, or experiences severe memory leaks and runs out of memory, or hangs, IIS can terminate it and create a new worker process. 

The application pools in IIS then have two different pipeline modes - the classic pipeline mode, which relies on unmanaged ISAPI modules to do the work, and the integrated pipeline mode, where the desktop CLR is loaded into IIS and .NET modules can be used to serve requests. 

#### Changes in ASP.NET 5.0

As of ASP.NET 5.0, the integrated pipeline mode for application pools is obsolete. Instead, ASP.NET 5.0 under IIS actually uses Kestrel! 

The resulting architecture looks like this: 

Diagram showing IIS->W3WP.exe->DNX->Kestrel

Under ASP.NET 5.0:

 1. The client sends a request
 2. HTTP.sys intercepts the request, sends it to the WWW Service, which then sends it to an application pool worker process
 3. The worker process hosts a single, unmanaged module WHATEVER
 4. This module launches DNX, which hosts Kestrel
 5. The request is forwarded to Kestrel over named pipes
 6. Kestrel processes the request and returns the response back to the worker process
 7. The worker process returns the response to HTTP.sys

Just as the worker process runs for hours or days between being recycled, the DNX process hosting Kestrel is created when the worker process is created, and serves multiple requests. 

When I look at this diagram, I do wonder how relevant IIS will remain. In the Linux world, general purpose web servers like Apache seem to have been replaced by application-specific HTTP servers (Node.js processes for example) fronted by something like NGINX which simply proxies requests. IIS has effectively been reduced to this. The integrated pipeline is obsolete, and most of the modules in IIS will probably just become OWIN middleware. 

### WebListener

This just gives you all the problems of HTTP.sys with none of the performance (kernel-mode caching of static content doesn't work anyway). Just use Kestrel. (Though it does provide windows auth - I wonder if Kestrel can)


## ASP.NET 5.0 in Production



## In the olden days



