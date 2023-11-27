---
title: LLM
date: 2023-11-27
tags:
  - sapling
enableToc: true
---
## Notes from [Intro to Large Language Models](https://www.youtube.com/watch?v=zjkBMFhNj_g&ab_channel=AndrejKarpathy)

- Training process
	1. Chunk of the internet 10TB of text
	2. Run 6000 GPU's for 12 days ($2 million)
	3. 140 GB file of llama 2
- like a zip file of the internet
- Neural network: predict the next word in a sentence
- Transformer model
	- little is known in full detail
		- billions of parameters are dispersed through the network
		- we know how to iteratively adjust them to make it better at prediction
		- we can measure that it works, but we don't know how billions of parameters collaborate to do it
- Think of LLMs as mostly inscrutable artifacts, develop correspondingly sophisticated evaluations
- How to train your chatGPT
	- stage 1: pre-training
		1. Download ~10TB of text
		2. Get a cluster of ~6000 GPUs
		3. Compress the text into a neural network, pay $2 million, wait ~12 days
		4. Obtain base model
	- stage 2: fine-tuning
		1. write labeling instructions
		2. hire people and collect 100k high quality ideal Q&A responses, and/or comparisons
		3. fine-tune base model on this data, wait ~1 day
		4. obtain assistant model
		5. run a lot of evaluations
		6. deploy
		7. monitor, collect misbehaviors, got to step 1
- LLM scaling laws
	- performance of LLMs is a smooth, well-behaved, predictable function of N: the number of parameters in the network, and D, the amount of text we train on
- LLMs currently only have a System 1 thinking (Thinking fast and slow) -> wants LLMs to have System 2 thinking
	- we want to "think": convert time to accuracy 
- Want self-improvement 
	- What does step 2 look like in the open domain of language (lack of reward criterion)
- LLM OS could be what comes next
- **LLM security**
	- Jailbreak: certain prompts can fool chatGPT through roleplay
	- universal transferable suffix 
	- certain images have enough noise to jailbreak model
	- prompt injection attacks (white text on white backgrounds)
	- data poisoning/backdoor attacks 
		- trigger words causes model outputs to become random or changed in a specific way

![[Pasted image 20231127110732.png]]