---
layout: post
title: Xamarin Forms - loading your app
published: true,
thumb: "assets/images/blog/blog-post-thumb-1.jpg"

---

Xamarin Forms is great, but getting an app up and running with some more advanced scenarios cna be frought with error. IoC, bespoke start-up scenarios, splash screens, all peices that need to be managed. Fortunately for you, I have the answer. This guide will show you the best way I've found to get an app started all while showing a custom loading screen while the app gets itself going. 

### Setting up an IoC Container

While not strictly neccessary, a good IoC configuration will make your app more testable and more easy to configure in a component structure. With recent versions of Xamarin, we're able to levarage ServiceCollections to get everything set up, and then have these services auto resolved within your view models and dependent services. 
