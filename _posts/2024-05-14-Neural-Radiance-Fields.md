---
layout: post
title:  "Neural Radiance Fields"
excerpt_separator: ""
---

<!-- # Background

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

- Operation to the sampled scene is equal to the operation of the reconstruction kernel $$\rho_k(\cdot)$$. -->

---

## Volume Rendering

### Radiative Transfer Equation
The change in radiance when going through a medium is equal to the difference between the incoming and outgoing radiance at point $$\boldsymbol{x}$$ within the medium along the direction $$\vec{\omega}$$.

$$\text{d}L(\boldsymbol{x}, \vec{\omega})=\underbrace{\sigma_a(\boldsymbol{x},\vec{\omega})L_e(\boldsymbol{x}, \vec{\omega})\text{d}z}_{emission} + \underbrace{\sigma_s(\boldsymbol{x},\vec{\omega})L_s(\boldsymbol{x}, \vec{\omega})\text{d}z}_{in-scattering} - \underbrace{\sigma_a(\boldsymbol{x},\vec{\omega})L(\boldsymbol{x}, \vec{\omega})\text{d}z}_{absorption}-\underbrace{\sigma_s(\boldsymbol{x},\vec{\omega})L(\boldsymbol{x}, \vec{\omega})\text{d}z}_{out-scattering}$$

- **Absorption coefficient**: $$\sigma_a(\boldsymbol{x},\vec{\omega})$$ represents the probability density that <u>light is absorbed</u> per unit distance traveled in the medium.
- **Scattering coefficient**: $$\sigma_s(\boldsymbol{x},\vec{\omega})$$ represents the probability that <u>a photon is being scattered<u> by the volume per unit distance
- **Extinction coefficient**: $$\sigma_t(\boldsymbol{x},\vec{\omega})=\sigma_a(\boldsymbol{x},\vec{\omega})+\sigma_s(\boldsymbol{x},\vec{\omega})$$ represents the loss in radiance as the light beam travels along the direction vector $$\vec{\omega}$$.

- The radiance of point $$\boldsymbol{x}$$ within the ray can be calculated by

$$
L(\boldsymbol{x}, \vec{\omega}) = \overbrace{I_0 e^{-\int_{0}^{\boldsymbol{x}}\sigma_t(\boldsymbol{t},\vec{\omega})\text{d}\boldsymbol{t}}}^{attenuation} + \underbrace{\int_{0}^{\boldsymbol{x}}e^{-\int_{0}^{\boldsymbol{t}}\sigma_t(\boldsymbol{u},\vec{\omega})\text{d}\boldsymbol{u}}\left[\sigma_a(\boldsymbol{t},\vec{\omega})L_e(\boldsymbol{t},\vec{\omega})+\sigma_s(\boldsymbol{t},\vec{\omega})L_s(\boldsymbol{t},\vec{\omega})\right]\text{d}\boldsymbol{t}}_{enhancement}
$$

### Rendering Equation in NeRF

- Assume the medium is **homogenous**, both the three coeffients are irrelevant to the direction $$\vec{\omega}$$:

$$\sigma_a(\boldsymbol{x}, \vec{\omega}) = \sigma_a(\boldsymbol{x}),\;\sigma_s(\boldsymbol{x}, \vec{\omega}) = \sigma_s(\boldsymbol{x}),\;\sigma_t(\boldsymbol{x}, \vec{\omega}) = \sigma_t(\boldsymbol{x})$$

- Denote $$T(\boldsymbol{x})=e^{-\int_{0}^{\boldsymbol{x}}\sigma_t(\boldsymbol{t})\text{d}\boldsymbol{t}},\;C(\boldsymbol{t})=\frac{\sigma_a(\boldsymbol{t})L_e(\boldsymbol{t})+\sigma_s(\boldsymbol{t})L_s(\boldsymbol{t})}{ \sigma_t(\boldsymbol{x})}$$, and ignore the background light item, the raidiance of point $$\boldsymbol{x}$$ can be formulated as:

$$
L(\boldsymbol{x}) = \cancel{I_0T(\boldsymbol{x})}+\int_{0}^{\boldsymbol{x}}T(\boldsymbol{t})\sigma_t(\boldsymbol{t})C(\boldsymbol{t})\text{d}\boldsymbol{t}.
$$

- The discrete approximation of $$L(\boldsymbol{x})$$ is calculated by first dividing interval $$[0,\boldsymbol{x}]$$ into $$N$$ spaced intervals and then summing $$\Delta L$$ over these intervals:

$$
L(\boldsymbol{x}) \approx I(0) + \sum_{n=1}^{N}\left[L(\boldsymbol{x}_{n+1})-L(\boldsymbol{x}_{n})\right].$$

- $$C(\boldsymbol{t})$$ and $$\sigma_t(\boldsymbol{x})$$ can be treated as constants within a small interval $$[\boldsymbol{x}_n, \boldsymbol{x}_{n+1}]$$, thereby

$$
L(\boldsymbol{x}) \approx I(0) + \sum_{n=1}^{N}\left[L(\boldsymbol{x}_{n+1})-L(\boldsymbol{x}_{n})\right] = I(0) + \sum_{n=1}^{N}\sigma_nC_n\int_{\boldsymbol{x}_{n}}^{\boldsymbol{x}_{n+1}}T(\boldsymbol{t})\text{d}\boldsymbol{t}.
$$

- Observing that $$T(\boldsymbol{x})=e^{-\int_{0}^{\boldsymbol{x}}\sigma_t(\boldsymbol{t})\text{d}\boldsymbol{t}}=e^{-\int_{0}^{\boldsymbol{x}_n}\sigma_t(\boldsymbol{t})\text{d}\boldsymbol{t}}e^{-\int_{\boldsymbol{x}_{n}}^{\boldsymbol{x}}\sigma_t(\boldsymbol{t})\text{d}\boldsymbol{t}}$$, we have 

$$
\begin{aligned}
\int_{\boldsymbol{x}_{n}}^{\boldsymbol{x}_{n+1}}T(\boldsymbol{t})\text{d}\boldsymbol{t}&=\int_{\boldsymbol{x}_{n}}^{\boldsymbol{x}_{n+1}}e^{-\int_{0}^{\boldsymbol{x}_{n}}\sigma_t(\boldsymbol{t})\text{d}\boldsymbol{t}}e^{-\int_{\boldsymbol{x}_{n}}^{\boldsymbol{t}}\sigma_t(\boldsymbol{t})\text{d}\boldsymbol{t}}\text{d}\boldsymbol{t}\\
&=\int_{\boldsymbol{x}_{n}}^{\boldsymbol{x}_{n+1}}T(\boldsymbol{x}_{n})e^{-\int_{\boldsymbol{x}_{n}}^{\boldsymbol{t}}\sigma_n\text{d}\boldsymbol{t}}\text{d}\boldsymbol{t} = T(\boldsymbol{x}_{n})\int_{\boldsymbol{x}_{n}}^{\boldsymbol{x}_{n+1}}e^{\sigma_n (\boldsymbol{t}-\boldsymbol{x}_{n})}\text{d}\boldsymbol{t}\\
&=T(\boldsymbol{x}_{n})e^{\sigma_n \boldsymbol{x}_{n}}\int_{\boldsymbol{x}_{n}}^{\boldsymbol{x}_{n+1}}e^{-\sigma_n \boldsymbol{x}_{n}}\text{d}\boldsymbol{t} = \frac{1}{-\sigma_n}T(\boldsymbol{x}_{n})e^{\sigma_n \boldsymbol{x}_{n}}\left(e^{-\sigma_n \boldsymbol{x}_{n+1}}-e^{-\sigma_n \boldsymbol{x}_{n}}\right)\\
&=\frac{1}{\sigma_n}T(\boldsymbol{x}_{n})\left(1 - e^{-\sigma_n (\boldsymbol{x}_{n+1}-\boldsymbol{x}_{n})}\right)
\end{aligned}
$$

- Denoting sampling step as $$\delta_n = \boldsymbol{x}_{n+1} - \boldsymbol{x}_{n}$$, we can rewrite the transmittance as

$$T(\boldsymbol{x}_n) \approx \exp{\sum_{k=1}^{n-1}\sigma_k\delta_k} = T_n.$$

- Ignoring the background light item $$I(0)$$, we have the rendering equations used by NeRF:

$$
L(\boldsymbol{x}) \approx \sum_{n=1}^{N}\sigma_nC_n\int_{\boldsymbol{x}_{n}}^{\boldsymbol{x}_{n+1}}T(\boldsymbol{t})\text{d}\boldsymbol{t}= \sum_{n=1}^{N} C_nT_n(1-e^{-\sigma_n \delta_n}), \quad\text{with}\;T_n = \exp{\sum_{k=1}^{n-1}\sigma_k\delta_k}.
$$

<!-- 
### Beer-Lamber Law
- The transmittance is defined as the fraction of light that is transmitted through the volume:

$$T(\boldsymbol{x}_0, \boldsymbol{x_0}+\omega d)=\frac{T_{out}}{T_{in}} = \exp\left(-\int_{z=0}^{d}\sigma_a(\boldsymbol{x}_0+\omega z)\text{d}z\right).$$

- If we only consider absortion, the radiative transfer equation can be formulated as

$$L(\boldsymbol{x}_0 + \omega d, \omega)=T(\boldsymbol{x}_0, \boldsymbol{x_0}+\omega d) L(\boldsymbol{x}_0, \omega)$$

- If we consider emission and absortion, the radiative transfer equation can be formulated as

$$
L(\boldsymbol{x}_0 + \omega d, \omega)=T(\boldsymbol{x}_0, \boldsymbol{x_0}+\omega d) L(\boldsymbol{x}_0, \omega) + \int_{z=0}^{d}T(\boldsymbol{x}_0, \boldsymbol{x_0}+\omega z)\sigma_a(\boldsymbol{x_0}+\omega z)L_e(\boldsymbol{x_0}+\omega z, \omega)\text{d}z
$$

- If the medium is homogenous emitting, $$\sigma_a(\boldsymbol{x})\equiv\sigma$$ and $$L_e(\boldsymbol{x_0}+\omega z, \omega)\equiv L_e(\boldsymbol{x_0}, \omega)$$. Hence, we have $$T(\boldsymbol{x}_0, \boldsymbol{x_0}+\omega d)=\exp\left(-d\sigma\right)$$ and

$$
\begin{aligned}
L(\boldsymbol{x}_0 + \omega d, \omega)&W=T(\boldsymbol{x}_0, \boldsymbol{x_0}+\omega d) L(\boldsymbol{x}_0, \omega) + \int_{z=0}^{d}T(\boldsymbol{x}_0, \boldsymbol{x_0}+\omega z)\sigma_a(\boldsymbol{x_0}+\omega z)L_e(\boldsymbol{x_0}+\omega z, \omega)\text{d}z\\
&=\exp\left(-d\sigma\right)L(\boldsymbol{x}_0, \omega) + \sigma L_e(\boldsymbol{x_0}, \omega)\int_{z=0}^{d}\exp\left(-z\sigma\right)\text{d}z\\
&=\exp\left(-d\sigma\right)L(\boldsymbol{x}_0, \omega) + \sigma L_e(\boldsymbol{x_0}, \omega)\left(1-\exp\left(-d\sigma\right)\right)
\end{aligned}
$$

### Ray Matching
- The change of raidance when going through a volume from $$\boldsymbol{x}_0$$ to $$\boldsymbol{x}_1$$ can be formulated as

$$
L(x, \omega) = \int_{\boldsymbol{x}_0}^{\boldsymbol{x}}T(t, x_0)L(x_0, \omega)=\sum
$$


$$\int_{\boldsymbol{x}_0}^{\boldsymbol{x}_1}L(\boldsymbol{x}, \omega)\text{d}\boldsymbol{x}=\sum_{i=1}^{N}L(x_{i-1}+wd_i,\omega)$$ -->