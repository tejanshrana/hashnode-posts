## Introduction to Serverless Applications in < 10 minutes☁

I've had the opportunity to design and build some enterprise serverless applications and I wanted to share my experience with all of you. So, I would be doing a series on Serverless Applications where we first start with understanding what they are and why are they gaining popularity, and in the next post, we would be building one from scratch! Keep an eye and it's going to be fun 😀

Serverless applications are the talk of the town these days. Let's understand what they are and why is it the new cool thing! 

Let's take the definition from  [Wikipedia ](https://en.wikipedia.org/wiki/Serverless_computing) - the ultimate source of truth!


> “Serverless computing is a cloud-computing execution model in which the cloud provider runs the server, and dynamically manages the allocation of machine resources. Pricing is based on the actual amount of resources consumed by an application, rather than on pre-purchased units of capacity. It can be a form of utility computing.”


Sounds great but let's break it down to really understand what it means - 

**1. cloud-computing execution model**: 

Basically, only supported if you are hosting in the cloud. Not in an on-premise model. If you are not sure what an on-premise model is, keep reading, else, skip to the next point. 

A *very* long time ago (10 years in real life and maybe 25 in dev years), when cloud computing was not as mainstream as it is today, the process of hosting an application was to first build an application, then estimate the memory usage, disk space, throughput, HA requirements, DR requirements (it's a long list, you get the idea) and then buy a server and get a data center partner to host it for you. Some companies still do this since they invested in their data centers 10 years ago and want to make the most of it. This model of hosting applications is called "on-premise".


**2. dynamically manages the allocation of machine resources**: 

With Serverless applications, you are not responsible for allocating the memory your application needs. The cloud provider ( [Amazon Web Services](https://aws.amazon.com/),  [Google Cloud Platform](https://cloud.google.com/),  [Microsoft Azure](https://azure.microsoft.com/en-us/), etc.) will do it for you. You are only responsible for the application itself, not the memory allocation, scaling, availability, or any of that. That's the beauty of it!

What this also means is that when your application is not being used, there's no memory allocated to it meaning your application is turned off. However, as soon as somebody tries to use this application, the cloud provider allocates the necessary memory to bring it up and serve the request. This ensures that the requests are served but the application is not running when no one is using it. What is the benefit? You will see that in the next point.

**3. Pricing is based on the actual amount of resources consumed**: 

Since you are not responsible for allocating the resources to your application, you should only be paying for the resources your cloud provider had to allocate to cater to your traffic. That means if your application has been sitting idle with zero traffic, you are not billed for it. Technically, your application is not using any memory when it is not being used by anyone. This is not the case with applications deployed on your own servers since you still need to keep the application up and running to cater to the requests when they come in.

There's a "gotcha" here though. As you can imagine, if your application is "started" when someone starts using it, it would be slower in responding to the request as compared to an application that is already running. There are other ways to deal with this issue but we will discuss that in another article to avoid this one from being overwhelming.

In summary, Serverless computing is only supported in Cloud and you are only responsible for the application itself and the architecture around it. You don't maintain or even choose the server running these components and your application is scaled up and down depending on the traffic it is catering to. And thus, you only pay for the resources used by your application when it was serving requests, not when it was sitting idle doing nothing.

Okay, if you are still not excited and want reasons for why you should care about Serverless, here are a few I've summarized for you - 


- You only pay for what you use: The motive behind Cloud Computing was to save your money and not have to invest in infrastructure that you may not necessarily use. However, if you are building applications that are hosted in servers in the cloud, you are still paying for using this server even when no one is using your application, just to keep the application running. With serverless, this cost is eliminated as well. This is as close as it gets to "Pay for what you use". Don't get huge bills for that side project that you are building for fun or that SaaS application that is not gaining much traction yet. 

- You are not responsible for scaling: Scaling up/down your application should not be your overhead. If you think auto-scaling does it anyway for you, remember that auto-scaling still needs you to configure a whole lot of things before you can really use it. Here, auto-scaling comes out of the box for you.

- The scale is virtually all that the cloud provider can provide: Similar to the previous point, scalability is not going to be a limiting factor for your applications anymore. "Build to scale" is not optional here, it is the default!

- No operational overhead: No OS patching, no disk cleanups, no memory management. None of that. 

- Low time to market: There's a learning curve but once you are past that, building and shipping applications is pretty quick! And I've got you covered with how you can learn to build Serverless Applications in this series!

- Because it’s beautiful ❤: Innovation of this kind is what lured most of us into engineering. So let's learn, build, and deploy now!

So, that's Serverless in a nutshell. If you have any questions, you can comment them here or you can reach out to me on  [Twitter](https://twitter.com/TejanshRana) and I'd be happy to answer them. If you are excited to learn more about them, follow this series where we will build a Serverless application from scratch!



