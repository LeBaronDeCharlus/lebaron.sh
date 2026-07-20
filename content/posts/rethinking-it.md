---
title: "Rethinking IT: The Quest for Efficiency Over Complexity"
date: 2025-03-11
tags: ["career", "devops"]
image: "/images/rethinking-it/01-cover.jpg"
draft: false
---

*Photo by [Jason Goodman](https://unsplash.com/@jasongoodman_youxventures) on [Unsplash](https://unsplash.com)*

### Context

It took me a long time to finally decide to write this article. I’ve always been passionate about technology. It’s in my DNA. For over 10 years, I’ve worked in the field of IT engineering, taking on roles such as Full-Stack Developer, System Administrator, DevOps Engineer, and Site Reliability Engineer. My path has been unconventional: entirely self-taught, climbing the ranks step by step.

I don’t have a university degree beyond a high school diploma. Everything I know, I taught myself: books, online resources, YouTube, and hands-on experience.

The reason I’m publishing this article today is to highlight a **major shift happening in our industry**. This isn’t meant to criticize our field but rather to shed light on emerging trends that are becoming increasingly widespread.

## The Challenges of Rigid Methodologies in IT

Over the years, **countless** practices have been introduced to improve collaboration, efficiency, and robustness in our work. Many of them are well-known: Agility, Scrum, Kanban, DevOps, Lean, Waterfall, PRINCE2, Accelerate, and more.

While these methodologies are designed with **good intentions**, applying them too **rigidly** can create problems.

First, not all companies operate at the scale of Google, Amazon, or Microsoft. Trying to enforce methodologies designed for teams of hundreds in a small company with just a couple dozen engineers isn’t always **optimal**. Excessive time is often lost in meetings (also known as “rituals”), and in some cases, these practices don’t even align with the company’s **actual goals**. Trends come and go, and we’ve seen Scrum implemented everywhere, sometimes at the expense of true agility.

Additionally, many of these methodologies are based on books, yet few take the time to read and fully understand them. Instead, people tend to blindly replicate what others are doing. This leads to widespread misconceptions: for example, we’ve all heard the mistaken claim that “*DevOps is a job title.*”

I’m not saying these methodologies are bad or incompatible with business needs. What I find problematic is their rigid, one-size-fits-all application, often without real adaptation to a company’s specific needs or a deep understanding of their true value.

## State of the Art: Too Complex for Our Needs

A broad topic, isn’t it? Take, for example, the never-ending debate between monoliths and microservices.

Too often, before a single line of code is even written, architecture decisions become **over-engineered** and unnecessarily complex, ultimately leading to a system that far exceeds the company’s technical, financial, and human capabilities.

As engineers, we’re passionate about our craft. We appreciate elegant code, highly optimized infrastructures, and cutting-edge solutions, and **that’s a good thing**. However, the reality of day-to-day work is often quite different. Our actual working conditions don’t always reflect what’s presented in books, talks, or conferences, and we tend to forget that.

Sometimes, building a monolith in just a few days is **far more efficient** than spending weeks developing five microservices, Terraforming them, wiring them into a Kubernetes cluster (also Terraform-managed), configuring policies, and tuning autoscaling. Nothing stops us from gradually refactoring the monolith into a more modular system later, if needed. But in the early stages, the cost of infrastructure and engineering hours often outweighs the real objective: getting a functional service online.

This is where unnecessary debates, frustrations, and arguments arise, leading me to my next point.

## Losing Sight of the Real Objective

What is our core objective as an IT company? Delivering our product, right? Without that, we can’t be profitable.

Now, let’s pose this question to someone completely outside the tech world:

*Would you rather spend twice the time and twice the money to build a product, just to see if it might be profitable? Or would you prefer to launch as quickly as possible, validate its profitability, and then refine it?*

What do you think their answer would be?

Obviously, the goal is to become profitable as quickly as possible: otherwise, we’re heading straight for disaster. Infrastructure is expensive, and a Kubernetes cluster costs exponentially more than a simple VM, yet both can run our product just as well.

Our profession (and our passion) often lead us to make technically brilliant decisions that don’t always align with our company’s actual needs. It’s up to us to work smarter and make informed choices that balance innovation with business reality.

## Over-Processing: When Processes Get in the Way

To wrap things up, I want to touch on a topic that I’m sure many of you are familiar with: **over-processing**.

Sometimes, what should be a simple task ends up requiring five extra steps. Why? Because we’ve created processes to follow. While these processes often serve a purpose (whether for security, compliance, or consistency), once established, they tend to become inflexible.

Process creation is valuable, but it comes with two major drawbacks:

- As mentioned earlier, processes are often rigid, making it difficult to work efficiently in specific cases.
- More processes lead to a new question: **what happens when there’s no process for a particular task?**

See the vicious cycle? A request that falls outside of an existing process takes far too long to handle simply because **no process exists for it**, and spontaneous problem-solving disappears.

On the other hand, if a process exists but doesn’t actually fit the situation, the amount of wasted time (both human and technical) is excessive.

To make things worse, we frequently create **human single points of failure (SPOFs)**. If “Mr. X” is the only person authorized to validate a process and he’s out for a week, the entire workflow grinds to a halt. That’s a full week lost.

### Ask Yourself: What If You Knew Nothing About IT?

To conclude, I’d like to offer a simple way to approach these challenges.

For every decision we make, imagine asking the question to someone completely outside our field. Then, compare their answer to our problem. More often than not, the simplest, most logical solution is the best one, especially in the early stages.

There will always be time later to refine, optimize, and bring real value as experts by making our solutions more robust. But first, we need to focus on what truly matters.

Thanks for reading!
