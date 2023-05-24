---
title: Notes for Singular integrals and differentiability properties of functions
date: 2023-05-23 20:57:50
categories:
tags:
---

 <!-- more -->



Singular integrals and differentiability properties of functions-Elias M. Stein - Princeton University Press (1970)

## 1-1
The most important theorem: Hardy Littlewood Maximal function Inequality.

Calderón-Zygmund Decomposition


## 2-2
Thm 1:



Condition (b): 
$$
|\nabla K(x)| \leq B/|x|^{n+1} \qquad \forall x \neq 0
$$
Proof Steps of weak 1-1:
1. Calderon-Zygmund decomposition. The good function can be controled easily by T is weak 2-2.
2. The obstacle is the bad function, which is 0 in the "small set" $F$, and integration is 0 in any cube $Q_k$. Decompose $b$ to each of these cube $Q_i$ as $b_i$ . Considering any $x\in F$,  we can modify the convolution $T(f)$ by a constant times $\int b_j(x)$ .
3. Use Mean value thm, and by **porportional fact**, we get 
4. Marcinkiewicz integral
5. We get weak 1-1 !

## 2-3
Comparing to Thm with Condition (b'), which is
$$
\int_{|x|\geq2|y|} |K(x-y)-K(x)|dx\leq B' \qquad |y|> 0
$$
Consider slightly bigger cubes $Q_j^*$ which share the same center $y^j$ with cube $Q_j$, but exapnd its diameter $2n^{\frac{1}{2}}$ times. 

> Just because we have the $|x|\geq2|y|$ in condition (b')  

By  Fubini we can evaluate $\int_{Q_j}\leq\int_{Q_j^*}$ with condition (b'), and get to (15) in the original proof.

**Think: what is the relation between (b) and (b')?**


Now also change the condition (a)
$$
|\hat{K}(x)|\leq B
$$
to condition (a')
$$
|K(x)| \leq B|x|^{-n}
$$
???

We try to get Thm 2:
Suppose the kernel $K(x)$ satisfies the conditions

$$
\begin{align}
&|K(x)| \leq B|x|^{-n} \qquad &|x|>0 \\
& \int_{|x|\geq2|y|} |K(x-y)-K(x)|dx\leq B \qquad &|y|>0
\end{align}
$$
> then second line is called a "regularity property", for it discribe some kind of smoothness 

and the cancellation property
$$
\int_{R_1<|x|<R_2} K(x)dx=0 \qquad 0<R_1<R_2<\infty
$$
For $f\in L^p(\mathbb{R}^n),\; 1<p<\infty$ , let 
$$
T_\varepsilon(f)(x)= \int_{|y|\geq \varepsilon}f(x-y)K(y)dy
$$
then we have
$$
\|T_\varepsilon(f)\|_p \leq A_p \|f\|_p
$$
with $A_p$ independent of $f$ and $\varepsilon$ . 
And $\forall f\in L^p(\mathbb{R}^n),\; 1<p<\infty$, we have
$$
\lim_{\varepsilon \to 0}T_\varepsilon(f) =: T(f)
$$
exists in $L^p$ norm. The operator $T$ so defined also satisfies the inequality above.

Steps of proof:
1. We take Thm1 as granted, and the modification of condition(b') as granted;
2. We show that if we truncated the operator $K$ to some $K_\varepsilon$ , then we have a Lemma(in 2-3.3) that  $K_\varepsilon \in L^2$ , and $$\sup_{y} |\hat{K}_\varepsilon(y)| \leq CB \qquad \forall \varepsilon >0$$ where $C$ depends only on the dimension $n$.
	- the proof of lemma is some kind of tricky: firstly we do only for $K_1$
	- Then we 
	- 
	- 
  3. the $K_\varepsilon \in L^2$ and fixed $C$ give us the condition (a) in Thm1, so we can just use it to obtain a uniform $L^p$ bound for $T_\varepsilon$ ;
  4. For the existance of limit, we first consider smooth function with compactly support: seperate $T_\varepsilon(f)$ to $\int_{|y|>1}$ and $\int_{1 \geq |y|\geq \varepsilon}$ , the first part is fixed and is a $L^p$ function, modify the second part with $f_1(x)$ times the cancellation property integral, then by Mean Value Thm we get it has a limit;
  5. For arbitrary $f\in L^p(\mathbb{R}^n)$ , we use the approximation by smooth function with compactly support. The approximation is in norm, so by the inequality $\|T_\varepsilon(f)\|_p \leq A_p \|f\|_p$ , the leftover can be dropped;
  6. So finally the limiting poerator T is well defined, and by taking limit on both sides of the inequality $\|T_\varepsilon(f)\|_p \leq A_p \|f\|_p$  we get inequality $\|T(f)\|_p \leq A_p \|f\|_p$ . Finished!

### Operator topology

https://en.wikipedia.org/wiki/Operator_topologies


# 2-4
> 2023/05/16-
> 2023/05/22 meeting
> 2023/05/23 
> 

singular integral operators which commute with dilations, and also with translations.

commute with dilations: homogeneous of degree -n

commute with transformations: T can be realized in terms of a multiplier $m$, s.t. $T(f)\hat{} = \hat{f} \cdot m$      

So we know that any singular integral operators which commute with dilations and translations will fall into the situation mentioned in following Theorem 3:

Theorem 3: Let $\Omega$ be homogeneous of degree 0, and suppose that $\Omega$ satisfies this cancellation property 

$$
\int_{S^{n-1}} \Omega(x) d\sigma = 0
$$

and the smoothness property
$$
\text{let } \omega(\delta) = \sup_{|x-x'|\leq \delta,\; |x|=|x'|=1} \int_0^1 \frac{\omega(\delta) d\delta}{\delta} < \infty
$$
then for $1<p<\infty$, $f\in L^p(\mathbb{R}^n)$, let
$$
T_\varepsilon (f) (x) = \int_{|y|\geq \varepsilon} \frac{\Omega(y)}{|y|^n}f(x-y) dy 
$$
(a) Then there exists a bound $A_p$ independent of $f$ or $\varepsilon$ , so that
$$
\|T_\varepsilon(f)\|_p \leq A_p \|f\|_p
$$
(b) The limit of $T_\varepsilon(f)$ exists in $L^p$ norm, and by defining $T(f) = \lim_{\varepsilon \to 0} T_\varepsilon(f)$, we have
$$
\|T(f)\|_p \leq A_p \|f\|_p
$$
(c) If $f\in L^2(\mathbb{R}^n)$, then the Fourier transforms of $f$ and $T(f)$ are related by $(Tf)\hat{} (x) = m(x) \hat{f}(x)$, where $m$ is a homogeneous function of degree 0. We can write out the explicit form of $m$ (....).

#### Cauchy principal value: 
Actually it is a tempered distribution.
https://en.wikipedia.org/wiki/Cauchy_principal_value#Distribution_theory

If we change the way to choose end point, what will we have?

