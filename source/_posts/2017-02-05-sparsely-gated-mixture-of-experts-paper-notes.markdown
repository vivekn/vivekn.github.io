---
layout: post
title: "Sparsely gated mixture of experts - paper notes"
date: 2017-02-05 21:07
comments: true
categories: 
---

“[The sparsely gated mixture of experts layer](https://openreview.net/pdf?id=B1ckMDqlg)” is an interesting paper by Geoffrey Hinton and others at Google. 

When you have a large number of training examples and you want to extract the most out of your data, one way of doing it would be increasing the capacity of the machine learning models. But this also means that for small incremental gains in accuracy, you have to increase the number of parameters by a few orders of magnitude. This is computationally very expensive, if the entire network is activated for each example. But only certain parts of the network might be needed for different types of inputs. This could be done deterministically by training multiple models and selecting a particular model based on certain heuristics.

But it might be hard to come up with such heuristics and decide what to include in each model. This paper introduces the concept of a trainable gating layer. The model is a collection of experts, each individual expert can be a feedforward neural network, the gating layer selects a small number of experts for each input.

$$ y = \sum_{i=1}^{n} G(x)_i E_i(x) $$

$$ G(x) = Softmax(PreserveTopK(W_gx)) $$

The gating function is a modified softmax layer, which sets the values not in the top $$k$$ to $$-\infty$$. It can be trained jointly with the expert networks using gradient descent. This sparsity ensures that both training and prediction can be really fast as the zeroed out networks don't need to be evaluated or backpropagated through. The final prediction is a sparse linear combination of the experts. 

There are some issues with this approach though, during training the weights on some of the experts can get larger and due to reinforcement only these experts would be trained. The authors suggest two ways to solve this.

1. Adding Gaussian noise to the outputs before applying the softmax.

2. Introduce a loss penalty for unbalanced distribution of examples to experts, penalizing high variance in the distribution of examples.

$$ \text{Importance}_i = \sum_{x \in X} G_i(x)  $$

$$ \text{loss}_{\text{imp}} = w_i CV(\text{Importance}_i) $$

The coefficient of variation ($$CV$$) is the ratio of standard deviation to the mean of a random variable. 

If there are hundreds of experts and mini-batch training is used, each expert network will only receive a small number of examples per batch, which would be computationally inefficient. So, the batch size should be as large as possible. But for a large batch size, all the activations and weight updates should be stored in memory, which is also infeasible for a large number of neural networks. They suggest using a very large batch size for the gating network and then sharding the individual models across multiple machines and training with model parallelism. This way individual networks will still have reasonable batch sizes and it also helps with memory usage during evaluation time.

This is an interesting way of increasing the capacity of the model while keeping computational costs reasonable. Different experts can act on different properties of the data and all of them need not be active simultaneously.


