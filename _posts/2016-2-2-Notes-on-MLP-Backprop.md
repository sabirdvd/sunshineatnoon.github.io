---
layout: post
title: Notes on MLP Backprop
published: true
use_math: true
---

Thanks to this great [tutorial](http://cs231n.github.io/neural-networks-case-study/), I finally have some clues about how backpropagation works in multi-layer perceptrons and want to write them down while my memories are still fresh. 

In this post, I will mainly focuse on implementing a simple two-layer perceptron on MNIST. The loss function I chose is softmax. For theoratical understanding of the backpropagation algorithm, I recommend this [tutorial](http://neuralnetworksanddeeplearning.com/chap2.html).

## Notations

- Image array X: an (N,D) array, with each row a flattened image
- Target array y: an (N,1) array, with each row a label of corresponding image
- Hidden: The activation of a hidden layer
- scores: The output of the second(final) layer
- W1,W2: weight array of the first layer and second layer
- b1,b2: biases of the first layer and second layer
- N: The number of training samples
- D: The demension of a training sample
- C: The number of classes

## Feedforward Pass

The feedforward pass is straightforward to understand and implement. At every layer, we just multiply the inputs with weights, add a bias and go through an activation function. Here I will use [Rectified Linear Unit](https://en.wikipedia.org/wiki/Rectifier_(neural_networks).
#### Equations

- Hidden = ReLU(X*W1 + b1)
- scores = Hidden*W2 + b2

#### Implementation

  ```
  def Forward(self,X):
  
     Hidden = np.maximum(0,X.dot(self.W1) + self.b1)
     score = Hidden.dot(self.W2) + self.b2
     return Hidden,score
  ```
  
## Normalize scores into probabilities

A probablistic interpretation about softmax classifers is that we can regard the scores predicted for each class as unnormalized probabilities. So after using the softmax function to normalize the scores, we get probabilities indicating how likely an image belongs to a certain class.

#### Equations

$$p_k = \frac{e^{s_k}}{\sum{e^{s_j}}}$$

#### Implementation

```
probs = np.exp(scores)/np.sum(np.exp(scores),axis=1,keepdims=True)
```

So probs now is a (N,C) matrix and probs[i,j] indicates the probability of the ith sample belonging to the jth class.

## Loss

#### Equations

$$L_i = -log(\frac{e^{s_{y_i}}}{\sum{e^{s_j}}}) $$

$$L_i = -log(p_{y_i}) $$

$$L = \frac{1}{N}{\sum{L_i}} + \frac{1}{2}{\lambda}({W_1^2+W_2^2}) $$

#### Implementation

```
correct_logprobs = -np.log(probs[range(N),y])
data_loss = np.sum(correct_logprobs)/N
reg_loss = 0.5 * reg * (np.sum(self.W1 * self.W1) + np.sum(self.W2*self.W2))
loss = data_loss + reg_loss

```

In the above implementation, the correct_logprobs is a (C,1) matrix, with each row indicating the predicted probability for the correct class of this sample, i.e. the $p_{y_i}$ in the equation above.

## Gradient

Here is what backprobagation comes in. The key for backprobagation is the <strong>chain rule</strong>. From back to front, we regard each operation as a gate, a gate receives a bunch of gradients and pushes them back to its input varibles. For instance, the addition gate below receives a gradient and distribute this gradient to all its input varibles equally. More informations about different gates can be found [here](http://cs231n.github.io/optimization-2/).

![Addition Gate](https://raw.githubusercontent.com/sunshineatnoon/sunshineatnoon.github.io/master/images/mlp.png)

#### Equations

$$L_i = -log(\frac{e^{s_{y_i}}}{\sum{e^{s_j}}}) \Longrightarrow \frac{\partial L_i}{\partial s_k} = \frac{e^{s_k}}{\sum{e^{s_j}}} - {1({y_i == k})}$$

$$s = Hidden * W_2 + b_2 \Longrightarrow \frac{\partial L_i}{\partial W_2} = \frac{\partial L_i}{\partial s} * \frac{\partial s}{\partial W_2} = Hidden^T * \frac{\partial L_i}{\partial s}$$

$$s = Hidden * W_2 + b_2 \Longrightarrow \frac{\partial L_i}{\partial b_2} = \frac{\partial L_i}{\partial s} * \frac{\partial s}{\partial b_2} = \frac{\partial L_i}{\partial s}$$

$$s = Hidden * W_2 + b_2 \Longrightarrow \frac{\partial L_i}{\partial Hidden} = \frac{\partial L_i}{\partial s} * \frac{\partial s}{\partial Hidden} = \frac{\partial L_i}{\partial s} * W_2^T$$

$$Hidden = ReLU(X*W_1 + b_1) \Longrightarrow \frac{\partial L_i}{\partial W_1} = \frac{\partial L_i}{\partial Hidden} * \frac{\partial Hidden}{\partial z} * \frac{\partial z}{\partial W_1}= X^T * \frac{\partial Hidden}{\partial z} * (\frac{\partial L_i}{\partial Hidden})^T$$

$$Hidden = ReLU(X*W_1 + b_1) \Longrightarrow \frac{\partial L_i}{\partial b_1} = \frac{\partial L_i}{\partial Hidden} * \frac{\partial Hidden}{\partial z} * \frac{\partial z}{\partial b_1}= \frac{\partial Hidden}{\partial z} * (\frac{\partial L_i}{\partial Hidden})^T$$

The equations above may seem intimatating, but they are nothing more than the results of chain rule and pretty easy to implement. A trick when deriving all these equations is <strong>demension-match</strong>. For instance, from $s = Hidden * W_2 + b_2$ and chain rule, we know that $\frac{\partial L_i}{\partial W_2}$ should be the multiplication of $\frac{\partial L_i}{\partial s}$ and $\frac{\partial s}{\partial W_2} = Hidden$. But in what order should we multiply these two matrixes and do we need to transpose them? Here is when we can use demension-match. We know the demension of $\frac{\partial L_i}{\partial s}$ is the same as s, which is a (N,C) matrix and the demension of Hidden is (N,H) and the demension of our target $\frac{\partial L_i}{\partial W_2}$ is (H,C), so the only way to achieve this is:  $Hidden^T * \frac{\partial L_i}{\partial s}$. Then we got our equation!

#### Implementation

```
dscores = probs
dscores[range(N),y] -= 1
dscores /= N


dW2 = Hidden.T.dot(dscores) + reg * self.W2
db2 = np.sum(dscores,axis=0,keepdims=True)

dHidden = dscores.dot(self.W2.T)
dHidden[Hidden <= 0] = 0 #Back prop ReLU

dW1 = X.T.dot(dHidden) + reg * self.W1
db1 = np.sum(dHidden,axis=0,keepdims=True)

```

Above implementation is a straightforward translation from equations to code. Special attention maybe needed to the fourth line of the code, which pushes gradient through the ReLU gate. The ReLU is just a max gate, which only distributes one to its maximum input and zero to all other inputs.

## Reference
[1] [http://cs231n.github.io/neural-networks-case-study/](http://cs231n.github.io/neural-networks-case-study/)

[2] [http://cs231n.github.io/optimization-2/](http://cs231n.github.io/optimization-2/)

[3] [http://neuralnetworksanddeeplearning.com/chap2.html](http://neuralnetworksanddeeplearning.com/chap2.html)

##Code

The code is available on my [GitHub](https://github.com/sunshineatnoon/Deep-Learning-Practice). It gets about 93% accuracy on MNIST.





















