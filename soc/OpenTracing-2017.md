### OpenTracing

前言： 2017 Google Summer of Code 活动已经启动了，参与地址为https://developers.google.com/open-source/gsoc/ 。CNCF基金会是该项目的201个参与组织之一，而OpenTracing作为CNCF的重要项目，也提供了相关的项目课题。欢迎大家参与。

[OpenTracing](http://opentracing.io/) is an open standard for distributed tracing. A trace tells the story of a transaction or workflow as it propagates through a (potentially distributed) system. OpenTracing makes it easy for developers to add (or switch) tracing implementations, by offering a vendor-neutral APIs that popular platforms can bind to.

With v1.0 of OpenTracing complete, we are now in the process of intrumenting major web frameworks, services, and networking/controlflow libraries. See [Instrumenting Frameworks](http://opentracing.io/documentation/pages/instrumentation/instrumenting-frameworks.html) for more information.

In addition to the following projects, students may choose an equivalent framework, library, or service, provided it is widely used and Open Source.

#### OpenTracing adaptor for AWS X-Ray
* Description: X-Ray is a distributed tracing service provided by AWS. X-Ray intrumentation does not currently conform to OpenTracing, it provides a similar (but proprietary) API. Make X-Ray vendor-neutral by building an OpenTracing/X-Ray adaptor, so that it can be plugged in to the OpenTracing ecosystem.
* Recommended Skills: golang
* Mentor(s): Ben Sigelman (@bensigelman)

#### OpenTracing Instrumentation for Nginx, HAProxy, or other Load Balancers
* Description: Load Balancers and Gateways provide a variety of important services, and are present in almost every distributed system. Instrumentation at this layer is often the best first step towards tracing an entire system. Add intrumentation to Nginx, HAProxy, or equilvant gateway service via an OpenTracing plugin. See [Envoy](https://lyft.github.io/envoy/docs/intro/arch_overview/tracing.html) as an example.
* Recommended Skills: C++, nginx
* Mentor(s): Paul Draper (@pauldraper)

#### Add "net/rpc" and "database/sql" support for Golang
* Description: Instrumented wrappers for the io functionality in Golang's stdlib, such as net/rpc and database/sql, is critical. So far, only net/http has been instrumented.
* Recommended Skills: golang
* Mentor(s): Paul Draper (@pauldraper)
* Issue: https://github.com/opentracing-contrib/go-stdlib/issues/8

#### go-restful + OpenTracing
* Description: Instrument go-restful (https://github.com/emicklei/go-restful), a wildly used Golang REST web services framework, with OpenTracing.
* Recommended Skills: golang
* Mentor(s): Wu Sheng (@wu-sheng)
