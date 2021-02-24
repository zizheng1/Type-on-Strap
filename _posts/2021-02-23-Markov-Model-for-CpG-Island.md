---
layout: post
title: Markov Model for CpG Island
tags: [CpG Islands, Markov Chain]
excerpt_separator: <!--more-->
---
Hidden Markov Model for CpG Island

<!--more-->

### Does a Short DNA Stretch Come from a CpG Island?

This is a homework from one of my graduate level Modeling & Simulation class which I find very interesting for the demonstration of Markov Chain Model. We need to model both the CpG Islands and the regions that are not CpG Islands. For the putative CpG Regions compute the transition probabilities from nucleotide $$s$$ to nucleotide $$t$$ using a Markovian Model.

$$
a^+_{st} = \frac{c^{+}_{st} }{ \sum_{t'} c^+_{st'}}
$$

where $$c^+_{s,t}$$ is the number of times that $$s$$ is followed by $$t$$ in the database of CpG Islands. Similarly for the regions without CpG Islands.

$$
a^-_{st}= \frac{c^-_{st}}{\sum_{t'} c^-_{st'}}
$$


$$
    \begin{matrix} 
    \text{Model +} & A & C & G & T\\ 
    A & 0.180 & 0.274 & 0.426 & 0.120\\
    C & 0.171 & 0.368 & 0.274 & 0.188 \\
    G & 0.161 & 0.339 & 0.375 & 0.125\\ 
    T & 0.079 & 0.355 & 0.384 & 0.182 \\ 
    \text{station} & 0.155 & 0.341 & 0.350 & 0.154\\
    \end{matrix}
$$

Also

$$
    \begin{split}
    \begin{array} {crrrr} 
    \text{Model -} & A & C & G & T\\ 
    A & 0.300 & 0.205 & 0.285 & 0.210\\ 
    C & 0.322 & 0.298 & 0.078 & 0.302 \\ 
    G & 0.248 & 0.246 & 0.298 & 0.208\\ 
    T & 0.177 & 0.239 & 0.292 & 0.292 \\ 
    \text{station} & 0.0.262 & 0.246 & 0.239 & 0.253\\
    \end{array}
    \end{split}
$$

Then calculate the Log-Odds ration for a sequence $$x$$:

$$
    S(x)=log_2 \frac{[P(x|model+)]}{[P(x|model-)]} = \sum_{i=1}^L log_2 \frac{a^+_{x(i-1)x(i)}}{ a^-_{x(i-1)x(i)}} = = \sum_{i=1}^L log_2 \beta_{x(i-1)x(i)} +log_2 \frac{[P(x_1\mid model+)]}{[P(x_1\mid model-)]}
$$

Scores $$S(x)$$ allows discrimination of a model $$+$$ against another $$-$$

We can summarize the computations by writing once and for all the $$log_2 \beta$$ matrix, and adding each relevant term from this matrix, for instance from the two transition matrices above we have:

$$
log_2 \beta_{AC}=log_2 \frac{a^+_{AC}}{ a^-_{AC}}=log_2 \frac{274}{205}=0.41855
$$

So we can calculate the table, rather than recompute each $$\beta$$ every time:

$$
\begin{split}\begin{array} {crrrr} \beta & A & C & G & T\\ A & -0.740 & 0.419 & .580 & -0.803\\ C & -0.913 & 0.302 & 1.812 & -0.685 \\ G & -0.623 & 0.461 & 0.331 & -0.730\\ T & -1.169 & 0.573 & 0.393 & -0.679 \\ \end{array}\end{split}
$$

For example, the sequence $$x = \{CACTAAGCTA\}$$ would have a score: $$-0.913+0.419+1.812-1.169-0.740+0.580+0.461-0.685-1.169-0.757=-1.404$$ normalized by length: $$-2.161/9=-0.2401$$. The score is negative indicating that this is not a CpG Island. 

### MATLAB Implementation

```matlab
v_1 = eig(ModelPlus');
[V,D] = eig(ModelPlus');
P = V(:,1)';
P = P./sum(P); % Stationary Distribution
```

