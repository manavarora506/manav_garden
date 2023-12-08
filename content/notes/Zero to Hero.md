---
title: Zero to Hero
date: 2023-11-10
tags:
  - evergreen
enableToc: true
---
These will be my notes for Andrej's Zero to Hero playlist on youtube

## The spelled out intro to neural networks and back-propagation: building micrograd

Need to know what a derivative is
- how does a function respond to a slight change in sensitivity (that is the slope)
Back Propagation: for every single value, compute the derivative of that node with respect to the output. Recursive application of chain rule, backwards through the computation graph
- Need to review some basic calculus 
- chain rule
- [[Backpropagation]] helps us fine tune neural networks     
- Review topological sort 
- Review PyTorch
	- very efficient with tensor objects
- A layer of neurons is a set of neurons evaluated independently 
- Gradients for inputs are not very useful because it is fixed
- For very large amounts of data, we use batches which is usually some subset of the data

## The spelled-out intro to language modeling: building makemore

- Bigram language model: only looking at two characters at a time
- https://pytorch.org/tutorials/beginner/basics/tensorqs_tutorial.html
- https://pytorch.org/tutorials/beginner/nlp/pytorch_tutorial.html
- https://cs231n.github.io/python-numpy-tutorial/
- Goal: maximize likelihood of the data w.r.t. model parameters (statistical modeling)
	- equivalent to maximizing the log likelihood (because log is monotonic)
	- equivalent to minimizing the negative log likelihood
	- equivalent to minimizing the average negative log likelihood
	- log(a*b*c) = log(a) * log(b) * log(c)