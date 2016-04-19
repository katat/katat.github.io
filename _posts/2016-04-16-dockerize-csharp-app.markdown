---
layout:     post
title:      "Dockerize C# Application"
subtitle:   "Start and run C# application with one command on Linux system"
date:       2016-04-16 14:16:00
author:     "Katat Choi"
header-img: "img/docker-csharp.png"
---

## Why Dockerize?
No doubt Docker is quickly gaining the popularity in the DevOps, becoming the must-to have tool box, no matter in the development or production environments. With Docker, system environment can be version controlled, easily cloned for new instances or setting up the development environment for new developers with minimum repeated hassles in the processes.

It is quite common to see the same code works well on one machine, and not another. This creates panics in the team, and harms to a team's morale, because it wastes the time on something that is not valuable.

I was having large legacy C# CLI application, while the new features are developed in another tech stack that is not in the Microsoft platform. Rebuilding it with new tech stack would require a lot of efforts. I ended up managing two suits of applications in two different operation systems.

#### Then why bother to run an application, which works well on Windows, on Linux?
 If there are more than one operation system involved for the applications, it means developers need to understand the both. This requires more man efforts and cross-stack skills, increase the costs resulted in context switch. Eventually it complicates the development works. In this situation, dockerizing C# application helps simplify the daily DevOps. Also, Docker helps simplify standardize the development environment, most of the time, developers are able to start up the application with one command even there was zero-configurations done before hand.

This post will demonstrate how we can start and run C# application as a docker container on linux systems, in just one command. We will use the Mono and Docker as the foundation.


## What is Mono?

> Mono is an open source implementation of Microsoft's .NET Framework based on the ECMA standards for C# and the Common Language Runtime. A growing family of solutions and an active and enthusiastic contributing community is helping position Mono to become the leading choice for development of cross platform applications.

Mono compiles .NET applications, supports most of the .NET features, and of cause, it supports on Linux or Mac.

In addition to this compilation toolkit, there is a cross-platform IDE [MonoDevelop](http://www.monodevelop.com) to facilitate developing the .NET applications on platforms other than on Windows.

## Setup
Supposes we have a simple C# CLI application, in order to run it with just one command one linux, these are the tools will be needed:

1. [docker-compose](https://docs.docker.com/compose/), helpful for version control the docker commands.
2. [mono](http://www.mono-project.com/)
3. [expect](http://expect.sourceforge.net/), useful for automating inputs in CLI.


---------------


### Basics
Below is the simple C# code we want to run inside Docker container. I have shared the [code](https://github.com/katat/dockerize-csharp.git) used in this post on github, feel free to pull it and play around.

```csharp
using System;

namespace dockerizecsharp
{
	class MainClass
	{
		public static void Main (string[] args)
		{
			Console.WriteLine ("What is your name?");
			var name = Console.ReadLine ();
			Console.WriteLine ("Hello " + name);
		}
	}
}
```

Prepare a Dockerfile as below to build the application environment. It basically install the mono packages support building and running the C# application.

```bash
FROM ubuntu:latest

# Install mono packages
RUN apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 3FA7E0328081BFF6A14DA29AA6A19B38D3D831EF
RUN echo "deb http://download.mono-project.com/repo/debian wheezy main" | tee /etc/apt/sources.list.d/mono-xamarin.list
RUN apt-get update

RUN apt-get install -y mono-devel
RUN apt-get install -y mono-complete
RUN apt-get install -y referenceassemblies-pcl
RUN apt-get install -y ca-certificates-mono
RUN apt-get install -y expect

# Create app directory
RUN mkdir -p /usr/src/app
ADD . /usr/src/app
WORKDIR /usr/src/app
```

Use the following commands to build the C# docker container.

`docker build -t csharp .`

The following command will get into the container's shell with a `mono` prepared environment, where it can build and execute the .NET executables.

`docker run -it csharp bash`

Build the .NET code with xbuild toolkit, then it should generate the `exe` file.
`xbuild /t:clean&&xbuild /p:Configuration=Release`

Run it using `mono`.

`mono dockerize-csharp/bin/Release/dockerize-csharp.exe`

You now should see the C# CLI up and asked to type in your name.

Suppose your application requires entering inputs before it executing something, and you want to automate this process. We can use [expect](http://expect.sourceforge.net/), which automatically enter inputs when the predefined text matched with the prompts. Below is `expect` script

```bash
#!/usr/bin/expect -f
set timeout 100
spawn mono dockerize-csharp/bin/Release/dockerize-csharp.exe
expect "your name"
send "kata\r"
interact
```

To simplify the commands above, they can be organized into an entrypoint script file, then all the commands are encapsulated into the following docker run command. It should build and execute the .NET application.

`docker run -it --entrypoint=./run.sh csharp`

What is more? We can even simplify it further with `docker-compose`, which can save the configurations of the docker container at runtime in a `yml` file. That means the docker execution commands can be version controlled as well, very nice!

The docker `build` and `run` configuratinos can be samed as below `docker-compose.yml` file.

```yml
csharp:
  build: .
  entrypoint: ./run.sh
```

Now, it should needs one command to build and run:

`docker-compose run csharp`


## Summary
This post demonstrate how to dockerize a .NET app, by encapsulating lots of environment setup commands into a single command, also allowing it to be version controlled. It could ease a lot of DevOps works and avoid lots of panics during setting up the system environment, if you are in the similar situation where you have codes from both Windows and Linux worlds.
