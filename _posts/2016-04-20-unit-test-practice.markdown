---
layout:     post
title:      "Do Unit Test the right way"
subtitle:   "Unit Test itself doesn’t guarantee better works on the source code. Doing it in a right or wrong way could lead to the very different results."
date:       2016-04-19 12:00:00
author:     "Katat Choi"
#header-img: "img/post-bg-01.jpg"
---

There are numerous blog posts about why and how to create the unit tests along with the functions to be developed. I believe most of the developers understand the reasons of having unit tests. But I also believe, in practice, there is a large number of developers, from junior to senior, got stuck with the unit tests and start doubting about the payoff of making the efforts. Without getting any real benefits from making the unit tests, the developers lost the motivations to make that efforts. Even worse, in some situations, the unit tests built became a huge hassle for the developers to maintain.

In this post, I try to outline the possible hurdles faced during writing unit tests and suggest ways to overcome them. Also, I share the practical benefits I experienced. Hopefully, this would help unit tests methodology start making senses to you practically.

## Hurdles
The optimal time to build the unit tests is before writing any actual functional code. The unit tests should be written first, which is helpful to be served as specs and for that developers can start prototyping the functional codes based on these unit tests in the agile style.

In practice, the development schedules for most projects are highly limited. Many developers rush into the functional coding without making any efforts to unit tests. This seems making the development going fast in the early stage, but I can assure the headaches come along with the changes and new features added. Later, the development start getting slow down and is not as productive as the early stage. That is because as more changes needed, it requires more brain power, more manual tests to do the checks and more pressure to deal with. Eventually, this take more time than in the case where they got the unit tests in the beginning, and developers hate the codes written by themselves.

I have seen developers runaway from the unit tests at a certain stage, including myself. This comes to the question: Why the unit tests should be built at the earlier stage even before writing the functional code? One of the major benefits to have the unit tests is that it constrains the patterns of the actual code. It force the developers to think about the code structure when coding, making the code easier to be tested by the unit tests. This is extremely helpful to decouple the components of the software.

When there are unit tests need to handle a lot of assertions and are relating to more than one functions of the actual code, the unit tests start getting difficult to build and even hard to maintain. This probably imply there are changes needed to refactor the actual code to make it healthy. Once you got the code structured in the right way, the unit tests should be easy to build and maintain. So when you see the unit tests were difficult to maintain, it could be the signal to improve your functional code structure by refactoring.

Building Unit Test, in my opinion, is like paying taxes. It is neither good to do too much or less. It is kind of like a balance in terms of the project development schedules. In real world projects, there are many simple logics are too obvious to have unit tests. Writing unit tests for these kind of functions kills the developers’ motivation to contribute the efforts. I suggest focusing on the key components/logics in a project when building the unit tests. This way, developers can begin the works from the core parts of the projects, and able to prototype quickly around the core functions.

Prototyping around the core functions along with the unit tests, giving the developers a sense that they are making the most valuable works from the beginning, enabling them to notice the technical issue and understand the business logics quickly. The unit tests can be served as the notes of the key specifications, while they give the developers the confidences to do the necessary refactors when they faced major changes from the business side.

## Benefits

I think most of you who read this post know and believe the DRY(Don’t Repeat Yourself) rule, even you are not a developer. Repeating manual tests destroy the motivation and energy of developers on the projects. Talent developers meant to do the creative works. Reducing the amount of the repeated works gives developers more time on the more valuable and creative stuffs. This also makes developers happier and be more productive on the projects.

If you do it right, Unit Test is an useful way to encapsulated the tests in a place, reducing a huge amount of time on the repeated manual tests and debugging. When there are certain issues needed to be re-created and debug, we can just create the unit tests for them to quickly do the debugging at the relevant parts. It is much more efficient than doing manual tests with the whole process, most of which are un-related to the issues we are trying to resolve in most of the cases. At the same time, you have supplemented the unit tests to make the program more robust.

Are you dare to change the complicated code even written by yourself? Unit Test is like doing proof reading on the text writings. It tries to further check at the writings and make sure it looks right before sending it to the readers. Different from the text writings, Unit Tests for the program code checks the key parts of the system, and try to ensure it is not going to break down when it goes production. When developers can write the code following the flow of their mind and let the unit tests to validate the new code doesn’t break the existing ones, the process of writing the code would be more enjoyable, more productive and creative.

Continuously refactoring is really the natural of the software development, particularly for the large/long term projects. Unit Test help developers to be more confident to do the refactors.

There are situations where developers don’t really know where to start coding as the key functions, even they got the detailed documentations on the business requirements. Learning by doing would help people understand the business domain more quickly. In text writing, when you haven’t gotten the clear ideas on the parts you are trying to convey on a topic, writing the drafts following your mind is helpful to get a better sense of what you really want to talk about. This is similar for the coding in the fuzzy situation.

Building the key unit tests at the beginning even you are not sure whether these are really the key ones, would help the developers’ mind flow. After a few attends of building up the key parts of the software, it would help understand what the things are missing and which parts should be the key to focus on. It makes the developers to think practically and raise questions at the early stage.

In the team works, there are many chances for the team members to change the code built by others in emergency. Even it is a small change needed to be done, it could cost some time for the other team members to get the grip of the codes’ logics in order to make the change. As I mentioned earlier, if there are unit tests built for the key parts of the logics, the other team members can quickly get an overview of the key logics by looking at the assertions of the unit tests. They can also drill down to the details of the logics by playing around with the existing tests very quickly without doing lots of bored manual testings.

With the unit tests, the developers are more confident to make changes to the code not originally created by them. This implicitly create a team culture to share the knowledge of the business requirements, as well as the source code logics between the team members, so as easier to make up each others when there are emergent changes needed. There is a post talking about how to bring in the unit tests culture to the team.

## Summary
Unit Test itself doesn’t guarantee better works on the source code. Doing it in a right or wrong way could lead to the very different results. It requires a certain number of attends to get a better sense of how the unit tests should be fit in your projects.
