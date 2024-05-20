---
layout: post
title:  "3D Gaussian Splatting"
excerpt_separator: ""
---


## Background

## Signal sampling therom

- Given a continuous signal $$x_a(t)$$ with frequency spectrum $$X_a(j\Omega)$$, the sampled signal by ratio $$f_s=1/T$$ is discrete:

$$\hat{x}_a(t)=\sum_{n=-\infty}^{\infty}x_a(t)\delta(t-nT),\quad \hat{X}_a(j\Omega)=\frac{1}{T}\sum_{n=-\infty}^{\infty}X_a(j\Omega-jn\frac{2\pi}{T}).$$

- To avoid aliasing caused by sampling, a low-pass filter $$g(t)$$ with band $$[-\frac{\Omega_s}{2},\pi \frac{\Omega_s}{2}]$$ is applied to $$\hat{x}_a(t)$$:

$$g(t)=\frac{\sin(\frac{\Omega_s}{2}t)}{\frac{\Omega_s}{2}t}=\text{Sa}(\frac{\Omega_s}{2}t),\quad G(j\Omega)=\left\{\begin{matrix}
T_s,&|\Omega|\leq \frac{\Omega_s}{2}\\
0,&|\Omega|> \frac{\Omega_s}{2}
\end{matrix}\right.$$

- The filtered output is

$$\begin{aligned}y(t)&=\hat{x}_a(t)\ast g(t)=\sum_{n=-\infty}^{\infty}x_a(nT)g(t-nT)=\sum_{n=-\infty}^{\infty}x_a(nT)\frac{\sin(\frac{\pi}{T}t-n\pi)}{\frac{\pi}{T}t-n\pi}\\&=\sum_{n=-\infty}^{\infty}x_a(nT)\text{Sa}(\frac{\pi}{T}t-n\pi).\end{aligned}$$

## 3D objects from the perspective of sampling
- Rendering can be decomposed into (a) irregular sampling in the 3D space and (b) 2D projection of the sampled signals.
- The irregular sampling at location $$\boldsymbol{u}\in\mathbb{R}^3$$ can be represented by weighted sum of reconstruction kernel $$r_k(\cdot)$$ on the integer set $$\mathbb{IN}$$:

$$f_c(\boldsymbol{u})=\sum_{k\in\mathbb{IN}}w_kr_k(\boldsymbol{u}),$$

- Projection operator $$\mathcal{P}$$ is further applied to $$f_c(\boldsymbol{u})$$, yielding 2D signals $$g_c(\boldsymbol{x})$$ at location $$\boldsymbol{x}\in\mathbb{R}^2$$:

$$g_c(\boldsymbol{x})=\{\mathcal{P}(f_c)\}(\boldsymbol{x})=\sum_{k\in\mathbb{IN}}w_k\mathcal{P}(r_k(\boldsymbol{u}))=\sum_{k\in\mathbb{IN}}w_kp_k(\boldsymbol{u}).$$

- Band-limit filter $$h(\boldsymbol{x})$$ is used to filter$$g_c(\boldsymbol{x})$$:

$$\hat{g}_c(\boldsymbol{x})=g_c(\boldsymbol{x})\ast h(\boldsymbol{x})=\sum_{k\in\mathbb{IN}}w_k\int_{\mathbb{R}}p_k(\boldsymbol{\eta})h(\boldsymbol{x}-\boldsymbol{\eta})\text{d}\boldsymbol{\eta}=\sum_{k\in\mathbb{IN}}w_k\left[(p_k\ast h)(\boldsymbol{x})\right]=\sum_{k\in\mathbb{IN}}w_k\rho_k(\boldsymbol{x}).$$

- Operation to the sampled scene is equal to the operation of the reconstruction kernel $$\rho_k(\cdot)$$.