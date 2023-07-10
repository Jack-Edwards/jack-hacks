---
title: Approaching open-source contributions
description: Understanding the two-way road that is accepting open-source contributions.
date: 2023-07-09
layout: post
---

The way I approach open-source contributions changed a couple years ago.

At the time, Crypter was already open-source and I had already accepted a couple pull requests from people I never met
before. I thought this was how open-source was supposed to work; people who know what they are doing would submit
pull requests and all I needed to do was review those pull requests for completeness, defects, and style.

### What happened

One day, someone reached out to me and said they wanted to add a new feature to Crypter. The feature was a good
idea, so I added an issue to the issue tracker and assigned the individual to the issue.

This began a conversation that lasted several weeks. This person needed quite a bit of direction in order to make
progress on the feature. Add this file here, modify that code over there, use this other pattern instead, etc. It
felt as though this person was a novice and I was put in the position of providing guided learning to get this
feature implemented.  I began to feel the feature would not be worth all this time and effort.

After going back-and-forth with this individual on a specific part of the implementation, I decided to write my own
implementation and shared it with them. I thought it might help to show how I would approach the current problem.

This is where I messed up.

Inevitably, it was my code that was used to implement the feature.

Inevitably, I killed this persons drive to work on the project any further.

This person came right out and told me what I had done. I had provided a solution to the problem they were
committed to solving. This was not a feature I needed. Nothing was broken. This person had their own idea for a feature
and wanted to prove to themselves they could implement it. I took that opportunity away from them because I was
impatient and too narrow-minded on the code.

### Lesson learned

It is important to understand open-source software does not simply spawn into existence. Software requires people to
build and maintain it. Without the hard work and dedication of the few people who maintain them, software projects
would otherwise wither and die.

Open-source maintainers have as much responsibility to the project as they do to the people involved.
A project without people is dead. A project where the maintainer fails to build up its people may as well be dead.
Those people are going to find other, more welcoming projects to work on. Those people may even begin to compete with
the original project. The project with a better developer experience is likely going to succeed over the alternative.

### Moving forward

I am now taking a much different approach when interacting with others who want to contribute to one of my projects.

If I tag an issue as a "*good first issue*" in GitHub, I make sure to provide ample context and information on that
issue, without getting into implementation details. People do not want to be bound by a pre-determined implementation,
nor is the pre-determined implementation always the best one. Implementation should be decided on the person doing
the work in the moment, provided they have adequate context to make good decisions.

In addition to the standard issue details, I also include a statement on who this issue may be a good fit for. For
example, "*this may be a good first issue for someone interested in ASP.NET development.*" Someone looking to
contribute to a project can very quickly determine whether this is an issue they want to work on. This is not meant
to act as a barrier or a requirement. I understand many people look at the "*good first issue*" tag as a way to
learn something new. If someone is not comfortable with ASP.NET, but is eager to learn, then one of these issues may
actually be a perfect fit.

Once someone does show interest in an issue, I do my best to make sure they have the information they need,
without getting in their way. I do not provide them with unsolicited code.

Providing your own code defeats the purpose of adding the issue in the first place. The reason people add issues to
their own projects is to seek help from the community. Not because the person is incapable, but because the person
wants their project to grow beyond themselves.  That is real beauty of writing open-source software.
