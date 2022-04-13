---
layout: post
title:  "AWS Lambda Function URLs for C2 Redirection"
description: Leverage the new AWS Lambda Function URL feature to expose a function to the Internet and redirect red team command and control traffic.  
tags: redteam, AWS, C2
---

## Overview

Using features of cloud provides for command and control (C2) is not a new concept. 
That said, AWS has recently released a new feature called [_Lambda Function URLs_](https://aws.amazon.com/blogs/aws/announcing-aws-lambda-function-urls-built-in-https-endpoints-for-single-function-microservices/) which allows developers to **easily** expose their Lambda functions to the internet with a HTTPS url and an AWS-signed TLS certificate.
With a little coding and configuration, the AWS Lambda service and Lambda Function URL feature can make for a quick and easy C2 redirector.

## Background

AWS has a service called Lambda which allows developers to write event-driven code which can be triggered through various events. 
Some triggers could be other AWS service actions, a scheduled timer, or an HTTP(S) request. 
That last one sounds a bit interesting to red teamers! 
So how does one code a Lambda function and send HTTP(S) traffic to it?

## Developing a Lambda redirector

The concept of using Lambda as a redirector is not new at all.
In fact, [*@\_xpn\_*](https://twitter.com/_xpn_) published a blog post back in early 2020 titled [AWS Lambda Redirector](https://blog.xpnsec.com/aws-lambda-redirector/) which covers exactly how one could accomplish this (highly recommend you read).
To summarize the blog post a bit, Cobalt Strike can send traffic in three primary ways via HTTPS:

1. HTTP body
2. HTTP Headers
3. HTTP query strings  

Therefore, the Lambda Function needs to read in each of these characteristics and forward them to the actual C2 server hidden behind the scenes. To break this down step by step, the Lambda function has to perform the following:

1. Read the full HTTP request sent from beacon, including any POST body.
2. Build a new HTTP request ensuring that the received HTTP headers, query string parameters, and POST body (if included) received from the beacon are used.
3. Forward the HTTP request to the team server and receive the response.
4. Add the received HTTP headers and body (if included) to the API gateway response.
5. Forward the response to beacon.

Unlike \_xpn\_ who wrote their Lambda redirector in Go, I opted to write mine in Python.
The code used for the Lamda function can be found in my red-lambda GitHub project.

## Sending C2 Traffic 

With the lambda function written, we now need to start sending C2 traffic to it!
In \_xpn\_'s blog post, the AWS HTTP API Gateway service was used in conjunction with a Lambda function to expose it to the internet.
However, with the new _Lambda Function URL_ feature, developers do not need the AWS HTTP API Gateway and can directly expose the function to the internet.

### Lambda Function URLs

As stated by AWS in their [announcement](https://aws.amazon.com/blogs/aws/announcing-aws-lambda-function-urls-built-in-https-endpoints-for-single-function-microservices/) about this new feature, _"sometimes all you need is a simple way to configure an HTTPS endpoint in front of your function without having to learn, configure, and operate additional services besides Lambda"_.
Lambda Function URLs can be assigned to any Lambda function to provide quick and easy internet access. 
A randomly generated URL will be assigned to the function with the following scheme: `https://<randomid>.lambda-url.<region>.on.aws`.
In addition, the TLS certificate is valid and signed by AWS themselves which helps blend in with normal traffic.
Below is a screenshot of the lambda function url in action with the TLS certificate information shown.


![rhce-cert](/images/rhce-cert.jpg){:class="img-responsive"}
