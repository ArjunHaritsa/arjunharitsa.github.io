+++
title = "Quantization in deep learning architectures"
date = 2026-01-12
tags = ["AI", "TensorflowLite"]
categories = ["DeepLearning"]
language = "en"
math = true
+++

## Introduction

Quantization, in deep learning is a lossy compression process of mapping weights of a neural network model from real values to fixed bit-width integers.

The process of quantization, can be effectively applied either while training a network or post training the network - depending on the input data distribution within a dataset and the kernels distribution within the network. The mapping constraints a network however, for a chosen bit-width of integers while yielding an affordable accuracy drop in the model's performance.

Quantization, improves memory footprint of a model; specifically, in resource constrained devices. In tailored hardwares - consisting of [Accelerators](https://en.wikipedia.org/wiki/Hardware_acceleration) or CPUs supporting [**SIMD**](https://en.wikipedia.org/wiki/Single_instruction,_multiple_data) quantization can help in vectorization of weights (learnt parameters) and activations (intermediate computations between layers of a network) in accordance with the core-matrix multiplier register arrays of the hardware; thereby, increasing the inference speed of a model significantly.

### Formulation

---

Mathematically, the weights of a kernel undergoes transformation from $\mathbb{R}$ domain to $\mathbb{Z}$ as,

$$ \\{ x \ \epsilon \ \mathbb{R} \ | \ \alpha \leq x \leq \beta\ \\} \ \mapsto \\{ x_q \ \epsilon \ \mathbb{Z} \ | \ \alpha_q \leq x_q \leq \beta_q \ \\} $$

The range of the integer values are decided by $b$-bits of integer representation chosen for mapping $x \ \mapsto x_q$.

For example, if the range of $x$ is $[-8.05,  8.08]$ - by choosing $b=8$, the quantized space will have $2^b=2^8=256$ values. The quantized space can be uniformly or non-uniformly spread across the ranges where only integers can exist between $[-128,  127]$. For instance, in an uniform distribution $\forall x$ each $x$ will be mapped to a nearby integer value $x_q$ representing a constant scale of $(-8.0 + x_q * (1/256))$ in the original domain.

We will see in the later section, distinct approaches of achieving the scaling factor of the quantization process. For most of the practical cases in edge computing, the uniform (or linear) integer mapping is far than enough to represent a mathematical approximation to which an outcome belongs to (an example can be [classification problem](https://www.ibm.com/think/topics/classification-machine-learning)).

### Transformations

---

Theoretically, the weights can be mapped from $\mathbb{R}$ to $\mathbb{Z}$ either by,

- Linear spacing (uniform): Quantized values are equally spaced in the original $\mathbb{R}$ domain and mapped linearly
- Non-Linear spacing (non-uniform): A careful mapping of the $\mathbb{R}$ values depending on the part of the data that often occurs frequently but has data at larger extremes as well. The scaling depends on the position at which the bit value is represented at

#### Linear Transformation

---
Let us consider linear scaling for the transformation and define the quantization process as:
$$
\begin{equation}
x_q = round(x/s - b) \tag{1} \label{eq:quantization}
\end{equation}
$$  
achieves the integer mapping. Note that, the '_round off_' introduces noise into the conversion.

Later, to rescale the data into a real value, de-quantization process will be applied through:
$$
\begin{equation}
x = s(x_q + b) \tag{2} \label{eq:dequantization}
\end{equation}
$$

When $x$ is at $\alpha$ the scaling imposes $\alpha_q$ after transformation. Similarly, $\beta_q$ will be the final outcome when $x$ reaches $\beta$. In reference to equation (\ref{eq:quantization}), we can confide the values of $s$ and $b$ by formulating a linear systems of equation[$^1$][Link 1] as we are trying to solve for $s$ and $b$ that imposes the minima and maxima of the transformation,
$$
\begin{equation}
\alpha=s(\alpha_q  + b) \tag{3} \label{eq:alpha scaling}
\end{equation}
$$

$$
\begin{equation}
\beta=s(\beta_q  + b) \tag{4} \label{eq:beta scaling}
\end{equation}
$$

To solve for $b$, we can rearrange $s$ in the equation (\ref{eq:alpha scaling}) to be

$$
\begin{equation}
s=(\alpha / (\alpha_q + b)) \tag{1a} \label{eq:alpha scaling rearrangement}
\end{equation}
$$

substituting equation (\ref{eq:alpha scaling rearrangement}) in equation (\ref{eq:beta scaling})  we get,

$$
\begin{equation}
\beta = {(\alpha \beta_q + \alpha b) \over (\alpha_q + b)} \tag{4a}
\end{equation}
$$

$$
\begin{equation}
\alpha \beta_q - \beta \alpha_q + b (\alpha - \beta) = 0 \tag{4b}
\end{equation}
$$

$$
\begin{equation}
b = {(\alpha \beta_q - \beta \alpha_q) \over (\beta - \alpha)} \tag{4c} \label{eq:unrounded zero point}
\end{equation}
$$

Technically, equation (\ref{eq:unrounded zero point}) cannot be a fractional number. It is the zero-point or the base point from which the scaled numbers are distributed. This could be derived by keeping $x=0$ in equation (\ref{eq:quantization}), resulting in mapped $x_q = round(-b)$. Therefore we must impose,

$$
\begin{equation}
\boxed{z = -round({(\alpha \beta_q - \beta \alpha_q) \over (\beta - \alpha)})} \tag{5} \label{eq:zero point}
\end{equation}
$$

and substituting $b$ from equation (\ref{eq:unrounded zero point}) in equation (\ref{eq:alpha scaling}) we can simplify the equation in terms of $s$ as,

$$
\begin{equation}
\beta = {s (\beta_q (\beta - \alpha) + \alpha \beta_q - \beta \alpha_q) \over (\beta - \alpha)} \tag{3a}
\end{equation}
$$

$$
\begin{equation}
s = {\beta (\beta - \alpha) \over \beta_q (\beta - \alpha) + \alpha \beta_q - \beta \alpha_q}\tag{3b}
\end{equation}
$$

$$
\begin{equation}
\boxed{s = {(\beta - \alpha) \over (\beta_q - \alpha_q)}} \tag{6}
\end{equation}
$$

We can finally redefine the quantization transformation process from equation (\ref{eq:quantization}) to
$$
\begin{equation}
x_q = round((x/s) + z)
\end{equation}
$$

where the mapping constraint must be $\\{ \alpha_q <= x_q <= \beta_q \\}$ and the values needs to be clipped off if they are beyond the boundary i.e.,
$$
\begin{equation}
\boxed{x_q = clip(round((x/s) + z), \alpha_q, \beta_q)} \tag{7} \label{eq:final quantization}
\end{equation}
$$

The de-quantization transformation process from equation (\ref{eq:dequantization}) can then be defined as,

$$
\begin{equation}
\boxed{x = s (x_q  - z)} \tag{8} \label{eq:final dequantization}
\end{equation}
$$

#### Non-Linear Transformation

---

A non-uniform transformation extends the linear transformation merely by two additional steps,

1. By compressing the weights of a kernel with a non-linear function and then linear transforming the data
2. At inference, by decompressing the quantized data - while computing the intermediates of the forward pass

We can mathematically formulate these steps by extending the equation (\ref{eq:final quantization}) to define quantization process as:

$$
\begin{equation}
\boxed{x_q = clip(({round(f(x) * \lambda) \over s}) + z), \alpha_q, \beta_q)} \tag{9} \label{eq:final non-linear quantization}
\end{equation}
$$

Note that, $\forall x$ the $\lambda$ in equation (\ref{eq:final non-linear quantization}) controls the squashing factor of the near by values. The choice of $\lambda$ is optional but depends upon the compression function that is applied. The $\lambda$ often is the resolution dial of the overall compression function and will be within the space $\\{ \alpha < \lambda < \beta \\}$.

The dequantization process however running at the inference, has to decompress the data with an exact inverse function of the equation (\ref{eq:final non-linear quantization}) as,

$$
\begin{equation}
\boxed{x = s (({f^{-1}(x_q) \over \lambda})  - z)} \tag{10} \label{eq: final non-linear dequantization}
\end{equation}
$$

The functions $f(x)$ and $f^{-1}(x_q)$ are to be chosen carefully depending on the architecture and the kernel distribution of the network, however the squashing factor $\lambda$ will be a parameter learnt while training the network or post training - for a chosen (de-)compression function.

In deep learning, often the kernel appears to have _Gaussian distribution_ with few outliers. Lets consider, `arcsinh` a specialized non-linear function often used in LLMs to compress 16-bit float to 4-bit integers!

## References

1. [Leo MAO's Log Book - Quantization for Neural Networks][Link 1]
2. [Quantization and Training of Neural Networks for Efficient
Integer-Arithmetic-Only Inference][Link 2]

[Link 1]: https://leimao.github.io/article/Neural-Networks-Quantization
[Link 2]: https://arxiv.org/pdf/1712.05877
