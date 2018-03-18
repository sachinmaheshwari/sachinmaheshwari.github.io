---
title: "Single responsibility principle"
date: "2018-03-18"
layout: post
---

In this post, I will talk about the first principle of SOLID design principles i.e. Single Responsibility Principle. It says that every module or class should have only one responsibility. It is also one of the most misunderstood principles, due to the word "single" in it, most of us make mistake and while designing classes we think that to conform to this principle it should only have one public method along with some private methods. We think that it should do single work only.

Now having one responsibility does not mean it will do only one work or it should have only one public method (in case of class). Let us take one metaphor example to understand this. Suppose in some company there is an employee who is responsible for security. Now to fulfill this responsibility he might have to do a lot of work (lots of methods) in collaboration with lots of people (dependencies). He might have to give all the employees some badge that can be used to authenticate them to allow access, he might have to check that all the security cameras are working and so on. But the responsibility is still one i.e. security.

We should apply a similar methodology while designing modules/classes and their responsibilities. Each module and class should have only one single responsibility at their abstraction level.

Another way to think about this is figuring out who will be the person whose input can cause this class to change. If the answer has multiple persons then definitely you are handling more than one responsibility in that class. For e.g., If I'm doing some business logic related work and also doing some analytics capturing task in the same class. Then any change that can come from the business logic team (product) or analytics (analytics team) will cause my class to change. Hence the rule of single responsibility is voided in that class.
