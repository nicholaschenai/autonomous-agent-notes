# Deprecation Warning
This repo is deprecated and elements will be moved [here](https://github.com/nicholaschenai/agi-potential-notes) over time
# Brief notes on autonomous agents
Literature review + quick notes for my own reference so pardon the untidyness. Will occasionally copy paste directly from the papers!

Autonomous agents generate commands which programmatically finds the answer in an environment, eg web

---
# Benchmarks / Datasets

[BENCHMARKS](BENCHMARKS.md)

------

# Methods

### ReAct
[[Paper](https://arxiv.org/abs/2210.03629)]
[[Site](https://react-lm.github.io/)]
[[Code](https://github.com/ysymyth/ReAct)]
- Context limit handled by getting last x characters 

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

---
# Frameworks
- [metagpt](frameworks/metagpt.md)

---
# Misc
[adversarial examples for agents](https://arxiv.org/pdf/2402.11208.pdf)