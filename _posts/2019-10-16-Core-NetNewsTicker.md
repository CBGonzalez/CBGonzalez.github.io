---
layout: post
title: Ported WPF News Ticker into Net Core
excerpt: A painless port of the newsticker to Net Core 3.0
date: 2019-10-16
categories: [.net, MVVM, netcore3]
---
A while ago I made a newsticker in WPF in order to learn the MVVM pattern. After the release of Net Core 3 I decided to port it and see how difficult it would be (the source code is available in [my repository](https://github.com/CBGonzalez/NetNewsTicker)).

#### Preparation ####

Microsoft has a [document](https://docs.microsoft.com/en-us/dotnet/desktop-wpf/migration/convert-project-from-net-framework) describing how a port from framework to core can be performed. It should be your starting point.

In my case, since I had no external dependencies (i. e. no non-Microsoft Nuget stuff) to worry about, the only thing I needed to do beforehand was to decide if I wanted to create a totally new solution in VS2019 or if I just wanted a new project. Being the economical (lazy!) type I decided to just build a new project inside the existing solution in order to change as little as possible.

Using `dotnet new wpf` in a temporary directory, I had my `csproj` file, which I copied into the solution, ignoring all the other files the command created. As described in the document linked above, I copied over some stuff from the original file (namespace related stuff so all files could be used as-is), the references to images, and added the project to the solution.

A first (very optimistical) build resulted in several errors, mostly related to the RSS stuff I use. Adding [`Microsoft.Windows.Compatibility`](https://docs.microsoft.com/en-us/dotnet/core/porting/windows-compat-pack) as a Nuget package solved most of the errors. What was left were weird errors related to double declaration of classes and variables, which had me puzzled for a while.

The document from Microsoft actually had the answer to that too: it turns out that a project file for net core will include all source files it can find in the solution's directories (as opposed to a framework project which has explicit mention of all the source files). I had an old folder with some test within the solution which was creating confusion. Deleting that solved the weird stuff.

#### Run ####

The next build showed no errors. A run in debug mode almost immediately crashed as I clicked on an article to open it up in the browser. It was a simple call to `Pocess.Start(url)` which in framework does the magic, given a well-formed url is passed.

It turns out that core is finicky and requires a different approach (described [here](https://brockallen.com/2016/09/24/process-start-for-urls-on-net-core/)). You basically start `cmd` and pass the url to it:

```
          string cleanUrl = secondary.Replace("&", "^&");
          Process.Start(new ProcessStartInfo("cmd", $"/c start {cleanUrl}") { CreateNoWindow = true });
```

After that no more crashes and all seemed well.

#### Build Take two ####

Attempting to build the original project (framework) turned out to be a chore. Several errors related to missing Nuget packages and assorted weirdness.

As it turns out (and is mentioned in the MS doc), keeping projects side-by-side will have the unfortunate side-effect that both will trample on each other's feet by sharing the `obj` folder. The lazy solution was to unload both projects, delete the folder, then reload the framework project and all was well. MS recommends to eliminate the original project and add add the version of the framework you target to `<TargetFramework>` in the new project file.

I might try that next, but for now the migration is done. As mentioned above, it's really remarkably painless for a simple app.
