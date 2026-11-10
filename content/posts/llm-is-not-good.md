---
title: "LLM Is Not Good"
titleHtml: '<span>LLM Is Not Go<span class="x-o">o</span>d</span>'
date: 2026-11-10
draft: false
tags: ["LLM", "Brain", "Human"]
summary: "People call LLMs AI and trust them to make decisions. But an LLM is a token generator, and using it as a mentor instead of an assistant costs us responsibility and the ability to think."
ShowToc: true
---

![A brain with ChatGPT, Claude, DeepSeek, and Gemini app icons stacked on top, and a torn note pinned to it: LLM IS NOT GOOD, with one O crossed out so it also reads LLM IS NOT GOD](/images/llm-is-not-good.png)

The 21st century brought many technologies into our world, but probably the most outstanding and the most widespread of them today are LLM models. People call them AI. Wrongly, or more precisely, because the providers of these models told them so. And unfortunately for humanity, this marketing trick is becoming a real poison of the future.

I want to show clearly what an LLM is, and figure out whether we should trust it to make decisions.

## Chats of the '90s

Remember the '90s, when different chats were popular in the internet community. It was quite fun. You could join a chat, pick a nickname and write to people. People could ask all kinds of questions, even how to write a regular expression, or get advice on what to wear on a date, and so on. Familiar?

These were advices from people. Sometimes just flood, sometimes jokes, but they were connected tokens on the topic of the question.

Would you trust that chat with decisions for your business? What about your health? Or choosing a profession? Or a simple command on a server?

You will say it's absurd! How can you trust unverified information from different people? But having said that, you open your favorite LLM tool and... do the same thing. The information gets into this tool from everything: books, websites, chats, etc.

It's just that when you pay, you think this set of tokens is safe and 100% correct. How else? But nobody guarantees you that. The only thing providers guarantee is that the number of tokens per second is such and such, the speed of LLM "thought". That's the whole guarantee.

Of course, LLM mechanisms are more complex and more expensive than advice from a chat, but this absolutely does not change the essence.

## Escape from Responsibility

People started using LLM chats and advice in everyday life so often that sometimes they don't even want to think about the problem. First they ask the LLM, and then, led by the hand by this token generator, they make a decision. From a psychological point of view it's convenient: it wasn't a human who made the decision, so the responsibility isn't theirs, it's the LLM's.

But the problem is that it's the human who asks. A human's health will suffer if they follow the LLM's advice without consulting a specialist. In the end the human is responsible anyway, just not for the human's own psyche.

Escaping responsibility is already a new trend, and it will grow more and more. But without responsibility there is no development. Evolution shows that it's exactly the responsibility for a decision that gives our most powerful tool, our brain, the stimulus to develop.

## A Developer at the End of 2026

Let's look at a typical developer at the end of 2026. They have 2-3 LLMs. They take a task, in most cases without even reading it (and how can you read LLM-generated text from a manager?), and give it to an LLM to implement.

On one side it's simple, fast, zero problems, the code goes further, everyone is happy. But if you look deeper: the developer closes more tasks, the manager creates more tasks, also with an LLM. But which of them, out of 8 working hours, thought about the problem for at least an hour? Who proposed something original and new? Who used the capabilities of their brain and not their hands?

This is the big problem of thinking degradation when using LLMs. Research already shows it:

- **MIT Media Lab, "Your Brain on ChatGPT"** (2025). 54 people wrote essays with ChatGPT, with a search engine, or without anything, under EEG. The ChatGPT group had the weakest brain connectivity, and many could not quote a single sentence from their own essay. The authors call it "cognitive debt". Small sample, preprint. [arXiv:2506.08872](https://arxiv.org/abs/2506.08872)
- **Microsoft Research and Carnegie Mellon** (2025). 319 knowledge workers, 936 tasks. More confidence in AI means less critical thinking. More confidence in yourself means more. [The Register](https://www.theregister.com/2025/02/11/microsoft_study_ai_critical_thinking/)
- **Gerlich, *Societies*** (2025). 666 participants. The more people use AI tools, the lower their critical thinking, because they offload thinking to the tool. Strongest at age 17-25. Correlation, not causation. [MDPI](https://www.mdpi.com/2075-4698/15/1/6)
- **Anthropic** (January 2026). Randomized trial, 52 engineers learning a new Python library. With AI they scored 50% on the quiz, by hand 67%. The biggest gap was in debugging. Those who just delegated to AI scored about 39%, those who generated code and then made sure they understood it scored about 86%. [Anthropic](https://www.anthropic.com/research/AI-assistance-coding-skills)

![Two bar charts. Left: developers learning a new library scored 67% on a quiz when coding by hand, 50% with AI assistance. Right: doctors' adenoma detection rate in colonoscopies without AI fell from 28.4% before exposure to AI to 22.4% after.](/images/llm-chart-skill-loss.svg)

## 2027, 2028...

Now imagine what it will be in 2027, 2028, and so on. When an engineer who could come up with original and bold ideas and take responsibility for rolling them out degrades into an LLM operator?

- **METR** (2025). 16 experienced open-source developers, 246 tasks in their own repositories. Before the study they expected AI to make them 24% faster. After, they believed it made them 20% faster. Measured: with AI they were 19% slower. [arXiv:2507.09089](https://arxiv.org/abs/2507.09089)
- **The Lancet Gastroenterology & Hepatology** (2025). After doctors in four Polish clinics started working with AI, their adenoma detection rate in colonoscopies without AI dropped from 28.4% to 22.4% in a few months. Experienced doctors, not students. Observational study. [News-Medical](https://www.news-medical.net/news/20250813/AI-use-in-colonoscopy-may-weaken-doctors-detection-skills-over-time.aspx)

![Bar chart: developers forecast AI would cut task time by 24%, estimated afterwards it cut time by 20%, but measured completion time was 19% longer with AI.](/images/llm-chart-perception-vs-reality.svg)

## Who Will Move Science Forward?

And who will move scientific progress further, who will solve problems? LLM? But an LLM can't invent new things. All an LLM can do is pick tokens based on past knowledge. An LLM will not formulate a new mathematical problem. It can solve one by brute force, and even that is disputed:

- In October 2025 an OpenAI VP wrote that GPT-5 "found solutions to 10 previously unsolved Erdős problems". Thomas Bloom, who runs erdosproblems.com, called it "a dramatic misrepresentation": GPT-5 found solutions that were already published. The post was deleted. [The Decoder](https://the-decoder.com/leading-openai-researcher-announced-a-gpt-5-math-breakthrough-that-never-happened/)
- On the other side, DeepMind's AlphaEvolve improved Strassen's 1969 result for 4×4 complex matrix multiplication (48 multiplications) and improved known results for about a fifth of 67 problems together with Terence Tao. But it does this by an LLM plus an automatic evaluator trying thousands of variants on problems that humans posed and humans know how to score. That is brute force, not a new problem. [arXiv:2506.13131](https://arxiv.org/abs/2506.13131), [arXiv:2511.02864](https://arxiv.org/abs/2511.02864)

## Shield and Spear

Next, in the world of software development there will be a race of shield and spear. Engineers will run their solutions through several LLMs for system security and close the holes, and hacker groups, on the contrary, will try to find these problems to extort money. And it won't be a race of brains, rather of LLM operators. First cloud, then self-hosted. The engineer will have no time for new ideas or tasks, like a squirrel in a wheel they will close any holes. It will be the dark age of SaaS companies, then there will be local SaaS solutions with access over VPN.

Nobody will think about development, it will stop. The brain won't wait until it's needed. It will just adapt to reality, and the processes of thinking and decision making will atrophy.

## Assistant, Not Mentor

This is the problem of using an LLM not as an assistant, but as a mentor. For some reason people believed that picking tokens knows everything, and they believe blindly. Maybe humanity will find the strength to admit the superiority of its own brain. In any case it will happen. The question is how fast, and what consequences the LLM mentor has for us.

![A cracked, crumbling brain with ChatGPT, Claude, DeepSeek, and Gemini icons driven into it like stakes](/images/llm-brain-atrophy.png)
