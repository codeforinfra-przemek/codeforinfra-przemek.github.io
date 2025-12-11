---
layout: default
title: "Ansible desired state – controller configuration as code"
permalink: /ansible-desired-state.html
---

# DevOps Foundations: Core Concepts and Fundamentals

## Lean Software Development

Lean software development is an agile methodology that focuses on delivering customer value by eliminating waste, increasing efficiency, and developing high-quality software. Its core principles include eliminating waste, building quality in, creating knowledge, deferring commitment, delivering fast, respecting people, and optimizing the whole. The approach originated from lean manufacturing principles, like the Toyota Production System. 
- just in time
- Jidoka
- Andon - anyone can stop prod to fix proces

<img width="1416" height="748" alt="image" src="https://github.com/user-attachments/assets/cab51012-1670-4f43-a267-6fab18bc45f1" /> 
---

Core principles
- Eliminate waste: Remove any activity that does not add value for the customer.
- Build quality in: Make quality a continuous process to prevent defects, rather than trying to fix them later.
- Create knowledge: Learn and share knowledge throughout the development process through activities like testing, feedback, and collaboration.
- Defer commitment: Make decisions as late as possible based on facts and customer needs, rather than making them too early and having to change them later.
- Deliver fast: Accelerate the delivery of software to get feedback and value to the customer sooner.
- Respect people: Empower the team, encourage communication, and involve everyone in solving problems.
- Optimize the whole: Look at the entire development process to ensure all parts work together seamlessly, rather than focusing on optimizing individual pieces in isolation. 

<img width="640" height="426" alt="image" src="https://github.com/user-attachments/assets/916e55dd-08be-419f-9035-872a0817d9f7" /> 
---

The seven wastes in Lean Software Development, adapted from manufacturing, are Partially Done Work, Extra Features, Relearning, Handoffs, Delays, Task Switching, and Defects, all representing activities that consume resources but don't add customer value, with goals to streamline processes, enhance quality, and deliver faster by eliminating these inefficiencies. 
Here's a breakdown of each waste:
- Partially Done Work (Inventory): Unfinished features or code that isn't deliverable, tying up resources and adding complexity without value.
- Extra Features (Overproduction): Building features users don't need or use, increasing complexity and maintenance costs.
- Relearning (Extra Processing): Wasted time relearning requirements, designs, or code because of poor information flow or documentation.
- Handoffs (Transportation): Passing work between teams or individuals, leading to miscommunication, delays, and lost information.
- Delays (Waiting): Idle time waiting for input, approvals, feedback, or other blocked activities, causing loss of momentum.
- Task Switching (Motion): Interruptions or context-switching between multiple tasks, reducing focus and effectiveness.
- Defects (Defects): Bugs and errors that require rework, testing, and debugging, consuming significant time and resources. 


Add:
- multitasking is not working - you loose 40% of effectivnes on multitasking.
- Paretto 80/20.
- If it hurt, you must do it.

## Understanding What DevOps Replaces

- Build:
  - force better version control model
  - poka-yoke
  - decision alone cures serveral problems

## Demo:
- repo: https://github.com/NetCoreTemplates/empty  (random dotnet repo)
- build using devops azure
