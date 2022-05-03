---
title: "How to build a great software"
date: "2022-05-03"
layout: post
---

Building a great software requires some preparation before you start the coding work. We need this preparation even in agile or TDD approach. It will also help in reducing the re-work that we might have to do at a later stage of software building. After working in this industry for about 15 years, here are a few patterns that I have observed while building great softwares.

## Features

<img src="./assets/how-to-build-great-software/thisisengineering-raeng--68PdYVLh3U-unsplash.jpg" alt="A person writing on a whiteboard" width="100%" display="inline-block"/>

Photo by <a href="https://unsplash.com/es/@thisisengineering?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">ThisisEngineering RAEng</a> on <a href="https://unsplash.com/s/photos/whiteboard-discussion?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

Always start by creating a feature list of the software that you are planning to build. Do not think about how to implement such features at this stage. Speak with your customers to find features. Sometimes, you might need to include some non-functional features as well, such as high performance, availability, secured, etc. Add such features in your feature list. Once you have such a list ready, you will have an answer to "What your software is supposed to do?" question. Get this aligned with all the stakeholders.

Let's say we are building a software for tasks management, then a feature list may look like this:

- Add new tasks
- Mark task complete
- Delete a task
- Generate a report of the tasks completed (daily, weekly, monthly, yearly, custom dates)
- Update a task
- Show incomplete tasks
- Show complete tasks
- Fast UI

## Use cases

<img src="./assets/how-to-build-great-software/generic-use-case-diagram.drawio.png" alt="Generic use case diagram" width="100%"/>

Next, focus on figuring out the use cases to cover all the features in the list. These use cases will allow you to understand more about your software and the details about how users will use this software in the real world. It will also bring out your thoughts related to the actor for each feature. This step will uncover the answer to this question "Who will do what in your system for each feature?".

For example, for our task management software, it may look like this:

- The customer will add new task using some UI.
- The customer will mark the task as completed using some UI.
- The system will generate a report of completed tasks using the input of dates provided by the customer.
- The system will show the incomplete tasks to the customer.
- The system will show the complete tasks to the customer.

## Create a high level design

Break it up. You need to figure out the key components of your system that you should create to cover up all the use cases and features you have listed in the last steps. It deals with how you are going to code your software. Follow guiding principles and best practices while deciding the components of your software.

Few principles which are highly used during this stage are:

- Single Responsibility Principle
- Don’t Repeat Yourself
- LSP (Liskov Substitution Principle)
- Encapsulation
- Delegation
- Polymorphism

## Low level design

Pick one use case from the list and then write detailed steps required to complete that use case. Write all the steps that your system needs to perform to complete a use case. Also add steps to validate the problems that might come because of user errors or anything.

Perform textual analysis to figure out the classes and their behaviors. Look for nouns and verbs. This will give you a preliminary design to start with. Your design decisions should be based again on guiding principles and best practices.

## Implementation

<img src="./assets/how-to-build-great-software/thought-catalog-UK78i6vK3sc-unsplash.jpg" alt="A person working on a laptop" width="100%"/>

Photo by <a href="https://unsplash.com/@thoughtcatalog?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Thought Catalog</a> on <a href="https://unsplash.com/s/photos/writing-code-on-laptop?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

Implement the code. You might have to revisit your original decision at this stage. Your design decisions should always improve your implementation, it should never make it more complicated or harder to understand. You should only expose clients of your code to the classes that they need to interact with. You should also write test cases here to show that your code works!
