## What is it?
Topshelf.Integrations is a collection of packages that extend the [Topshelf Project](http://topshelf-project.com). These packages handle the boiler plate code necessary for several use cases that we have found very useful for quickly developing small self contained services on Windows.

These use cases include the following:

*	Scheduled activities hosted within a Windows Service
*	Integration of your preferred IoC Container

These packages solve these problems by integrating the following technologies with Topshelf and providing extensions to quickly integrate them.

*	[Quartz.NET](https://www.quartz-scheduler.net/) - Scheduling
*	[Ninject](http://www.ninject.org/) - IoC Container

## Getting Started

These packages are available on [Nuget](http://nuget.org/) and can be used in any combination desired. The Ninject package is directly integrated with Topshelf, and also has accompanying packages for Quartz.Net project to make it possible to initiate Quartz.NET IJob instances via the container.

### Topshelf.Ninject

To get the package: `Install-Package Topshelf.Ninject`

To use Ninject with your Topshelf service, all you need is three lines:

	using Ninject.Modules;
	using Topshelf;
	using Topshelf.Ninject;

	...

    class Program
    {
        static void Main()
        {
            HostFactory.Run(c =>
            {
                c.UseNinject(new SampleModule()); //Initiates Ninject and consumes Modules

                c.Service<SampleService>(s =>
                {
                    //Specifies that Topshelf should delegate to Ninject for construction
                    s.ConstructUsingNinject(); 
                    s.WhenStarted((service, control) => service.Start());
                    s.WhenStopped((service, control) => service.Stop());
                });
            });
        }
    }

### Topshelf.Quartz

To get the package: `Install-Package Topshelf.Quartz`

You may schedule any number of Quartz jobs along with your service like this:

	using System;
	using Quartz;
	using Topshelf;
	using Topshelf.Quartz;

	...

	class Program
    {
        static void Main()
        {
            HostFactory.Run(c =>
            {
                c.Service<SampleService>(s =>
                {
	            s.ConstructUsing(() => new SampleService());
		    
                    s.WhenStarted((service, control) => service.Start());
                    s.WhenStopped((service, control) => service.Stop());

                    // Schedule a job to run in the background every 5 seconds.
                    // The full Quartz Builder framework is available here.
                    s.ScheduleQuartzJob(q =>
                        q.WithJob(() =>
                            JobBuilder.Create<SampleJob>().Build())
                        .AddTrigger(() =>
                            TriggerBuilder.Create()
	                            .WithSimpleSchedule(builder => builder
		                            .WithIntervalInSeconds(5)
		                            .RepeatForever())
	                            .Build())
                        );
                });
            });
        }
    }

### Topshelf.Quartz.Ninject

To get the package: `Install-Package Topshelf.Quartz.Ninject`

To get the package: `Install-Package Topshelf.WebApi`

You may schedule any number of Quartz jobs along with your service like this:

	using System;
	using Ninject.Modules;
	using Quartz;
	using Topshelf;
	using Topshelf.Ninject;
	using Topshelf.Quartz;
	using Topshelf.Quartz.Ninject;

	...

	class Program
    {
        static void Main()
        {
            HostFactory.Run(c =>
            {
            	// Topshelf.Ninject :
		// Initiates Ninject and consumes Modules
                c.UseNinject(new SampleModule());

                c.Service<SampleService>(s =>
                {
                    // Topshelf.Ninject :
		    // Construct service using Ninject
                    s.ConstructUsingNinject();

                    s.WhenStarted((service, control) => service.Start());
                    s.WhenStopped((service, control) => service.Stop());

                    // Topshelf.Quartz.Ninject :
		    // Construct IJob instance with Ninject
                    s.UseNinjectQuartzJobFactory();

                    // Topshelf.Quartz :
                    // Schedule a job to run in the background every 5 seconds.
                    // The full Quartz Builder framework is available here.
                    s.ScheduleQuartzJob(q =>
                        q.WithJob(() =>
                            JobBuilder.Create<SampleJob>().Build())
                        .AddTrigger(() =>
                            TriggerBuilder.Create()
	                            .WithSimpleSchedule(builder => builder
		                            .WithIntervalInSeconds(5)
		                            .RepeatForever())
	                            .Build())
                        );
                });
            });
        }
    }
