# ASP.NET 5.0 in Production

This chapter will cover:

 - What does an ASP.NET 5.0 app look like in production? What files are needed, what is the directory structure?
 - Does it run on Windows Server, on Linux? In EC2 or Azure websites? In Docker? 
 - Is it self-hosted with Kestrel or will it be hosted inside IIS? 
 - Where will production configuration data come from? 
 
It will drill into the `dnu publish` command as a way to prepare the app. 

At the end of this chapter, readers will have a good understanding of exactly what ASP.NET apps would ideally look like when deployed in production. 
They'll understand that the Roslyn thing is OK for local development or Azure websites, but not so suited to repeatable production deployments. If necessary, they could manually deploy an ASP.NET 5.0 app to IIS. 
