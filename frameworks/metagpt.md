---
date: 2024-03-14
time: 13:20
author: 
title: 
created-date: 2024-03-14
tags: 
paper: 
code: https://github.com/geekan/MetaGPT
---
# Description of result
- [data interpreter](https://docs.deepwisdom.ai/main/en/guide/use_cases/agent/interpreter/intro.html) for problem solving with code
- 87.7% on MBPP
---
# How it compares to previous work
- beats chatdev on most metrics with feedback?
---
# Main strategies used to obtain results
- SOPs, human-like workflows
- agents can communicate via structured outputs (docs n diagrams) rather than chat to prevent miscom
- test cases for feedback to refine code
---
# Implementation / practical notes
## Software company
call `software_company::app` which then calls `generate_repo`
make a `Team` object which contains `Environment`. latter holds the agents which can publish msg to it and it will distribute msg to agents. `Environment` also runs all agent's steps in parallel.

- customization for multiagent: see [this](https://docs.deepwisdom.ai/main/en/guide/tutorials/multi_agent_101.html)
- implement [memory](https://docs.deepwisdom.ai/main/en/guide/tutorials/use_memories.html)
- [human](https://docs.deepwisdom.ai/main/en/guide/tutorials/human_engagement.html) in the loop
	- using plan_and_act instead of default react allows `auto_run` to be set to false
- serialization n breakpoint [recovery](https://docs.deepwisdom.ai/main/en/guide/in_depth_guides/breakpoint_recovery.html)
- mermaid packages, if not installed, still gives .mmd
- `implement` or `code_review` flags uses the Engineer role, which writes code (but doesnt execute)
- `run_tests` flag uses the Qa_Engineer role which has the `run_code` action which is the only code execution module
- `inc` flag for incremental mode, i.e. for existing repo! also see [example](https://docs.deepwisdom.ai/main/en/guide/in_depth_guides/incremental_development.html)
## [agent](https://docs.deepwisdom.ai/main/en/guide/tutorials/agent_101.html)

- each agent is in `metagpt.roles`, which inherits from `metagpt.roles.role` and is called by `.run()`. in the base `Role` class
- each role has actions from the `actions` folder and `watch` (which causes agent to watch out for specific actions.)

> [!warning]
> 
> warning: `PrepareDocuments` action has a line
> ```python
> if path.exists() and not self.config.inc:  
>     shutil.rmtree(path)
> ```
> use a copy of ur code incase u forgot to on the inc flag!

### [Researcher](https://docs.deepwisdom.ai/main/en/guide/use_cases/agent/researcher.html)
- final report is given in 1 prompt so limited by tokens, whereas each qn/ subtopic is its own document
- search api: serpapi, google, serper n ddg  
- webbrowser: playwright or selenium
- CollectLinks action takes either topic or sys prompt, but latter not accessible from researcher role. need to customize an agent to use it