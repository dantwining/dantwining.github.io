---
layout: page
title: Optimising Work Types - A Worked Example
---

## Context
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

## Current scenario
Imagine having multiple software delivery teams working on business initiatives. These business-led changes can be prioritised by the teams; they have control over the backlog of work and can control the point at which they tackle each stage of the development. As such, the work to deliver these business-led initiatives is Type 1 work. However, as part of the delivery of this work, the teams require various infrastructure-related actions to be performed.

*This scenario uses the creation of github repos as a real-world example of infrastructure work required by software delivery teams; this is something that all developers and infrastructure teams should be familiar with. For anyone not familiar with github repos, it is sufficient to know that they are something that infrastructure controls and the dev teams need, and that the following all makes an equal amount of sense if you substitute “github repos” for “jam sandwiches” in all of the following text.*

Imagine we also have an infrastructure team responsible for provisioning github repositories for the company. This capability is centralised because:
- There is a desire for consistency in the creation and naming of our github repositories.
- Security requires oversight of our repositories, as they contain company IP, and access for creating/deleting repositories needs to be restricted.
- Historically, the total number of github repositories has had a cost/licencing implication, and so needs to be controlled.
- Like many infrastructure tasks, github administration requires a moderate amount of specialist knowledge that isn’t usually needed for software development, and so isn’t an existing skill within the dev teams.

In this scenario, software delivery teams are able to plan their business-led changes as Type 1 work; they can choose what to prioritise ahead of or behind the initiative, and they can control the point at which new github repos are required and so requested from the infrastructure team.

The infrastructure team, on the other hand, have no direct influence on the planning and execution of the delivery team, and so requests for their support in creating new github repos appear as Type 3 work; unavoidable routine work that just has to be done as and when requests appear.

### Conflict
With the work flowing between the teams in this manner, conflict occurs as the Type 3 work of provisioning the github repos by the infrastructure team blocks the Type 1 work of the teams looking to build new services. With every new github repo requiring a human from the infrastructure team to be on hand to craft and apply changes, repo creation often gets delayed by other, more urgent work.

## Strategies for change
### Strategy 1: Embedded Infrastructure
One possible solution to the problem is to embed infrastructure experts in each of the teams. By embracing a “cross functional team” mentality, teams are built with infrastructure experts in them, and so any work that would ordinarily be passed off to the infrastructure team as Type 3 routine maintenance becomes Type 1 business activity work for the infrastructure specialist in that team.

On paper, this can be very appealing, especially as it taps into the “cross-functional teams” narrative that is very popular amongst Agile development evangelists. Also, it appears to reduce our Type 3 work in favour of maximising our Type 1 work, which on the face of it is exactly what the Phoenix Project is telling us to do.

However, having lived this movie, one soon finds that there are a number of significant drawbacks:

- This approach doesn’t scale well. Your number of infrastructure experts now has to scale in line with the number of development teams in your organisation, despite the fact that the amount of specialist work to be done (provisioning github repos, in this instance) may well not scale in the same way.
- With the infrastructure team disbanded, there is a dilution of expertise across the business. Not all infrastructure specialists are equal, and you may well find that your networking specialist is now part of a delivery team that doesn’t particularly need networking problems solved, and your persistent storage expert is now having to do battle with ingress controllers and availability zones. With each infrastructure specialist now part of a different team, knowledge sharing and upskilling becomes much harder, and infrastructure specialists can quickly become isolated and disillusioned.
- There is a lack of consistency in the work that each infrastructure specialist does, as different specialists in different teams redo the same work as each other in isolation. In our example, with each infrastructure specialist having their own credentials to create github repos, when doing so, each specialist will do it “their way”. Any cross-cutting concerns, such as security requirements, naming conventions, and so on, become much harder to enforce.
- There is a significant amount of hidden repetition with this approach. Whilst it can look like we’re getting lots of Type 1 work done, what we’re actually doing is spending lots of time doing repetitive tasks under the guise of Type 1 work. The goal isn’t to maximise the amount of time we spend on Type 1 work, but to maximise the throughput of Type 1 work across the business; repeatedly provisioning git repos under the guise of delivering new services only slows our delivery of valuable work.

### Strategy 2: High automation and Self-Service

An alternative approach to the problem is to embrace a culture of automation, turning all of those manual tasks into automated workflows, and then surfacing that automation to development teams as self-service capabilities. The infrastructure team's goal should be to automate themselves out of all Type 3 work, to the point where in an ideal world every request from another team for unavoidable routine work or support is done by machines.

In the case of the github repo example, building a tool that enables developers to provision github repos on demand, whilst at the same time codifying all of the skills, knowledge, requirements and experience into the tool, both empowers development teams and eliminates manual intervention from the infrastructure team.

All of the capabilities that were centralised in the team are now centralised in the tool:
- Consistency in the creation and naming of our github repositories: Whatever rules the infrastructure team were following manually to validate requested repo names can be applied automatically. The tool can correctly format repo names, enforce any standard prefixes or suffixes, and could even provide an interactive way to produce compliant repo names.
- Security requirements: Only the tool has permissions to create or delete repositories; nothing can be done outside of the tool, and only the infrastructure team has the ability to change the tool's inner workings. Security standards can be enforced, such as ensuring that repos are private, or ensuring that the config for the repo is set to the company standard.
- Costs and licencing: Any commercial implications can be tracked, audited and controlled. If there's a hard limit on the number of github repos we can support, then the tool can enforce it. If we're just looking to manage costs, we can track repo creation, associate repos created with the users that created them, and even rate-limit teams that are at risk of abusing the tool.
- Knowledge gap: All of the specialist knowledge required for github administration is handled by the tool. Developers don't need any special skills to interact with the tool, and don't need to understand in full detail howit work, they just need to know how to run it.

Now, instead of the infrastructure team being interrupted by the ad-hoc needs of the developers to create new repos, it's the repo-generating script that does all the work. Development teams don't need to worry about delays in provisioning repos, and the infrastructure team can use the time that they would have spent manually applying changes automating the next most important Type 3 workflow, or even spend some time on their own Type 1 and 2 projects.

Which is not to say that such a shift to automation and seflf-service is easy, nor can it happen overnight. It requires discipline and support; teams drowning in labour-intensive Type 3 work are often also overwhelmed with fighting Type 4 fires and find themselves trapped in a cycle of never getting the breathing room required to make meaningful inroads into automating their workload. Senior management needs to accept that, especially at the start, automating tasks is going to take a lot longer than just doing the task once, and they need to buy in to the benefits. But the benefits are significant; not only will you see a much more engaged infrastructure team as they're able to automate the mundate and focus on the valuable, but empowered developer teams something something something.

So, yeah, in short, do this one.

## The state of the art

Much has changed in the years since I published the first version of this article. The idea of "platform teams", "platform as a product" and entire conferences about "developer experience" and platform teams acting at a bridge between infrastructure and developers. Spearheaded by backstage.io, a whole raft of tools have emerged to act as a focal point for delivering this sort of approach strategy culture.

## References
Whilst this is informed by real-world experience, much of that experience overlaps with [the thinking expressed in Daniel Jones’s keynote on Anthropic Sympathy](https://www.youtube.com/watch?v=QWMUYl0BkEI), which I encourage you all to watch.  
