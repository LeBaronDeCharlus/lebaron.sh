---
title: "Rethinking IT: The Quest for Efficiency Over Complexity"
date: 2025-03-11
tags: ["career", "devops"]
image: "/images/rethinking-it/01-cover.jpg"
draft: false
---

### Context

It took me a long time to finally sit down and write this article. I’ve always been passionate about technology. It’s in my DNA. For over 10 years, I’ve worked in IT engineering, taking on roles such as Full-Stack Developer, System Administrator, DevOps Engineer, and Site Reliability Engineer. My path has been unconventional: entirely self-taught, climbing the ranks one step at a time.

I don’t have a university degree beyond a high school diploma. Everything I know, I taught myself: books, online resources, YouTube, and hands-on experience.

I’m publishing this article today to highlight a **major shift happening in our industry**. It isn’t meant to criticize our field, but rather to shed light on trends that are becoming increasingly widespread.

## The Challenges of Rigid Methodologies in IT

Over the years, **countless** practices have been introduced to improve collaboration, efficiency, and robustness in our work. Many of them are well-known: Agility, Scrum, Kanban, DevOps, Lean, Waterfall, PRINCE2, Accelerate, and more.

While these methodologies are designed with **good intentions**, applying them too **rigidly** can create problems.

First, not every company operates at the scale of Google, Amazon, or Microsoft. Enforcing methodologies designed for teams of hundreds on a small company with a couple dozen engineers isn’t always **optimal**. Excessive time is often lost in meetings (also known as “rituals”), and in some cases these practices don’t even align with the company’s **actual goals**. Trends come and go, and we’ve seen Scrum implemented everywhere, sometimes at the expense of true agility.

On top of that, many of these methodologies come from books that few people actually take the time to read and understand. Instead, people tend to blindly copy what others are doing, which breeds widespread misconceptions: for example, we’ve all heard the mistaken claim that “*DevOps is a job title.*”

I’m not saying these methodologies are bad or incompatible with business needs. What I find problematic is their rigid, one-size-fits-all application, often without real adaptation to a company’s specific needs or any deep understanding of their true value.

## State of the Art: Too Complex for Our Needs

A broad topic, isn’t it? Take, for example, the never-ending debate between monoliths and microservices.

Too often, before a single line of code is even written, architecture decisions become **over-engineered** and unnecessarily complex, ultimately leading to a system that far exceeds the company’s technical, financial, and human capabilities.

As engineers, we’re passionate about our craft. We appreciate elegant code, highly optimized infrastructure, and cutting-edge solutions, and **that’s a good thing**. But the reality of day-to-day work is often quite different: our actual working conditions don’t always reflect what gets presented in books, talks, or conferences, and we tend to forget that.

Sometimes, building a monolith in just a few days is **far more efficient** than spending weeks developing five microservices, Terraforming them, wiring them into a Kubernetes cluster (also Terraform-managed), configuring policies, and tuning autoscaling. Nothing stops us from gradually refactoring the monolith into a more modular system later, if needed. But in the early stages, the cost in infrastructure and engineering hours often outweighs the real objective: getting a functional service online.

That’s exactly where unnecessary debates, frustration, and arguments creep in, which brings me to my next point.

## Losing Sight of the Real Objective

What is our core objective as an IT company? Delivering our product, right? Without that, we can’t be profitable.

Now, let’s pose this question to someone completely outside the tech world:

*Would you rather spend twice the time and twice the money to build a product, just to see if it might be profitable? Or would you prefer to launch as quickly as possible, validate its profitability, and then refine it?*

What do you think their answer would be?

Obviously, the goal is to become profitable as quickly as possible: otherwise, we’re heading straight for disaster. Infrastructure is expensive, and a Kubernetes cluster costs exponentially more than a simple VM, yet both can run our product just as well.

Our profession (and our passion) often lead us to make technically brilliant decisions that don’t always align with our company’s actual needs. It’s up to us to work smarter and make informed choices that balance innovation with business reality.

## Over-Processing: When Processes Get in the Way

To wrap things up, I want to touch on a topic that I’m sure many of you are familiar with: **over-processing**.

Sometimes, what should be a simple task ends up requiring five extra steps, because we’ve created a process for it somewhere along the way. These processes often serve a real purpose (security, compliance, consistency), but once established, they tend to become inflexible.

Process creation is valuable, but it comes with two major drawbacks:

- As mentioned earlier, processes are often rigid, making it difficult to work efficiently in specific cases.
- More processes lead to a new question: **what happens when there’s no process for a particular task?**

See the vicious cycle? A request that falls outside any existing process takes far too long to handle, simply because **no process exists for it**, and the ability to just solve the problem on the spot disappears.

On the other hand, if a process exists but doesn’t actually fit the situation, the amount of wasted time (both human and technical) is excessive.

To make things worse, we frequently create **human single points of failure (SPOFs)**. If “Mr. X” is the only person authorized to validate a process and he’s out for a week, the entire workflow grinds to a halt. That’s a full week lost.

### Ask Yourself: What If You Knew Nothing About IT?

To wrap up, here’s a simple way to approach these challenges.

For every decision we make, imagine putting the question to someone completely outside our field, then compare their answer to our own. More often than not, the simplest, most logical solution turns out to be the best one, especially in the early stages.

There will always be time later to refine, optimize, and add real expert value by making our solutions more robust. But first, we need to focus on what actually matters.

Thanks for reading!
