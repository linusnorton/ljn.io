---
title: Is it all Scrum's fault?
date: 2018-04-23 07:00
lastmod: 2018-04-23 07:00
description: Thoughts on the software development process, Scrum and the difficulty of developing software.
layout: post
---

Scrum is the closest thing we have to a generalised software development process, but many people are unhappy with it. Recently I read through [why Scrum is the wrong way to build software](https://medium.com/@ard_adam/why-scrum-is-the-wrong-way-to-build-software-99d8994409e5), it's well written, clear and contains many valid points. It also got me thinking about Scrum and what issues I've seen with it.

I'm not dogmatic when it comes to software development process. I'm happy to implement [Kanban](https://en.wikipedia.org/wiki/Kanban), [Scrum](https://www.scrum.org/resources/what-is-scrum) or no process at all, so when blog posts criticizing Scrum surface I'll always give them a read because they give me an insight into other how other teams work, and of course, the problems they come up against.

One common factor in all the complaints I've read about Scrum is that people have a hard time differentiating problems with their implementation of Scrum with problems with the Scrum blueprint. Let's go through some complaints and try to dissect them.

*By the way, if you are expecting a pro-Scrum or anti-Scrum post you will be disappointed.*

```Because all product decision authority rests with the “Product Owner”, Scrum disallows engineers from making any product decisions and reduces them to grovelling to product management for any level of inclusion in product direction.```

This is not inherently true, but it is common. I've worked in companies where the product owner was the line manager of the whole team and companies where they are just another member. To a degree you are reliant on the product owner being benevolent and accepting the feedback of developers. The theory is that the solutions to "the problem" should be discussed with the product owner so if members of the team have a better solution or a compromise solution that can be done in less time then it can be adopted.  

I think the issue the original author has is that the product owner ultimately decides, so you are reliant on them being a good product owner - managing quality of the product against time and resource available, not to mention the egos in the team. The problem is that if the product owner does not have final say then who does? Some kind of democracy would be very messy to implement and lead to design by committee.

My recommendation is that if you are that product owner and come up against a situation where developers are arguing passionately for a particular solution, be thankful you have developers that feel so strongly, try to understand their reasoning (validate them) and explain why you agree or disagree. It can be very disheartening as a developer to feel strongly about the product you are working but not feel like your input is being heard.

```Scrum, in accounting for all the engineer’s time in a tightly managed fashion, discourages innovation — which typically occurs spontaneously and outside of any schedule or system of good predictability.```

This is something I can say is true but not necessarily an issue with Scrum. When you are in a time boxed situation where you are aiming to complete a number of features in a given time you will often accept the first solution that you come up with, but many times the first solution is not the best solution. I find that the software I write in my spare time is often much better quality because I spend more time thinking and less time coding.

If I get stuck with a particular area of the code I will leave it for a few weeks to sit in the back of my mind, more often than not I will come up with a better solution. Unfortunately this process is not viable when working with other people that depend on your contribution or when features need to be delivered quickly. So the issue is not Scrum but a workplace environment. Putting tasks in an n-week sprint just exacerbates the problem.

```Scrum encourages “least amount of work possible” solutions — to conform to it’s strict predictability requirements.```

Again, this is true but I would argue it is a good thing in the workplace environment. Proponents of Agile (and Lean) have identified that the biggest source of waste is building the wrong thing, so working out whether you are building the right thing is very important. This means that you have to deliver the absolute minimum to work out if you are building the right thing.

This is an area where context is incredibly important, you may already know that what you are building is "the right thing" or you may not be delivering features.

```By dividing every task into small items that can theoretically be completed by anyone on the team, Scrum discourages engineers from taking pride in and/or ownership of their work```

I don't see the connection there. Just because anyone in the team can complete a task doesn't mean someone wouldn't take pride in completing it.

One issue I do see related to this is that it is quite common for developers to each pick a user story and go off into their corner and develop it. It's important to remind developers that they should all cluster around the single more important user story until it is done before moving onto the next one.

```Scrum is highly intolerant to modification, and it’s proponents typically espouse an all or nothing attitude in it’s implementation.```

I would say a more common issue is [ScrumBut](https://www.scrum.org/resources/what-scrumbut), partial implementation of Scrum that results in some Frankenstein process. Another is using Scrum in situations where it does not apply.

```Scrum is very management heavy. Typical teams have Product Owners, Scrum Masters, and Team Leads. Innovative, motivated teams operate better with less management, not more.```

This is an implementation issue. Scrum only mandates a Product Owner and Scrum Master, neither of those have to be full time jobs. At the end of the day you need someone to decide what is being built and someone to ensure that the process is working and there is a feedback loop to ensure continual improvement. I've seen successful lightweight Scrum implementations and the things I would note are, the teams a usually on the smaller side (around 5 people), they are all knowledgeable about the thing they are building and they all communicate frequently outside of planning/standup/review.

```Scrum is typically implemented with HORRIBLE task management tools (Jira, tfs, etc…) that enforce very bureaucratic interpretations of Scrum that waste huge amounts of developer time. Additionally, they effectively lock you into one mode of operation, no matter how ineffective.```

It is a common pitfall but if you are in a colocated team, nothing beats a whiteboard. The product owner might have a list of stack of cards somewhere but when it comes to the sprint a whiteboard can't be beaten.

```Scrum discourages bug fixing, reduction of technical debt, and risk taking, all because of its narrow, exclusive focus on only doing items that Product Owners would interpret as valuable.```

I think this is the biggest issue with Scrum. The Phoenix Project book [describes four types of work](https://uptakedigital.zendesk.com/hc/en-us/articles/115000524374-4-Types-of-Work-in-IT-The-Phoenix-Project-): business projects, internal projects, changes and unplanned work and Scrum only works for business projects.

When the team is actively trying to juggle support of a live product and building new features it leads to trade offs between a stable system and perceived "progress". It is the product owner's responsibility to decide whether it is more important to fix the bugs or deliver new features. The issue with Scrum is that pushes you towards new features as the team has committed to delivering a certain number of story points, no doubt expectations have been set outside the team that things will be delivered and it's difficult to turn around to stakeholders and say "we didn't deliver because there were more important bugs".

This issue is stakeholder management, the most difficult part of managing any project. The product owner and/or project manager have to be very careful about what is communicated inside and outside the team. While it's always important for the team to understand deadlines and the real world impact of the project the most important thing is that they're focused on delivering the current sprint (be it bugs or features) as best they can. If you are building "the right thing" and the team is doing it as quickly then there is nothing more they can do.

Externally it is a different story, delivery dates and scope are a constant and messy negotiation. Make sure that expectations are set correctly, never plan for 100% utilisation of the team. Most Scrum projects happen on shifting sand, when the project begins support time is low and the team will be 90% focused on delivering features but over the course the project that number will slide.

If you ask a Scrum proponent about technical debt they will often say that the team decides how much to bring into sprint and that if the team wants to tackle technical debt it's up to them and not the product owner. While this is true, it takes very confident developers to say to their product owner that they are going to take on less story points this week in order to tackle technical debt. Quite often it just doesn't happen.

In an effort to only focus on what is immediately important Scrum encourages short term thinking. Depending on what you are working on this may be a small problem or a big problem.

```Scrum story points are supposedly meaningless, yet they are tracked, recorded and presented at many levels in an organization and often are the only metric by which a team’s performance is represented (ie. Velocity)```

This is another common issue. The requirement here is visibility. The product owner needs to have visibility of how the team is doing so they can plan accordingly and manage stakeholder expectations. The problem is that even though the Scrum process tries mask estimations by using "meaningless" points, ultimately the points get projected forward into features delivered over time.

The theory is that even though humans are very bad at estimating but if we collect enough data over time then the aggregate is an accurate yard stick. The problems with this are threefold:

- the project needs at least three of four sprints before data comes in (8 weeks) and estimates are often needed much sooner
- the project changes over time, the team changes over time and support requirements change over time invalidating previous data
- at best it only gives medium term predictions, predicting the outcome of a single sprint is not accurate and long term predictions are not possible unless the team discusses every item in the backlog

While the Scrum burndown and velocity measurements are very flawed I've never seen a better way. The key is to highlight their deficiencies or even hide their usage to external stakeholders.

```Scrum is designed to manage the weakest Engineers and consequently dis-empowers the better ones```

This is a mindset issue. I accept the argument that not every developer is equal to every other developer but that does not mean that "good" developers are dis-empowered by "worse" developers. Understand that every team member has different strengths and though it may not possible to form the absolute best team with the hand you are dealt, you have to form the best team you can.

A fantastic example of this is in season 1 of [The Wire](https://en.wikipedia.org/wiki/The_Wire) where a band of misfit detectives come together in a dysfunctional organisation on a project no one wants to succeed and somehow make a success of it. The key is having everyone play to their strengths. Sometimes the "best" coders are not the most thorough and sometimes the "junior" developers are more keen to put the hours into to more meticulous work. Equally, assigning a crucial part of the system infrastructure to a junior developer may not be the best idea. Pay close attention to the interactions with in the team and subtly try to guide people towards areas they will help the team the most. Ultimately everyone will be happy if they feel they are contributing to the team. Remind people of the [ten commandments of egoless programming](https://blog.codinghorror.com/the-ten-commandments-of-egoless-programming/).

```Scrum is the opposite of many of the points set forth in the original agile manifesto```

Only if the team makes it so. Always value the principles of Agile over any software development process:

```Individuals and interactions over processes and tools
Working software over comprehensive documentation
Customer collaboration over contract negotiation
Responding to change over following a plan
```

The most critical thing for anyone implementing a software development process is **context**. Is Scrum right for this project, this team, this company?

The only levers you have to control a project are scope, resource, time and quality. Assuming quality is non-negotiable then do you control the scope, resource or time? There are always constraints so controlling all three is unlikely, but to implement Scrum (or any software process) you have to control at least one.

All too often external clients set the scope and deadline so the only thing you are left with is resource. Resource is a tricky one, adding more to a project doesn't have an immediate impact but it does have an immediate cost. Is your company willing to absorb that cost? This often leads to "Agile" companies spending a large amount of time haggling with the client over scope during contract negotiations, rejecting any form of change during the project... and we're back to waterfall.

You can't just implement Scrum during the development phase of waterfall and call it Agile. A key tenant of Agile and Scrum is shrinking the whole process: planning, development, testing and release into to a tight loop to get feedback quickly.

Remember that Scrum is best suited to business projects where features have a value to end users. It is not well suited to other types of work, if you are doing more support than features then maybe try Kanban or a more operational led process.

Whatever you do, the key thing any software development process should provide is visibility.
