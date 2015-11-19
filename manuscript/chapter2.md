# Execution, Web Servers and Platforms

This chapter will cover:

 - What does an ASP.NET 5.0 app look like in production? What files are needed, what is the directory structure?
 - Does it run on Windows Server, on Linux? In EC2 or Azure websites? In Docker? 
 - Is it self-hosted with Kestrel or will it be hosted inside IIS? 
 - Where will production configuration data come from? 
 
It will drill into the `dnu publish` command as a way to prepare the app. 

At the end of this chapter, readers will have a good understanding of exactly what ASP.NET apps would ideally look like when deployed in production. 
They'll understand that the Roslyn thing is OK for local development or Azure websites, but not so suited to repeatable production deployments. If necessary, they could manually deploy an ASP.NET 5.0 app to a production IIS box. 

-------

In the ASP.NET world, "Production" for an application always meant running on a version of Windows Server, and running under IIS. As we discussed in chapter 1, ASP.NET 5.0 is about choice. And that means:

 - **A choice of operating systems**  
   Production servers might run Windows Server 2012 R2, or they might equally run a Linux distribution like Ubuntu Server. While ASP.NET 5.0 applications that target the desktop CLR will still require Windows, the CoreCLR is cross platform. 

 - **A choice of web servers**  
 IIS has been the dominant Windows web server, and is still supported for ASP.NET 5.0. But since ASP.NET 5.0 builds on top of OWIN, any HTTP server that can host OWIN can host ASP.NET 5.0. 

In this chapter, we'll first look at how the process model for DNX applications has changed. We'll then focus on the two HTTP servers that we expect will comprise of the majority of production ASP.NET 5.0 deployments: the new kid on the block, Kestrel, and the old, reliable IIS. 

## DNX Processes

Before we get into HTTP servers and ASP.NET 5.0, it's worth taking a detour to look at how the process model for .NET applications in general have changed with DNX. 

Take this simple C# console application:

```
using System;
using System.Threading;

namespace MyApp
{
    class Program
    {
        static void Main(string[] args)
        {
            Console.WriteLine("Hello, world!");
            Thread.Sleep(10000);
        }
    }
}
```

Before DNX, this code would get compiled into an `.exe` file. The top of the `.exe` file would contain a small bit of native code that would check the .NET runtime is installed, then load the runtime, and then the runtime would run the rest of the application. In task manager, you would see your executable as the top process:

IMAGE

Under DNX, this changes. Firstly, in development, there is no compilation step - code is compiled on the fly with Roslyn, so you'll never see a `.dll` or `.exe` in the first place. When you are ready to ship the application, though, you'll eventually compile it. You do this using `dnu`:

```
dnu publish --no-source
```

Instead of creating a `.exe` as you might expect, the code is actually compiled into a `.dll` inside a directory structure. The directories contain the application bundled as a NuGet package, plus various JSON configuration files. At the root is a `.cmd` batch file which invokes the application:

IMAGE

The batch file invokes DNX.exe (the actual batch file is longer than this - snipped for brevity):

```
IF "%DNX_PATH%" == "" (
  SET "DNX_PATH=dnx.exe"
)
@"%DNX_PATH%" --project "%~dp0packages\MyApp-DNX\1.0.0\root" --configuration Debug MyApp %*
```

The MyApp file without the extension is a shell script, so the same application can run on Linux:

```
exec "dnx" --project "$DIR/packages/MyApp-DNX/1.0.0/root" --configuration Debug MyApp "$@"
```

The trick that DNX uses is similar to how Java applications run. Java applications aren't compiled into an `.exe`, they just ship as a `.jar` file which is then loaded by the Java executable:

```
java.exe myapp.jar
```

Something you'll notice is that if you look in task manager, you won't see the name of your console app anywhere - just DNX. In fact I'm not even sure which one of these three DNX processes contains my application: 

IMAGE

A good way to figure that out is by using SysInternals Process Explorer: 

IMAGE

One of the great benefits of DNX is that if your application targets CoreCLR, the runtime can be distributed with your application. You can now see how this can work - since the CoreCLR includes DNX.exe, all you need to do is distribute the CoreCLR runtime files with your application, and your batch file will invoke that DNX version. You can bundle the runtime and have your batch file call that simply by specifying the option when publishing:

```
dnu publish --runtime active --no-source
```

A> ## Room for improvement? 
A> Personally, I think this folder structure is a bit messy and makes this seem more complicated than it should be. There's a good opportunity to hide all of these etails by keeping the shell script and batch file, but compressing the rest of the files into a NuGet package. `dnx myapp.1.0.0.nupkg` would be much neater. 

Now that you are familiar with how DNX invokes processes, let's look at two HTTP server options for ASP.NET 5.0. 

## Kestrel

Kestrel is a cross-platform, open source HTTP server for ASP.NET 5.0. It's built by the same team at Microsoft that built ASP.NET 5.0, and it allows ASP.NET 5.0 applications to run consistently across Windows, Linux, and OSX. 
Where web servers like IIS and Apache are designed to be general-purpose web servers, with support for many languages and features like directory browsing and static content serving, Kestrel is designed specifically for hosting ASP.NET 5.0. Architectually, Kestrel builds on top of:

 - **libuv**, the open source asynchronous event library used by Node.js. This provides asynchronous TCP sockets to Kestrel in an OS-agnostic manner. 
 - `SslStream`, a .NET framework class to turn an ordinary stream into a TLS stream. This allows Kestrel to support HTTPS, including client-side certificates. On Windows, `SslStream` builds on SChannel, the standard Windows TLS components also used by IIS. 

Kestrel ships as a class library. 

In addition to TCP, Kestrel can also listen on named pipes or UNIX pipes. At first this may seem strange, but we'll explore why this exists a little later. 

Kestrel can be used to host your ASP.NET 5.0 application, or embedded inside your own process. This is handy when building long-running services that sometimes need to present a web interface. 



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


HTTPPlatformHandler
