---
layout: post
title: "API Versioning - Identifying Breaking Changes"
date: 2020-06-04
tags: programming coding C# API REST
---
# API Versioning - Identifying Breaking Changes

There are number of ways of targetting a .net core REST API at a particular version, including `path`, `querystring` and `header`. But how do you indentify breaking changes to instigate a bumping of the version in the first place?

1. **Developer hygiene**, developers understand what constitutes a breaking change in the context of their API. This has the advantage of being early in the process, but has all the disadvantages inherent to a manual process, and is dependent on developer experience.
2. **CI**, tooling/steps introduced into CI process to identify breaking changes. This has the advantage of being automated and therefore reliable and repeatable, and can include automation of the breaking change process (version bumping, release notes etc). 
3. **Integration Testing**, this may be part of the CI process, or take place later in the pipeline, in either case a comprehensive set of integration tests across the entire surface area of the API is likely to surface any breaking changes.
4. **QA**, manual regression testing of the API consumer(s) will surface both breaking changes to the API and regression in the product caused by non-breaking changes to the API.

A combination of the above is ideal, however in common with unit testing the more issues that are found and addressed in steps 1 and 2 the better. In this blog I want to discuss the techniques I have used for identifying breaking changes to APIs in the **CI** step (without using integration testing, which can dramatically slow the CI process).

1. **Unit Testing**, controllers can be unit tested, controller methods are public and can be invoked by unit tests in the same way as any other class public method (ish). However unit tests may not identify all breaking change to an API, and/or developers may have updated the unit tests to pass with the new _broken_ API.
2. **API Documentation**, tools like [Swagger](https://swagger.io/) generate JSON documentation for the API definition. This can be tested against the previously generated document, to show any differences between the two. However, this will highlight all changes, not just breaking changes, and its not trivial to indentify and address each change individually. We definitely used this as a blunt instrument, and I'm fairly certain there are better ways of using OpenAPI to manage breaking changes.
3. [**APICompat**](https://github.com/dotnet/arcade/tree/44a53c66de431dbd54b4277d6338d2b103d6852d/src/Microsoft.DotNet.ApiCompat#microsoftdotnetapicompat), this is a tool which only recently came to my attention through an excellent introductory blog by [Stuart Lang](https://stu.dev/check-for-breaking-changes-with-apicompat/). This identifies all breaking changes to an API at build time, with the option to _ignore_ the breaking changes and append them to a baseline document, which can then be added to any release notes when the new version of the API is made available. I've started to play with this to better understand its capabilities and think it could be a fantastic addition to the **CI** process for any .net core API. Its worth noting that it doesn't identify changes to HTTP methods (ie where a controller method has its HTTP method attribute changed). However this is as you'd expect, the tool inspects public methods directly, and not via controller attribution and middleware. But other than that its found every _test_ breaking change I've introduced, including changes to the object graph.

I'd be interested to hear whether others have experience of integrating [APICompat](https://github.com/dotnet/arcade/tree/44a53c66de431dbd54b4277d6338d2b103d6852d/src/Microsoft.DotNet.ApiCompat#microsoftdotnetapicompat) into their **CI** process, or any other suggestions for API breaking change identification.