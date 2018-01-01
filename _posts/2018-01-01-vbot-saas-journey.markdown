---
layout:     post
title:      "How I created a SaaS product"
color-css: "black"
subtitle:   "The journey of creating an UI testing SaaS product from the ground up"
date:       2018-01-01 12:00:00 +0800
author:     "Katat Choi"
header-img: "img/tree-cloud.jpg"
---

This post shares the journey of how I dealt with my own itch to improve my testing workflows. It discusses on what problems I faced during works and how I tried to get solutions to them. I started trying existing 3rd-party solutions. Then, I built an open source solution to better fit my own needs. Eventually,  its usage in team works motivated me to build a SaaS version.

## What is the problem?
Most of my projects are in full stack, which includes both the front end and the backend. The front end usually uses the JS MVC frameworks like, such as Angular.js or React.js. The backend is using Node.js stack. The front end connects with the backend using REST API architect.

For web SPA (single page application) apps, it is usually containing many states in a page. To reproduce one of these states, it may need many operations on the page in certain sequence.

Usually the front end is the central interaction point between the user and the backends. The testings of the backend may also need to go through the front end to verify the backend changes are compatible with the front end.

![diagram](/img/central-interaction-diagram.png)

If the depth of the target state is deep, I need to repeat a certain number of manual behaviors in the browser to verify. It is time consuming to do this sort of repeated manual tests. It kills my productivity and creativity as a developer.

![states-diagram](/img/states-change.png)

Does this sound familiar?

## Early try
I love unit tests, and create unit tests to the backend codes, but never for the front end codes.

So I gave it try to use the unit test concept for the front end. At that time, I was using Angular.js as the front end framework for most of the projects. I picked and tried the Angular e2e testing stack, [Protractor](https://github.com/angular/protractor) + [mocha](https://mochajs.org/
). It was the most popular stack to do automated test on the angular framework.

After some experiments, I realized this style of testing introduce other issues. It requires a tremendous amount of attentions to the testing codes. The testing codes are difficult to maintain and doesn't relieve much the pain from manual testings.

To have the tests to run repeatedly with this framework, it needs to create a lot of mocks for the backend API requests. It assumes the front end developers to have a very good understanding of the backend, and how it works with the front end. Quite often, there are magic codes added to the tests to pass the the tests. These magic codes are problematic and hard to maintain in a long run.

So, with this testing framework, it needs experienced developers to create good tests. Otherwise, the tests would become too fragile to maintain, and contribute more overheads than benefits to the development process. In practice, front end developers of a project could come from various backgrounds. They can be junior or senior developers.

The other problem is the testing are only verifying some parts of the logics. It doesn't test using the actual browser behavior. Passing all the tests doesn't give me the enough confidences. I still need to run manual tests to see if it works and looks as expected in the actual browser.

![close-door](/img/close-door.gif)

It makes me doubt if it worths writing codes with such kind of frameworks to test the UI codes.

## Try again
I am more align to what the book [Fifty Quick Ideas To Improve Your Tests](https://www.amazon.com/Fifty-Quick-Ideas-Improve-Tests-ebook/dp/B00XVFFK7E/ref=sr_1_1?ie=UTF8&qid=1514258592&sr=8-1&keywords=fifty+ideas+of+testing) emphasizes that the tests should focus on exploring the capability for the end users.

![Fifty Quick Ideas To Improve Your Tests](https://images-na.ssl-images-amazon.com/images/I/51DlCCMPOoL._SX260_.jpg)

I started looking for a simpler way to do the front end tests with  least overheads to development process.

There is. The other testing approach is called visual regression testing. It is like applying the [WYSIWYG](https://en.wikipedia.org/wiki/WYSIWYG) concept in the editor to the testing.

![robot-take-photo](/img/robot-take-photo.jpg)

> Basically, the idea is to compare the new UI screenshot with the old one. Then it automatically generates a image that highlights the differences between the screenshots.

When there is a difference detected, developers can review the difference image to see if it is as expected. To accepted a difference, developers only need to rebase the screenshot without the need to change the testing codes. This dramatically speeds up the testing process. It suits the human nature that our brain is much better reviewing the screenshots than the codes.

It turns out there were already several open source libraries to do this job. I reviewed some of them, picked the one called [BackstopJS](https://github.com/garris/BackstopJS) that seemed to easy to use to do testings for the UIs.

## Build a new one for my own itch
BackstopJS gave me a lot of inspiration of how visual regression test can help in the front end testing. I tried to use it to do testings against SPA (single page application) apps. It takes a final screenshot to do visual regression testing at the end of a scenario test. But, it also needs to write freestyle scripts to update the page's states. This introduces the same maintenance problems discussed in the early try section. It is very likely that the freestyle scripts will become messy to maintain in the end, particularly when there are more than one developers touching the testing scripts.

![fix-curtain](/img/fix-curtain.gif)

Also I need it to be flexible to take the screenshots and do comparisons whenever needed in any testing steps in a scenario, not only for the final step.


## Build one for my own itch
These experiments motivated me to wonder:
Why not formatting the common browser behaviors, such as clicking/typing/select/screenshot etc, into a standard JSON format. It would be easier to read and update for a long term than using freestyle testing scripts.

So I create an open source library called [vbot](https://github.com/katat/vbot). Since then, I have been using it for testings in my front end projects.

#### How my testing process has been improved?
1. No magic codes in the tests.
2. Use JSON defined in WYSIWYG fashion, much easier to refer to the actual browser behavior.
3. Review and approve the new screenshot, instead of updating testing codes to make the test pass.

![todo-diff](/img/todo-diff.png)

## Add ingredients for team work
Since I am also the user of the tool, it is easy for me to have the first hand feedbacks. The more I use it, the more ideas I have to improve and evolve it.

When it comes to using vbot in a team, the baseline screenshots need to be shared at somewhere. So the other team members can run the tests against the baselines.

Git is the initial place to store the baseline images. But these image files became too big to store on it. Furthermore, the screenshots generated for a same UI page from different development environment would be slightly different, that could be due to different development environment setups. Even it is a small different, it can produce false negative results.

It will be much easier if the testing results can be stored somewhere on a server, and can be reviewed simply via a browser. So it can be more efficient to communicate among the team by sharing the test results.

![vbot-cloud-diagram](/img/vbot-web-cloud.png)

Imagine a scenario, in which there is a failed test stored on a server, all the browser behaviors have been documented as well as the screenshots. To inform others the issue, it only needs to share an URL link, which the others open to review at the failed test. Even more convenient, they can download the playbook to run the test to reproduce bugs on their local machine when trying to resolve the issue.

## Go SaaS
There are multiple benefits in making the vbot as a service on the web, and I am motivated enough to build a [SaaS version](https://vbot.io) around it. Not only does it provide place for team members to share their tests, but also it serves as a compliment service to the open source version:

- It enables users to create the playbooks via UI, easier for new users to start with.
- Users only need a web browser to run the automated tests against public websites.
- The playbooks, created on the vbot.io, can be downloaded by the vbot CLI tool and run locally with a single command line. Easier to debug issues or reproduce a bug.
- Tests run under the same containerized environment on the server, reducing the odds of getting differences in the same tests due to the variety of development environment setups.
- Playbooks as well as the testing results can be shared with other users, easier for the team to communicate on the issues.

## Final words
The journey started in 2016, and the first version of vbot on the github was released in early 2017.  As a side project, I spent most of my off hours in doing iterations to make it better and easier to use. It took almost 6 months to build the SaaS version.

I wasn't expecting I would create an open source library in the testing field, neither create a SaaS product around it. It is an experimental process of building it from the ground up, during which there were many researches and try-errors practices.

I did have some doubts come from inside of myself in the process, wondering "Does it worth?". The main driven force of this whole journey is that it is a tool built for my own itch. I am glad it goes this far.

[vbot.io](https://vbot.io) is still in beta. It will need a lot of improvements to make it production ready.

This post shares a bit in the traces of my side project in UI testing. Hopefully it will be helpful for anyone who want to build their own projects. I will be happy to interact with any thoughts, please feel free to comment.
