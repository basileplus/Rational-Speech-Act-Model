# Rational Speech Act (RSA) Model

## Table of Contents
1. [Introduction](#introduction)
2. [Rational Speech Act (RSA) model explanation](#rational-speech-act-rsa-model-explanation)
3. [Dataset Creation](#dataset-creation)
    1. [Creating the Dataset](#creating-the-dataset)
4. [Simple RSA Model](#simple-rsa-model)
    1. [Definition of the model](#definition-of-the-model)
        1. [The literal listener](#the-literal-listener)
        2. [The pragmatic speaker](#the-pragmatic-speaker)
        3. [The pragmatic listener](#the-pragmatic-listener)
    2. [Use example](#use-example)
        1. [Maximum performance](#maximum-performance)
        2. [Results](#results)
5. [RSA with cost](#rsa-with-cost)
6. [RSA with prior](#rsa-with-prior)
7. [RSA with recursivity](#rsa-with-recursivity)
8. [Conclusion](#conclusion)


---

## Introduction
This document presents a comprehensive analysis of the Rational Speech Act (RSA) model, model implementation, and various enhancements including cost and prior knowledge. The code is provided in the ready-to-compile notbook available on the repo.

## Rational Speech Act (RSA) model explanation

RSA formalize interaction between agents. In the following exemple, two agents are discussing about a set of objects $o_1; o_2$. Each object has several attributes to describe them, in the example $a_1,a_2$.

For each of the object and each of the attribute, we define a truth condition : 0 if the object doesn't have the attribute, 1 if it has it.

A classic example to describe an application of the model is the following :
- Our two objects are two personnes, one with a hat and one with a hat and glasses
- The two attributes are then 'hat' and 'glasses'
- The truth matrix is then the following :

    | Object \ attributes | Glasses | Hat |
    |--------|-----|---------|
    | Person 1  | 1   | 0       |
    | Person 2  | 1   | 1       |

The speaker wants to describe one of the two objects (the 2 persons) to the listener. The speaker has to chose an attribute to describe it and an attribute to describe it. 

If the speaker wants to describe the person with no hat, he will say "the person I'm talking about has glasses". 

Now if the listener is _pragmatic_, he will without a doubt understand that the speaker is talking about the person with glasses and no hat, because "hat" is discriminative and if the speaker wanted to talk about the person with hat, he would have said "the person I'm talking about has a hat".

The Rational Speech Act model theorize this interaction between the speaker and the listener, supposing both the listener and the speaker are _pragmatic_ and have common knowledge of the truth table.


## Dataset Creation
### Creating the Dataset

As a Dataset, we use a random truth matrix $T : O\times A \to \{0,1\}$ 
- $O$ : number of objects
- $A$ : number of attributes for each object
This matrix represent for each object $o$ and each property $a$ if the object has the property or not.

A classic example of such a dataset is the following :


## Simple RSA Model

### Definition of the model

#### The literal listener

We begin by defining the parameters of our model :
- $\alpha$ : how much the speaker will be pragmatic
- $P$ : the prior probability of each object. If we already have information about the probability that the speaker will chose the object $o$ to describe.
- $C$ : the cost of each attribute. Typically the number of bits needed to transmit the attribute. 

We then define the _literal listener_ as the probability that the object $o$ has the attribute $a$ given the description $d$ of the speaker. This is defined as :
$$
L_0(o|a) = \frac{T(o,a)}{\sum_{o'} T(o',a)}
$$

Where $T(o,a)$ is the the coefficient of the truth matrix at the position $(o,a)$ which defines whether the object $o$ has the attribute $a$.

This literal listener is the naive one.

#### The pragmatic speaker

The speaker will choose the object that maximizes the probability that the listener will understand the object.

$$
S_1(d|o) \propto \exp(\alpha \log L_0(o|d) - C(d))
$$

where $C(d)$ is the cost of the description $d$, for now 0

#### The pragmatic listener

The pragmatic listener will then infer the object $o$ given the description $d$ of the speaker. It is defined as :
$$
L_1(o|d) \propto P(o) S_1(d|o)
$$

The obtained matrice $L$ is the probability that the speaker is takling about the object $o$ given the attribute $a$.

$o$ is the row and $a$ is the column.

### Use example

We use the precedent example with the following matrix $T$ :
$$
\begin{pmatrix}
0 & 1 \\
1 & 1
\end{pmatrix}
$$

#### Maximum performance

As values are chosen randomly it is possible than several objects have the same attributes. They are then not distinguishable. We thus are going to calculate the maximum performance of the model by counting the number of objects that are undistinguishable. We have to be concious of this for the following results.

#### Results

We obtain the following results :

|T:|	p1	| p2|
|---|---|---|
|o1|	0.0|	1.0|
|o2|	1.0|	1.0|

|Literal listener:|	o1|	o2|
|---|---|---|
|p1|	0.0|	1.0|
|p2|	0.5|	0.5|

|Pragmatic speaker:|	p1|	p2|
|---|---|---|
|o1|	0.0|	1.0|
|o2|	0.7|	0.3|

|Pragmatic listener:|	o1|	o2|
|---|---|---|
|p1|	0.0|	1.0|
|p2|	0.8|	0.2|

Now we can see what the model would do for a given object.
If the speaker wants to describe the object 1 ``o1`` : Both listener have chosen the correct object but the pragmatic listener is more confident on its choice.

## RSA with cost

We can add a cost function to the model. The cost function is a matrix $C$ of size $O\times A$ where $C[o,a]$ is the cost of the attribute $a$ for the object $o$. Intuitively, we can take as the cost the number of bits needed to transmit the attribute.

$$
S_1(d|o) \propto \exp(\alpha \log L_0(o|d) - C(o,d))
$$

Now the speaker will take into account the cost of the attributes to choose the object to describe.

## RSA with prior

We can also add prior knowledge of the probability distribution of the objects. For instance we may know that we are more likely to describe object 1 than object 2. We can add this information in the model by adding a prior matrix $P$ of size $O$ where $P[o]$ is the prior probability of the object $o$. 

This probability will be added in the litteral listener. For now we initialized the litteral listener with a uniform distribution as follows :
$$
L_\text{lit}(o|a) = \frac{T(o,a)}{\sum_{o'} T(o',a)}
$$
Now we can add the prior information in the litteral listener as follows :
$$
L_\text{lit}(o|a) = \frac{T(o,a)P[o]}{\sum_{o'} T(o',a)P[o']}
$$

We can test this new function with the example below using the prior matrix $P = [0.25, 0.75]$. We obtain :

|Literal listener | o1 | o2 |
|---|---|---|
|p1|	0.0|	1.0|
|p2|	0.5|	0.5|

|Literal listener with prior knowledge:|	o1|	o2|
|---|---|---|
|p1|	0.0|	1.0|
|p2|	0.2|	0.8|

## RSA with recursivity 

We saw that we used the litteral listener to calculate the pragmatic speaker. Then the pragmatic speaker to calculate the pragmatic listener. We can reiterate this process to get a better model : a model which is more pragmatic.

Let's build iterative functions ``pragmatic_listener`` and ``pragmatic_speaker`` which take as input a parameter ``k`` indicating the number of iterations.

Those functions will be methods of the class ``RSA``

We can test this new model with the example below using the prior matrix $P = [0.25, 0.75]$ and the cost matrix $C = \begin{bmatrix} 1 & 1  \\ 1 & 1  \end{bmatrix}$. We obtain :

| T:  | p1   | p2   |
|-----|------|------|
| o1  | 0.00 | 1.00 |
| o2  | 1.00 | 1.00 |

| Literal listener: | o1   | o2   |
|-------------------|------|------|
| p1                | 0.00 | 1.00 |
| p2                | 0.25 | 0.75 |

| Pragmatic speaker: | p1   | p2   |
|--------------------|------|------|
| o1                 | 0.00 | 1.00 |
| o2                 | 0.84 | 0.16 |

| Pragmatic listener: | o1   | o2   |
|---------------------|------|------|
| p1                  | 0.00 | 1.00 |
| p2                  | 0.86 | 0.14 |

Thus, we can see that the model becomes more confident in its choices as k increases (here we chosed k=3 instead of 1 previously). 

We can represent the evolution of the standard deviation of the probabilities for each object as a function of k to quantify this improvement.

![output](https://github.com/basileplus/Rational-Speech-Act-Model/assets/115778954/8fd9c1f2-28ee-442a-beb7-b0b8aff4b8eb)


## Conclusion

We have seen that the RSA model can be used to model the interaction between a speaker and a listener. We have seen that the model can be improved by adding cost and prior knowledge. We have also seen that the model can be iterated to get a more pragmatic model.
