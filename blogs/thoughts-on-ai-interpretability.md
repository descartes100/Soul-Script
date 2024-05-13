---
description: First published on Substack on Feb 26, 2024
---

# Thoughts on AI Interpretability

My contemplation on AI interpretability has been ongoing for some time. The spark for this interest likely traces back to my high school years, during the widespread acclaim of AlphaGo and DeepMind, which fueled my curiosity about the workings of AI. Diving into this field, I was immediately confronted with dizzying arrays of matrices and linear algebra. After learning about the logic behind neural network designs and conducting some experiments in deep learning, my enthusiasm for AI waned, leading me to explore other technological fields, mainly security and blockchain.

What caused this shift?

A pervasive, uncontrollable sense of incomprehension—I realized that the more effective an AI system, the more complex and inexplicable it became, resembling a black box. Influenced by Popper's views, I believe that knowledge that cannot be explained is meaningless. Even if AI algorithms can make better decisions than humans, they are pointless if we cannot understand how these decisions are made. In this process, human knowledge does not expand; it is AI that accrues knowledge, which remains inaccessible and non-reusable by humans, contributing nothing to the advancement of science and truth. At best, AI serves as a peculiar tool, akin to a witch's wand before the Enlightenment. And because we cannot comprehend this tool, controlling it becomes impossible, leading to unpredictable implications.

However, the past year's rise of Large Language Models (LLMs) has made them an undeniable part of everyday life, drawing me back into their study. I was inevitably faced with the question—how should we understand AI? Can the neural network structures, represented through frameworks, function mappings, and parameters, be decoded into insightful knowledge to help humans better understand the world and discover truth?

Fortunately, many in the academic world share this curiosity and have begun exploring the potential of this field. Through reading and applying their research, I've developed some preliminary, unrefined thoughts, which I'll briefly outline here.

Let me start with a brief overview of the current state of AI interpretability research, focusing on the interpretability of deep learning networks, which are most familiar to us.

The first approach to explaining models is what I call the "proxy" method. This method assumes that understanding the operation of deep learning networks, with their parameters numbering in the billions or even trillions, is nearly impossible for our three-dimensional brains. The "proxy" method uses simpler models (mainly linear, low-dimensional) to represent a complex model in a specific local area, allowing us to interpret a complex model through a simpler one. Another approach, which I also categorize under the "proxy" method, involves identifying the most influential parts of the neural network for explanation.

In my view, while the "proxy" method represents a significant portion of AI interpretability methods, it is a compromise that does not address the essence of interpretability. It merely provides a short-sighted engineering solution to gain scientific insights, which naturally has its limitations.

The second method, which I refer to as the "brute-force" approach, boils down to explaining a neural network by detailing the operation logic of each neuron involved. This approach, though straightforward, offers limited explanatory power, transforming a complex issue into another difficult-to-solve problem.

The third method could be termed the "cheating" approach, or more flatteringly, inherently interpretable models. This involves adding a layer during model training to guide the model towards a predefined interpretable path.

Reflecting on these academic approaches to AI interpretability, it's clear that progress in this field is limited, merely scratching the surface of truth without making substantial breakthroughs. The path to interpretability is paved with compromises—explaining only one aspect, transforming the issue into another complex problem, or deluding ourselves with "interpretability."

Why is this the case?

Two main reasons: the problem is indeed challenging, and research on it is still in its early stages.

Why is the problem difficult?

Firstly, there is no clear definition of "AI interpretability" in academia. Each research paper on the topic seems to propose its unique definition. My research has revealed an interesting phenomenon—the focus of AI interpretability research in computer science is more on the "form" of interpretability, while research in other application fields (like biology, finance, etc.) focuses more on the "content."

What are the "form" and "content" of interpretability? Before delving into specifics, let's shift perspectives and discuss what AI "intelligence" really means.

In my view (shared by many scientists), all AI essentially acts as an information compressor. This involves two key concepts—information and compression. Training AI is about compressing vast amounts of data into a large algorithm. The principle of Occam's Razor explains this compression process: among theories making the same predictions, the one with the fewest assumptions should be selected. This principle also explains why many AI models can solve general problems—they follow a universal rule enforced by reward and punishment mechanisms in training.

From this understanding of AI training, we gain several insights:

First, the larger the data volume, the better the AI training outcomes, as it means finding the most common elements among similar problems, closely approximating universal truths. This appears to be a boundless method to acquire "intelligence," as our world continuously generates new data.

Second, this could be considered an instance of "emergence" in complex theory. Just as human intelligence emerges from billions of neurons, AI might explore unknown boundaries using this logic. However, the question remains: how much can we understand AI, and can understanding AI be a shortcut to acquiring truth?

Returning to the "form" and "content" of interpretability, these concepts are intertwined, with form focusing on architectural and tool design, and content on the meanings represented by specific parameters and logic chains. A desirable AI interpretability framework should explain both the neural network itself and the relationship between inputs and outputs. But, the challenge is immense.

This leads to a thought experiment: if a system can explain a model, it must exist in a higher dimension than the model, possessing more knowledge and wisdom. If humans create such a system, who will explain the "knowledge" and "wisdom" of this system? This recursive problem highlights a cognitive race between humans and AI. If humans can always create a more powerful model to explain a previous one, AI remains controllable. However, if we reach a point where human cognition can no longer explain an AI's discoveries, AI becomes a higher intelligence, potentially escaping human control.

As a staunch technologist, I remain optimistic about human intelligence. This confidence stems from the scaling hypothesis. Despite individual limitations, the collective intelligence of billions over generations produces miracles.
