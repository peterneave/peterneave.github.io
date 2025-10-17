+++
title = "Dev Proxy"
categories = ['development']
tags = ['tools', 'development', 'code']
date = 2025-10-17T14:42:27+11:00
draft = false
+++

[Dev Proxy](/devproxy) is a nice little tool written by Microsoft that allows you to test our your web application beyond the happy path. You proxy your web requests through it and it can perform some action for you. There are a bunch of [plugins](https://learn.microsoft.com/en-us/microsoft-cloud/dev/dev-proxy/technical-reference/overview).

I had been meaning to learn more about it but I couldn't get it to work for local development until I realised that Chrome based browsers don't usually send localhost request via the proxy. You have to explicitly set it up by completely terminating your browser so there aren't any background processes and [launch it with the right parameters](https://learn.microsoft.com/en-us/microsoft-cloud/dev/dev-proxy/how-to/intercept-localhost-requests).

Learn more about Dev Proxy with this slide deck.

{{< iframe src="https://neave.dev/devproxy" >}}
