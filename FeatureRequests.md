# How to write better feature requests with functional requirements
You think a feature is missing which is essential for the business to run correctly? Or do you have an idea for a feature that would improve the way we work? Great! You may feel that the natural next step is to ask a developer to build this for you. However, you probably shouldn't do this just yet. There are probably some things you need to figure out before that, so that you can better inform the developer, enabling her to produce a higher quality result in less time.

## What are functional requirements
As the name suggest, functional requirements describe *how* something should work.

From Wikipedia:
> a functional requirement defines a function of a system or its component. A function is described as a set of inputs, the behavior, and outputs.

In order to help the product engineers do their job better, they need to know not only *how* something should work but also *how the new feature will be used*, by whom and *why*. A good functional specification should contain answers to as many of the following questions as possible:

- Who needs this feature and why?
- Under what conditions this will be used? 
- What is expected to be achieved by this feature?
- What is not expected to be part of the implementation?
- Are there any expected side-effects, and what to do about them?
- What might come as future requirements?

In other words: Context.

Developers are not code-spitting machines but humans that can think and usually want be prepared for future scenarios. The only way they can generate a scalable code is if they understand the context. A developer that truly understand the context is able to fill in the logical gaps in your specification, saving you from submitting bug reports and unnecessary iterations. Contrary to what many believe, writing code which considers future scenarios doesn't really take more time and usually results in more organized and clean code. Make sure your developers truly understand how your features forms a part of a whole workflow in somebody's day.

### Except for context, what else should be contained in functional requirements?

- Details: Give details, don't be cheap on details and also don't be afraid to write too many details, people don't have to read it if they don't want to. Your developers will appreciate the details.
- Open issues: Things you couldn't sort out yet. Things that don't make sense or details that you didn't have time yet to nail down. Maybe you are waiting for feedback from user testing or sales department? Write this down and make your team aware of it. They might prefer to wait for these issues to get solved before starting implementation. Please, try to respect that.
- Side notes: Your specs have various audiences, from business to tech. For example, you could flag a paragraph as "Technical Notes" to allow non-techs to skip them. Joel Spolsky mentions using "Testing Notes", "Marketing Notes" and "Documentation Notes" as well.

## Why should you invest time on writing clear specs?
One immediate drawback for writing specifications is that it takes more of your time initially. You don't feel this instant gratification of getting things done by quickly writing an idea down and passing it to a developer. In the following lines I will try to convince you that this is a time wisely invested.

### Because it saves time
Giving precise information including background will help the developer to come up with a better structure for her code and having a well structured code helps developers to work faster and making changes as spec change. Yes, it's okay to change your spec a bit and it happens all the time.

### Because it makes you realize what you need
When you are forced to sit down and formulate what is it that you need, you will start to use your imagination and construct the feature in your mind. This will give you a better picture of what you really need and most importantly, of what you don't need.

### Because a developer might send you back to write them, before she starts implementing
This is not a bureaucracy nor an ego-game. This is your developers trying to do their work good. Believe me you don't want to have a feature that was implemented by developers that didn't understand what they were working on. The world is full of bad software and that's usually due to lack of context. Remember? It's all about context.

## Inspiration and further reading
This article is inspired by Joel Spolky's series of articles about functional specifications. Please take some time and read it, as it's very nicely written and an easy read.

[Functional specifications why bother - Part 1](https://www.joelonsoftware.com/2000/10/02/painless-functional-specifications-part-1-why-bother/)
[What's a spec - Part 2](https://www.joelonsoftware.com/2000/10/03/painless-functional-specifications-part-2-whats-a-spec/)
[Sample Spec](http://global.joelonsoftware.com/English/PainlessSpecs/WhatTimeIsIt.html)