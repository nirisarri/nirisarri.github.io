---
layout: post
published: false
title: Install and run Minikube on a Virtual machine
date: '2020-02-12'
tags:
  - Kubernetes
  - Minikube
  - Hyper-V
---
*It is a fact: [Kubernetes](https://kubernetes.io/) won*.  

Now, from the development standpoint this poses a nice problem, which is related to the toolset necessary to do development in an inherently distributed stack: Kubernetes assumes highly distributed applications, providing orchestration so the multi-server deployment is seamless and the obstacles of deploying distributed apps are minimized.

In my task to understand how this affects development, I am in a process to understand this technology and see how this will affect my system architecture and development strategies.

In an internal training I am taking in my company, te first task was to set up a [Minikube](https://github.com/kubernetes/minikube) environment. Minikube is a minimal Kubernetes deployment that allows you to locally test and develop applications that later will be deployed to a Highly scalable environment.  The instructions focused on installing a windows native minikube; which installs a VM and manages it through the CLI. 

The recommended toolset includes installing all the prerrequisites ([Kubectl](https://kubernetes.io/docs/reference/kubectl/overview/), [Minikube](https://github.com/kubernetes/minikube), [Docker](https://www.docker.com/products/docker-desktop), [helm](https://helm.sh/)) in my bare-metal Windows 10 machine, and use the git bash console to run commands, as they are really similar to what you need to run in a linux prod environment.  The problem I see with this is the fact that git bash is a pseudo-emulation that runs a subset of commands and has to jump through some hoops in order to run windows commands (related to the pseudo-emulation). 

My first idea to solve this issue was to use the wonderful WSL2 and install everything I needed inside it to be able to use a real bash in a real linux environment. Unfortunately there are some [complications](https://github.com/kubernetes/minikube/issues/5392) that are to be resolved by the Kubernetes team and are a show-stopper when installing minikube this way.

The available option is to use powerShell instead of bash. I mean, it works, but you want to be as close as your deployment environment, right? this is what we try to do with containers and with WSL. So I found an alternative: Run a linux VM in my windows 10 machine, and install everything there! also it has the added benefit of enclosing all in a sinlge VM that I can turn on or off at will, and I can move away from my box or distribute along my dev team! 

There are a couple issues that need to be solved in order for this strategy to work, but as far as I can tell they are working and they are documented all over the place, so I will try to aggregate all the information and provide a nice recipe for creating a VM that runs Kubernetes via minikube.

## Overview

In order to have your instance running we need to perform  the following steps.  Some of these will be windows-only but others are platform independent: they can be performed on mach or windows hosts.

1. Ensure Hyper-V is running (windows Only)
2. Create an ubuntu VM
1. Install KVM2
3. install Kubectl
4. install minikube
5. configure minikube

Lets take a look at each part:
