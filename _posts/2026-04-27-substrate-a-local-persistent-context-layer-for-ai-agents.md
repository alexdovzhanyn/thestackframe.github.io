---
layout: post
title: Introducing Substrate, a local persistent context layer for AI agents
categories: [AI,Agents,Substrate]
---

Over the last year or two, AI coding tools have become good enough that it no longer makes sense to dismiss them outright. They can explain unfamiliar code, 
generate boilerplate, scaffold new features, write tests, and often get you 70% of the way through tasks that would have previously required uninterrupted focus. 
Even when they are wrong, they are frequently wrong in useful ways. They can give you momentum. At the same time, after using them regularly, I kept running 
into a much less impressive limitation. They are surprisingly bad at remembering things that are already known.

Not "known" in the universal sense, but "known" in the local sense. Known by me, known by my machine, known by the environment they are operating inside. They do not 
know which repository is active and which one is an abandoned prototype with a similar name. They do not know that one project uses pnpm, another uses cargo, 
and a third only starts correctly if Redis is already running. They do not know that a certain directory is generated output and should be ignored, 
or that a service called “backend” is actually three separate repositories and a cron job held together by convention.

So instead, they search. They inspect files, they infer structure from partial evidence, they repeatedly spend effort learning facts that were already true 
before the session began. It feels pretty backwards.

## Intelligence is not the problem

When these tools fail, the instinct is usually to blame the model. Sometimes that is correct. Models hallucinate, misunderstand requirements, or occasionally make 
decisions with full confidence while being completely lost.

But in many day-to-day workflows, the real issue is simpler than that: the tool is starting from zero every time. If a capable engineer joined your team tomorrow, 
you would not expect them to be productive without context. You would tell them where the repositories live, how to run the stack locally, which systems are brittle, 
which conventions matter, and what kinds of changes the team tends to prefer. AI tools usually skip that layer entirely. They are handed a prompt and a filesystem, and are 
then expected to reconstruct everything else on demand. Sometimes they manage it, but often they waste time doing so.

The obvious answer is to keep notes. Many developers already do this in some form. A markdown file with useful commands. A README with setup steps. A scratchpad with 
architecture notes. A prompt template containing preferences and project context. More recently, some tools have introduced features like “rules,” where you define 
instructions for the agent to follow. In practice, these are often just better-packaged markdown files. I do not mean that dismissively. Notes are useful. READMEs are useful. 
Cursor rules are useful. I use these kinds of tools myself.

The problem is that documents are not the same thing as memory. Documents are optimized for reading top to bottom. They are good at explanation, onboarding, and capturing 
broader context. Memory systems are optimized for retrieving one relevant fact at the moment it is needed. Over time, notes accumulate stale information. Temporary reminders
sit next to durable truths. The command that used to start the app remains beside the command that starts it now. One-off observations become indistinguishable from long-term conventions.
Rules files grow into grab bags of preferences, warnings, architecture notes, and outdated assumptions.

This is manageable for a person because people bring judgment to the document. They know what can be ignored, what is outdated, and where to skim. Software generally does not. If an agent
needs to know how to start a project, I do not want it scanning prose from three markdown files, a README, and a rules file while trying to infer which instruction is current, all while filling
its context with irrelevant information, and burning tokens all the while. I want it to retrieve a clean, relevant fact.

## Enter Substrate

Substrate is a local-first memory layer for AI coding tools. The core unit in Substrate is a belief. A belief is one atomic, reusable fact that is worth remembering. It should be
self-contained, useful outside the moment it was created, and specific enough that a tool can act on it. A belief might be that the main frontend repository lives in a 
certain directory. It might be that a project should be started with `docker compose up`. It might be that generated files under a specific path should be ignored during
searches. It might be that I generally prefer smaller targeted refactors over sweeping rewrites.

Those facts are stored locally and can be queried semantically by tools that need context. Instead of re-deriving the same truths every session, they can begin with what is already known.

That is the entire idea.

## How it actually works

Substrate runs locally as a small background service. Tools communicate with it through an MCP interface, which gives coding agents a standard way to ask questions and record useful discoveries.
At a high level, an interaction looks like this:

An agent needs to find the main frontend repository. Instead of recursively searching your home directory and guessing based on folder names, it can ask
Substrate something like: “Where is the main frontend repo for project Alpha?” Substrate searches stored beliefs and returns the most relevant match. If the agent later discovers 
that the repository moved, or that a startup command has changed, it can propose a new belief. Substrate can then store it directly or flag potential conflicts with older beliefs 
so the memory stays coherent instead of accumulating duplicates and contradictions.

That means the system is not just a static note store. It is designed to participate in an ongoing workflow where tools can both consume and improve context over time.

## Local first

Most of the context that makes this valuable is deeply tied to one machine and one person. Filesystem paths are local. Installed tools are local. Shell habits are local. 
Repository layouts are local. The odd workaround required by one old internal service is almost certainly local. Shipping all of that to a remote service simply to make 
software less forgetful feels like an unnecessary detour. A local-first model is simpler. It avoids adding network dependency to something that should be immediate. 
It keeps sensitive environment details on the machine where they already exist. It also makes the tool easier to treat like infrastructure rather than a subscription.

## The hard part is selectivity

Storing information is easy. The difficult part is deciding what deserves to persist. Some facts are temporary and should expire quickly. Some are trivial to rediscover 
and not worth storing at all. Some are wrong often enough that they should be treated cautiously. Some are extremely valuable because they save friction repeatedly over long periods of time.
A useful memory layer needs to make those distinctions. If it stores everything indiscriminately, it becomes a junk drawer. If it stores too little, it becomes irrelevant. That filtering problem 
is much more interesting to me than the database itself.

Some beliefs can be checked automatically. If a stored repository path no longer exists, that is useful to know. If a startup command now fails, that matters. 
If a dependency is no longer installed, the belief should not continue to present itself as reliable truth. Most note systems silently rot. They preserve outdated information with 
perfect confidence. I would rather have a smaller memory system that can question itself than a larger one that cannot.

## Check it out
Substrate is still under active development, but the core idea is already real enough to use, iterate on, and improve in public. If this sounds interesting to you—whether as a developer 
using coding agents or as someone building tools in this space, you can check out the project on [GitHub](https://github.com/alexdovzhanyn/substrate). Read the code, open an issue, suggest ideas, 
or try it in your own workflow. I think persistent local context is going to become an increasingly important part of developer tooling, and I’d like to build Substrate in the open.
