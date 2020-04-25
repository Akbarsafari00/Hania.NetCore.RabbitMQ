# Hania.NetCore.RabbitMQ

[![Build status](https://ci.appveyor.com/api/projects/status/q261l3sbokafmx1o/branch/master?svg=true)](https://www.nuget.org/packages/Hania.AutoIncluder/)
[![NuGet](http://img.shields.io/nuget/v/Hania.NetCore.RabbitMQ.svg)](https://www.nuget.org/packages/Hania.NetCore.RabbitMQ/)
[![Author](https://img.shields.io/badge/Author-Akbar%20Ahmadi%20Saray-brightgreen.svg)](https://www.nuget.org/packages/Hania.NetCore.RabbitMQ/)
[![Linkdin](https://img.shields.io/badge/Linkdin-Akbar%20Ahmadi%20Saray-orange.svg)](https://www.linkedin.com/in/akbar-ahmadi-saray-5a5b9016b/)


### What is Hania.NetCore.RabbitMQ?

Hania.NetCore.RabbitMQ built To improve and make better use of RabbitMQ Message-Broker in .Net Core 3

### Where can I get it?

First, [install NuGet](http://docs.nuget.org/docs/start-here/installing-nuget). Then, install [Hania.NetCore.RabbitMQ](https://www.nuget.org/packages/Hania.NetCore.RabbitMQ/) from the package manager console:

```
PM> Install-Package Hania.NetCore.RabbitMQ
```


## How do I get started?
RabbitMQ consists of two parts : Producers and Consumers
### Producer
RabbitMQ Producer writes AMQP messages to a single RabbitMQ queue.

When you configure the destination, you specify the information needed to connect to RabbitMQ client. You also define a queue and the bindings to use. You can use multiple bindings to write messages to one or more exchanges. You can also configure SSL/TLS properties, including default transport protocols and cipher suites. You can optionally configure AMQP message properties.

### Consumer
RabbitMQ is a messaging broker. It accepts messages from publishers, routes them and, if there were queues to route to, stores them for consumption or immediately delivers to consumers, if any.

Consumers consume from queues. In order to consume messages there has to be a queue. When a new consumer is added, assuming there are already messages ready in the queue, deliveries will start immediately.

### How To Create Producer?
Producers in RabbitMq sends event through the  client
#### so , let's create client :
```csharp
using Hania.NetCore.RabbitMQ.Attributes;
using Hania.NetCore.RabbitMQ.Class;
using RabbitMQ.Client;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;

namespace RabbitMQ.NetCore.Client.Producers
{
    public class TestProducer : RMQClient
    {
        public override void Builder(RMQClientBuilder builder)
        {
            // In the client we set the connection on which the rabbitmq server is active.
            // We also write the name of the page to which we are going to send the request
            builder.SetQueue("rmqtest");
            builder.SetConnection(new ConnectionFactory { HostName = "localhost" });
            base.Builder(builder);

        }
    }
}
```
In the client we set the connection on which the rabbitmq server is active.
We also write the name of the queue to which we are going to send the request.

#### Initialize Client in Startup Class 
To use this library, you must Initialize it in the Startup.cs.
###### Note: You only Add one Type of client  in [ AddRMQClient ]  method so that the library found the all of the clients from the project
example:
```csharp
using System.Reflection;
using Hania.NetCore.RabbitMQ.Extensions;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using RabbitMQ.Client;
using RabbitMQ.NetCore.Client.Consumers;
using RabbitMQ.NetCore.Client.Producers;

namespace RabbitMQ.NetCore.Client
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public void ConfigureServices(IServiceCollection services)
        {
            // Add This Code to your ConfigureServices method And Library Found All Client With This Type
            services.AddRMQClient(typeof(TestProducer));
            services.AddControllersWithViews();

        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
            }
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Home}/{action=Index}/{id?}");
            });
        }
    }
}
```




#### Now You Can use RabbitMQ Client In Controller
you can inject client in your controller or service and use it
```csharp
using System;
using System.Collections.Generic;
using System.Diagnostics;
using System.Linq;
using System.Threading.Tasks;
using Hania.NetCore.RabbitMQ.Abstract;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Extensions.Logging;
using RabbitMQ.NetCore.Client.Models;
using RabbitMQ.NetCore.Client.Producers;

namespace RabbitMQ.NetCore.Client.Controllers
{
    public class HomeController : Controller
    {
        private readonly ILogger<HomeController> _logger;
        private readonly TestProducer _testProducer;

        public HomeController(ILogger<HomeController> logger, TestProducer testProducer)
        {
            _logger = logger;
            _testProducer = testProducer;
        }

        public IActionResult Index()
        {
            return View();
        }

        public IActionResult Privacy()
        {
            // you can use client and send event with route-key
            _testProducer.SendEvent<TestModel>("send-model", new TestModel {Id=1,Name="Akbar Ahmadi Saray"});
            return View();
        }

        [ResponseCache(Duration = 0, Location = ResponseCacheLocation.None, NoStore = true)]
        public IActionResult Error()
        {
           return View();
        }
    }
}

``` 
___
### How To Create Consumer ?
Use the RMQConsumer Attribute to know which route-key to listen.
And write you logic in handle method.
```csharp
using Hania.NetCore.RabbitMQ.Abstract;
using Hania.NetCore.RabbitMQ.Attributes;
using RabbitMQ.NetCore.Client.Models;
using System;
using System.Threading.Tasks;

namespace RabbitMQ.NetCore.Client.Consumers
{
    [RMQConsumer("send-model")]
    public class TestConsumer : IRMQConsumer<TestModel>
    {
        public async Task Handle(TestModel model)
        {
            Console.WriteLine(" [x] TestConsumer Received :'{0}'", model.Name);
        }
    }
}

```

#### Initialize Consumer in Startup Class 
To use this library, you must Initialize Consumer in the Startup.cs.
###### Note: You only Add one Type of consumer  in [ AddRMQConsumer ]  method so that the library found the all of the clients from the project
example:
```csharp
using System.Reflection;
using Hania.NetCore.RabbitMQ.Extensions;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using RabbitMQ.Client;
using RabbitMQ.NetCore.Client.Consumers;
using RabbitMQ.NetCore.Client.Producers;

namespace RabbitMQ.NetCore.Client
{
    public class Startup
    {
        public Startup(IConfiguration configuration)
        {
            Configuration = configuration;
        }

        public IConfiguration Configuration { get; }

        public void ConfigureServices(IServiceCollection services)
        {
            // Add This Code to your ConfigureServices method And Library Found All Consumer With This Type
            // rmqtest : name of queue to consumer should be listen
            services.AddRMQConsumer("rmqtest",new ConnectionFactory() { HostName = "localhost" }, typeof(TestConsumer)) ;
            services.AddControllersWithViews();

        }

        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }
            else
            {
                app.UseExceptionHandler("/Home/Error");
            }
            app.UseStaticFiles();

            app.UseRouting();

            app.UseAuthorization();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapControllerRoute(
                    name: "default",
                    pattern: "{controller=Home}/{action=Index}/{id?}");
            });
        }
    }
}
```


You Kan Use this library in your netcore project and
Enjoy It :))))

Follow me on Social Media : 

Linkdin : https://www.linkedin.com/in/akbar-ahmadi-saray-5a5b9016b/
Instagram : https://www.instagram.com/amd.akbar


