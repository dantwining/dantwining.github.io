---
layout: page
title: Optimising Work Types - A Worked Example
---

# Context
We want to optimise the delivery of valuable work and minimise our unproductive labours. To do so, we categorise our work, allowing us to first measure, and then control, the work we do in our team.

As explored in The Phoenix Project, work can be categorised as:

| Type | Description |
| --- | --- |
| Type 1 | Business Projects: Work from the business |
| Type 2 | Internal IT Projects: Work from IT |
| Type 3 | Changes: Unavoidable routine work and support |
| Type 4 | Unplanned Work: Unplanned or recovery work |

The goal is to maximise the amount of Type 1 and 2 work done, especially Type 1, so that the teams are delivering valuable work and not spending time attending to Type 3 and Type 4 issues.

In the rest of this article, I explore how one team’s Type 3 work can be driven by another team’s Type 1 / 2 work, and possible strategies for mitigating the impact of this undesirable Type 3 work.

# Current scenario
Imagine having multiple software delivery teams working on business initiatives. These business-led changes can be prioritised by the teams; they have control over the backlog of work and can control the point at which they tackle each stage of the development. As such, the work to deliver these business-led initiatives is Type 1 work. However, as part of the delivery of this work, the teams require various infrastructure-related actions to be performed.

*This scenario uses the creation of github repos as a real-world example of infrastructure work required by software delivery teams; this is something that all developers and infrastructure should be familiar with. For anyone not familiar with github repos, it is sufficient to know that they are something that infrastructure controls and the dev teams need, and that the following all makes an equal amount of sense if you substitute “github repos” for “jam sandwiches” in all of the following text.*

Imagine we also have an Infrastructure team responsible for provisioning github repositories for the team. This capability is centralised because:
- Historically, the total number of github repositories has a cost/licencing implication, and so needs to be controlled
- There is a desire for consistency in the creation and naming of our github repositories.
- Security requires oversight of our repositories, as they contain company IP, and access for creating/deleting repositories needs to be restricted.
- Like many infrastructure tasks, github administration requires a moderate amount of specialist knowledge that isn’t usually needed for software development, and so isn’t an existing skill within the dev teams.

In this scenario, software delivery teams are able to plan their business-led changes as Type 1 work; they can choose what to prioritise ahead or behind the initiative, and so have control over the point at which new github repos are required and so requested from the infrastructure team.

The infrastructure team, on the other hand, have no direct influence on the planning and execution of the delivery team, and so requests for their support in creating new github repos appear as Type 3 work; unavoidable routine work that just has to be done as and when requests appear.

## Conflict
With the work flowing between the teams in this manner, conflict occurs as the Type 3 work of provisioning the github repos by the infrastructure team blocks the Type 1 work of the teams looking to build new services. With every new github repo requiring a human from the infrastructure team to be on hand to craft and apply changes, repo creation often gets delayed by other, more urgent work.

# Strategies for change
## Strategy 1: Embedded Infrastructure
One possible solution to the problem is to embed infrastructure experts in each of the teams. By embracing a “cross functional team” mentality, teams are built with infrastructure experts in them, and so any work that would ordinarily be passed off to the Infrastructure team as Type 3 routine maintenance becomes Type 1 business activity work for the infrastructure specialist in that team.

On paper, this can be very appealing, especially as it taps into the “cross-functional teams” narrative that is very popular amongst Agile development evangelists, and it appears to reduce our Type 3 work in favour of maximising our Type 1 work, which on the face of it is exactly what the Phoenix Project is telling us to do.

However, having lived this movie, one soon finds that there are a number of significant drawbacks:

- This approach doesn’t scale well. Your number of infrastructure experts now has to scale in line with the number of development teams in your organisation, despite the fact that the amount of specialist work to be done (provisioning github repos, in this instance) may well not scale in the same way.
- With the infrastructure team disbanded, there is a dilution of expertise across the business. Not all infrastructure specialists are equal, and you may well find that your networking specialist is now part of a delivery team that doesn’t particularly need networking problems solved, and your persistent storage expert is now having to do battle with ingress controllers and availability zones. With each infrastructure specialist now part of a different team, knowledge sharing and upskilling becomes much harder, and infrastructure specialists can quickly become isolated and disillusioned.
- There is a lack of consistency in the work that each infrastructure specialist does, as different specialists in different teams redo the same work as each other in isolation. In our example, with each infrastructure specialist having their own credentials to create github repos, when doing so, each specialist will do it “their way”. Any cross-cutting concerns, such as security requirements, naming conventions, and so on, become much harder to enforce.
- There is a significant amount of hidden repetition with this approach. Whilst it can look like we’re getting lots of Type 1 work done with this approach, what we’re actually doing is spending lots of time doing repetitive tasks under the guise of Type 1 work. The goal isn’t to maximise the amount of time we spend on Type 1 work, but to maximise the throughput of Type 1 work across the business; repeatedly provisioning git repos under the guise of delivering new services only slows our delivery of Type 1 work.

## Strategy 2: High automation and Self Service

Do this one.

# References
Whilst this is informed by real-world experience, much of that experience overlaps with [the thinking expressed in Daniel Jones’s keynote on Anthropic Sympathy](https://www.youtube.com/watch?v=QWMUYl0BkEI), which I encourage you all to watch.  