---
title: "Code review"
date: "2023-09-27"
layout: post
---

<img src="./assets/code-review/alvaro-reyes-fSWOVc3e06w-unsplash.jpg" alt="Two persons looking at two monitors" width="100%" display="inline-block"/>

Photo by <a href="https://unsplash.com/@alvarordesign?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Alvaro Reyes</a> on <a href="https://unsplash.com/photos/fSWOVc3e06w?utm_source=unsplash&utm_medium=referral&utm_content=creditCopyText">Unsplash</a>

## What is code review

Code review is an activity to view or read the code written by a peer to ensure there are no gaps in the desired quality. In a nutshell, it is a software quality assurance activity. It can happen online by viewing the code via a pull request or it can happen offline by checking the code directly on the code author’s screen.

## What to check in code review

We should check at least the following things during a code review. This list is not comprehensive and may change as per your use case.

1. **Readability**
   - The code should be readable. Any code which is too cryptic to read can hide bugs and make it difficult to maintain. Even the person who wrote this code will have a hard time remembering and figuring out what’s happening in it after a few months of gap. As a software developer, we spend most of our time by reading the code and therefore it is at the top of my list.
2. **Functionality**
   - For every code change, there is a reason. In this aspect, we check whether the code is doing what it is supposed to do. The functionality could mean different things for different purpose. For e.g. it could mean performance improvement, logging changes, plain feature implementation.
3. **Performance**
   - We have to be cautious of the change and verify if the recent change is causing any performance degradation. It could be a badly written for loop or recursive function. Or we could relate it to terrible object handling.
4. **Multi-threading**
   - Ensure that the code will work as desired in a multi-threading environment. Most of the systems we usually work with support multi-threading. We should ensure that the new code is not causing any race condition or unexpected behavior during multi threaded execution.
5. **Exception handling**
   - The exception handling is something that requires special attention. If the code author has failed to handle the plan B in case something goes wrong, then it will definitely go wrong in production.
6. **Memory utilization**
   - Make sure that there are no potential memory leak issues in the code. Some examples that can cause memory leak: Recreating long lived objects such as Executors in Java instead of re-using them.
7. **Corner scenarios**
   - Usually developers are busy while handling the happy scenarios and they sometime miss the corner scenarios. Try to find the corner scenarios and verify and run it to figure out that the system is behaving as expected for all scenarios (including the corner ones).
8. **Logging and monitoring**
   - Ensure that the logs are proper. They will be your ear and eyes when the code is working in a production environment. Without it you will have no way to debug if something unexpected happens. Also, ensure that monitoring is in place to monitor various aspects of your system.

## When to do the code review

There are multiple places when you can do the code review:

1. At the time of code writing - With this you can give the feedback early as soon as the author commits the code. The disadvantage of this approach is that you will have to be constantly involved in the code review process. Also, you may not know whether the code in the current state actually works or not.
2. Before testing of code - With this approach, you can continue to review the code while the author is testing it for its correctness. Any changes required to make it correct will automatically get added to your code review process.
3. The code is ready for release - This approach may backfire if the code author missed something important while writing the code as it would mean lots of re-work. But if the author missed nothing, then it would be faster from a code review point of view.

 
  My opinion is to involve the code reviewer as early as possible. Even while we are figuring out the design for your changes. This way there will be very less chance of surprise when we get any review comment. The review will be faster and the learning will also improve for both code author and reviewer.

## Tips and tricks

The following tips and tricks have helped me as both code reviewer and author:

1. Write comments in the code if you are doing something unexpected or complex. Give the source of information in the comment. For e.g., if you are doing some optimization that reduced the readability of your code, then give sources of information which proves that the optimization done is actually worth the cost of readability. There could be more such examples unrelated to performance such as doing some extra work because of the limitation of any downstream service.
2. Try to finish the review as soon as possible to unblock the author and reduce the cost of change. The longer we wait for the review to complete, the cost of change will keep on rising as the author would have to re-test the whole flow if something major changes because of the review comments.
3. Never ever feel that the code review is about you as a person. You should always take any constructive feedback in code review professionally, as it's always about the code and never about you as a person.
4. Be professional in code comments, give no personal comment.

