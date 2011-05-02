---
title: Please do REST right
layout: post
---

I recently received an email from a large email delivery company
bragging about a great new RESTful API, which got me quite excited
because their current offering is pretty poor.

When I read the following though I became a little concerned:

> #### REST & JSON FORMATS
>
> Our new RESTful API mediates XML to REST and JSON formats for easier
> adoption and integration.

That sentence doesn't read very well to the point of almost being
completely nonsensical.

A RESTful API should use HTTP methods to interact with resources. When I
want to know about all users I should `GET /users`, when I want to know
about one user I should `GET /users/some-unique-identifier`, when I want
to create a user I should `POST` to `/users`. Mixing up these methods or
verbs and adding superfluous path parameters is not an example of a good
RESTful API.

If you want to do it a different way that's fine. Just don't call it
RESTful if it's not.

[Representational State Transfer on Wikipedia](http://en.wikipedia.org/wiki/Representational_State_Transfer#RESTful_web_services)
