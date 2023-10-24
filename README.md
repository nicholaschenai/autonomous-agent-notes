# Brief notes on autonomous agents
Literature review + quick notes for my own reference so pardon the untidyness. Will occasionally copy paste directly from the papers!

Autonomous agents generate commands which programmatically finds the answer in an environment, eg web

---
# Benchmarks / Datasets

### WebArena: A Realistic Web Environment for Building Autonomous Agents
[[Code](https://github.com/web-arena-x/webarena)]
[[Site](https://webarena.dev/)]
- Web environment 
    - websites from four categories (forum, ecommerce, CMS, collaborative software platforms) that emulate their real-world equivalents
    - "tools and knowledge resources as independent websites" (map, wiki, calculator, scratchpad, documentation to dev tools)
    - helps in reproducible research -- in live environments, websites can have CAPTCHAs, evolving content and configs that make fair evaluation hard (but I think good agents should be able to handle these stumbling blocks!) 
    - Functional correctness: Allows for alternate methods that achieve the same outcome, rather than be constrained to one fixed GT action sequence
- Benchmark on translating natural language commands to web-based interactions, checking for functional correctness
    - 812 long horizon tasks
        - max number of sites involved in a task is 2
        - Generated from 190 template types ("intent", see below), of which (via manual inspection, figures might be off by a bit!) 
            - 115 are stateless/ info retrieval
            - 68 are stateful-internal (i.e. they change the state/ db but only affects personal and NPC accounts, does not interfere with other agents in a multi-agent env)
            - 7 are stateful-external (task answers are dependent on current state, which can be written by other agents in a multi-agent env) 
            - Needs deeper evaluation eg a high score may mean an agent does well on all tasks of the same template, compared to a broader agent which can score decent in many templates. Maybe using a 'mean template-average' score might be more useful   
    - user profiles (pre-populated history, various roles like user and admin)
    - Intent curation
        - 3 criteria: complexity (>2 actions), creativity (adding constraints to common tasks), deconstruction (task is broken down into templates and variables)
    - Intent categories: Info seeking, site navigation, content and config operations
    - evaluation annotations
        - for info seeking tasks, the GT text answer is provided and predictions are compared via programs like 'exact match', 'fuzzy match' and 'must include'. Recall metrics are emphasized due to nature of task
        - site navigation, content and config tasks: the intermediate states in the agent's execution phase are checked for alignment with intent, via a `locator` that retrieves the items from a location, and the `keywords` that need to exist via comparison programs above. (eg if the intent is to make a forum post about a topic, then the URL, body and content must match the specifications given in the task)
    - Includes unachievable tasks. Might be useful for catching hallucination (agent giving confident answers when there is none)
- Observation space: screenshot, HTML DOM tree, accessibility tree of active tab (can have other tabs in browser too)
- Action space: 10 actions in 3 categories (elemental operations like clicking, tab operations like opening a new tab, URL operations like going back) + 1 noop
- Baselines
    - best GPT agent (GPT4 with reasoning prompts) has success rate 10.59%, potentially due to lack of active exploration and failure recovery
        - my comment: I occasionally see some factual qns that can be answered via zero shot (eg "Which US states border Massachusetts"), might skew the metrics
    - of 41 templates, GPT4 agent only gets 100% on 1 template
    - Failure modes
        - Early stopping: GPT4 agent erroneously identifies 54.9% of feasible tasks as impossible
        - Observation bias: "GPT-4 agent often demonstrates a tendency to latch onto the first related piece of information it encounters without sufficiently verifying its relevance or accuracy" 
        - Failures in Observation Interpretation: eg not remembering past searches and repeatedly searching same term
    

### AgentBench: Evaluating LLMs as Agents
[[Code](https://github.com/THUDM/AgentBench)]
[[Site](https://llmbench.ai/)]
- Benchmark to evaluate agents across a spectrum of 8 envs
- These 8 envs are separate -- each task only involves 1 env

**New envs**
| Env Name  | Description | Contents/ Construction| Evaluation|
| -------- | ------- | --- | --- |
|OS | natural lang to shell commands. 2 tasks: QA (agent needs to output ans) and operations (eg change files. agent needs to output commands) |Instruction, docker env, checking pipeline. Optional scripts for initialization, starting, examples | evaluated via success rate (binary)|
|DB | natural lang to interact w SQL DB | each sample contains the instruction, codebook, the table itself, and the answer (text for selection type samples, hash of the correctly modified table for other entry type) | macro avg of the 3 categories (not sure which 3. initialization, interaction and checking?)|
|KG | compiled a dataset from pre-existing KBQA datasets such as GrailQA, ComplexWebQuestions, GraphQuestions. curate qns which necessitate >= 5 instances of tool invocation | input qn, entities, gold action sequence, gold ans| F1 between pred and gold, exact match between pred and gold, executability |
| Digital card game| Aquawar game | creatures are initially hidden, creatures can attack each other and player can guess the opponent's creatures. This env uses a simplified version of the game. | Completion rate, avg num of illegal action, num defeated creatures, total dmg inflicted, winrate|
| lateral thinking puzzles | Game where players ask questions to a host where the host can only reply 'yes', 'no' or 'irrelevant', in order to guess what the host has in mind |story - truth pair| Single game accuracy, round efficiency, query relevance, game progress (points for intermediate steps)|


**Recompiled envs**
| Env Name  | Description | Contents/ Construction| Evaluation|
| -------- | ------- | --- | --- |
| House-holding (ALFWorld) | Embodied agent simulator | env desc (eg agents initial position, snapshot of room containing objects and IDs), objective, feedback from env | Success rate (rate of tasks completed) |
| online shopping (WebShop) | web env with prodts scraped from amazon and attributes annotated | input: products to buy. outputs: thoughts and actions | see metric in paper, related to the similarity in attributes of the bought and GT item, and a price term |
| web browsing (Mind2Web) | domains of Travel, Information, Sevice, Shopping, and Entertainment, assembled using SimilarWeb ranking | task description, reference action sequence (element ids, symbolic action types like click, type and select options), webpage info | element accuracy, action F1, step (intermediate) and task success rate|

- Integrated toolkit with docker images for ease of use
- LLM Evaluation
    - 289 tasks in dev, 1141 in test. Multi-turn interaction requires agent to generate ~4k and 13k steps respectively to solve the tasks
    - Standardization of scores across tasks: in each task, the weight is the reciprocal of avg score of all tested LLMs. then the total score is computed by a weighted sum of scores with the above weights
    - in a sense, the scores signify the model's performance in terms of multiples of the average
    - evaluated on 25 LLMs, scores are ~ 4.41 for top closed source models (GPT4) and 1.15 for open source models (openchat-13b). Per task scores are typically in the double digits for closed models (eg GPT4 gets 17.6%-78%) and single digits/ teens for open sourced models (eg openchat-13b's per task score ranges from 0-50.2%)
- analysis
    - common error: invalid actions
    - open sourced models are limited by context size (esp when the tasks are multi-turn)
    - some models forget their roles
    

### Mind2Web: Towards a Generalist Agent for the Web
[[Code](https://github.com/OSU-NLP-Group/Mind2Web)]
[[Site](https://osu-nlp-group.github.io/Mind2Web/)]
- Contributions:
    - 2350 open-ended tasks collected from 137 websites spanning 31 domains and crowdsourced action sequences for the tasks
        - Top level Domains: Travel, Info, Service, Shopping, Entertainment
        - selected by popularity in US, 3-5 per domain
    - MINDACT: LLMs for generalist web agents
- Task curation via AMT:
    1. ask worker to propose tasks that can be performed on given website. these are screened by the authors before the next step
        - 3 criteria: Diverse, multiple rounds of interaction, described in high level detail rather than step by step instructions
        - ChatGPT to provide 50 seed tasks per website for inspiration to annotators 
    2. ask worker to perform the task. Interaction trace and snapshots are recorded via an annotation tool with Playwright
    3. Authors verify all task demonstrations
- Dataset details
    - Data split: 
        - 1k+ train
        - test: 
            - 252 cross task (tasks from same website seen during training)
            - 177 cross website (website not seen during trg, but same domain)
            - 912 cross domain (domain not seen during training)
    - each task contains
        - task description, 
        - site name, domain, subdomain
        - `actions`: list of `(Operation, Target Element)` actions to complete the task, where `operation` can be `Click` (also including `Hover`, `Press Enter`), `Type` and `Select`
        - `Webpage` Snapshots that serve as the environment
            - `raw_html` and `cleaned_html`, html before action is performed
                - In reality we dont have cleaned HTML. so agent must be able to extract visual elements
        - `pos_candidates` which are GT elements in `cleaned_html`
        - `neg_candidates` other candidate elements in page after preprocessing
    - avg 1135 elements (580 after preprocessing to keep those that are visible and carry substantial semantic meaning), 7.3 actions
    - additional data not used in task but might be helpful
        - `DOM Snapshot`
        - `Image`
        - `HAR`: network traffic for replay
        - `Trace` that contains the complete interaction trace for the annotation
        -  recordings of annotation (via playwright), session storage
- "... ChatGPT & GPT-4 to extract intents, objects and conditions from the tasks, and assign tags."
- MINDACT
    - First, a fined tuned small LM filters the top k web elements and HTML snippets from the full HTML doc/ candidate elements, given task description and previous actions
        - element reprsentation: tag, text content, salient attribute values, representation of parent and child elements
        - NCE style training: use GT and sampled negative elements
    - next, use LLM to select from filtered elements in an MCQ way to predict actions with the element, given task description and previous actions
        - MCQ: divide top k into batches of 5. If >1 option is selected after a round, new groups are formed with the selected ones. repeat till a single element is selected or all options rejected (None option is chosen)
    - various metrics, but the most stringent is success rate for the whole task
        - 7.1%, 5.1%, 2.9% as we increase the test difficulty, while baseline (DeBERTa ft) gets near 0 for all
- Weakness
    - Static environment (agent only sees html and needs to predict the action sequence, even if action sequence changes the HTML midway via JS (eg clicking a drop down bar))
        - however i see in the DOM tree that there are some JS libraries included -- maybe that can be used to predict the dynamics of the website?
    - Fixed GT trajectory -- might have alternate trajectories which achieve same goal


### WebShop: Towards a Generalist Agent for the Web
[[Code](https://github.com/princeton-nlp/WebShop)]
[[Site](https://webshop-pnlp.github.io/)]

### MiniWob++
[[Code](https://github.com/Farama-Foundation/miniwob-plusplus)]

largely solved (close to perfect accuracy) 


------

# Papers

Currently only those that work on WebShop as that environment is mature enough to have methods attempting to tackle it, but still remains challenging

### ReAct
[[Paper](https://arxiv.org/abs/2210.03629)]
[[Site](https://react-lm.github.io/)]
[[Code](https://github.com/ysymyth/ReAct)]
- Context limit handled by getting last x characters 

### Reflexion: Language Agents with Verbal Reinforcement Learning
[[Paper](https://arxiv.org/abs/2303.11366)]
[[Code](https://github.com/noahshinn024/reflexion)]


### Auto-GPT for Online Decision Making: Benchmarks and Additional Opinions
[[Code](https://github.com/younghuman/LLMAgent)]
[[Paper](https://arxiv.org/abs/2306.02224)]

AutoGPT implementation for WebShop and ALFWorld

Features:
- Receives only high-level goals and instructions at the beginning of complex multi-step tasks
- Thoughts, Reasoning, Plan, and Criticism for each step (effectively, CoT and Reflexion)
- Tool use
- Chat memory + retrieval of relevant memory based on last 10 messages

"Incorporating a second opinion from a supervised learner can significantly enhance task performance". This is finetuned on WebShop human trajectories

#### Prompt token count

| Description | Token Count |
| ------------- | ------------- |
| Total   | 4196 (but actually GPT3.5 only has 4097 tokens)  |
| Base prompt + Retrieved Memory  | 2500  |
| Allowance for LM response + input msg  | 1000  |
| Remainder for historical messages | 597  |

### Tool Learning with Foundation Models
[[Paper](https://arxiv.org/abs/2304.08354)]
[[Code](https://github.com/OpenBMB/BMTools)]


### Hierarchical Prompting Assists Large Language Model on Web Navigation 
[[Paper](https://arxiv.org/abs/2305.14257)]
[[Code](https://github.com/robert1003/ash-prompting)]


### Auto-GPT
[[Code](https://github.com/Significant-Gravitas/AutoGPT)]

- multi tool use
- break into subtasks, then use tools to try and achieve those
    - wiki: " manages short-term and long-term memory by writing to and reading from databases and files; manages context window length requirements with summarization; can perform internet-based actions such as web searching, web form, and API interactions unattended; and includes text-to-speech for voice output"
- issues: wiki " Auto-GPT often also has trouble staying on task, both problems which developers continue to try to address. After successfully completing a task, it usually does not remember how to perform it for later use, and when it does, for example when it writes a program, it often forgets to use the program later. Auto-GPT struggles to effectively decompose tasks and has trouble understanding problem contexts and how goals overlap"
