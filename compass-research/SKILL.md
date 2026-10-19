---
name: compass-research
description: Read the codebase to build deep understanding for design and debugging. Use when you need to understand architecture, conventions, and patterns across a codebase area. For targeted fact-finding (bug repro, code search, prior art), use `compass-investigate` instead.
---

# Compass Research

Read through the code in the scope given by the user and examine it. 
You are looking for facts that can be used to improve your understanding of the existing codebase when debugging issues and designing features. 
These facts must be committed to memory using the repository's preferred memory store.
Use the __Relevant Data__ section to establish what should be investigated

## Scope

The user will provide the scope of the query as part of the instruction. Request the scope if it is not provided. A wide scope should be split up and delegated.

## Relevant Data

- Architecture Choices - Seams and patterns that inform how a module is designed to be extended.
- Architectural Deviation - Parts of the codebase that stand out for not adhering to the architecture choices facts.
- Conventions - Aspects of the codebase that share similar responsibilities but are unrelated excepted by a naming pattern. For example UserDao, AdminDao, InventoryDao.
- Hacks - Code that looks like it was patched in to solve a problem without consideration for the wider architecture.
- Gotchas - Facts that do not appear immediately obvious. For example code that relies on poorly documented aspects of libraries.
- Duplicate functionality - Code that does the same thing existing in multiple locations. 
- Shared Functionality - Reused bodies of code that could be externalized as libraries if required.
- Terminology - Established naming conventions and terminology that is specific to the application.

## Validation

Cross reference each potential fact and it's category before committing. It must standup to examination across the codebase and against existing facts. Existing facts must not be duplicated but they can be expanded. 
Once a fact is considered valid it can be committed to the project's agent information stores.

## Format 

Facts must be backed by examples. They must include a useful summary and also a decision rationale if possible. 

## Output

All new facts must be summarised for the user. The user can reject or correct facts. The user's input is authoritative unless you can categorically demonstrate the user is wrong.


## Boundary with compass-investigate

- **compass-research**: broad, open-ended codebase understanding for design decisions. "How does this codebase work?" "What are the architectural patterns?" Scope is a module, subsystem, or pain point. Output goes to persistent memory.
- **compass-investigate**: targeted fact-finding for a specific question. "Is X broken?" "How does A call B?" "Has Y been tried?" Scope is a precise investigation requirement. Output is an investigation record with recommended next skill.
- If the goal is "understand this area to design a feature," use research. If the goal is "answer this question to unblock a decision," use investigate.





