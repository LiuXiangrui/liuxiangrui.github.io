---
layout: post
title: "Neural Radiance Fields"
excerpt_separator: ""
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
L(\boldsymbol{x}) \approx \sum_{n=1}^{N}\left[\sigma_nC_n\int_{\boldsymbol{x}_{n}}^{\boldsymbol{x}_{n+1}}T(\boldsymbol{t})\text{d}\boldsymbol{t}\right]= \sum_{n=1}^{N} C_nT_n(1-e^{-\sigma_n \delta_n}), \quad\text{with}\;T_n = \exp{\sum_{k=1}^{n-1}\sigma_k\delta_k}.
$$


## Camera Pose and Transform

### World Coordinate System

|  Name   | x-axis | y-axis | z-axis |    Example    |
|:-------:|:------:|:------:|:------:|:-------------:|
|Cartesian|(1,0,0) |(0,1,0) |(0,0,1) |       -       |
|   RDF   |(0,1,0) |(0,0,-1)|(-1,0,0)| OpenCV, Colmap|
|   DRB   |(0,0,-1)|(0,1,0) |(1,0,0) |      LLFF     |
|   RUB   |(0,1,0) |(0,0,1) |(1,0,0) | OpenGL, NeRF  |
|   LUF   |(0,-1,0)|(0,0,1) |(-1,0,0)|   Pytorch3D   |


### Extrinsic Matrix

- Camera extrinsic matrix, aka world-to-camera matrix (w2c), is used to transform a point $$\boldsymbol{p}_w = [x_w,y_w,z_w]^T$$ from world coordinate system to camera coordinate system.

$$
\begin{bmatrix}
x_c \\ y_c \\ z_c \\ 1
\end{bmatrix}=\begin{bmatrix}
\boldsymbol{R} & \boldsymbol{t}\\
\boldsymbol{0} &1
\end{bmatrix}\begin{bmatrix}
x_w \\ y_w \\ z_w \\1
\end{bmatrix}
$$

- Translation vector $$\boldsymbol{t}\in\mathbb{R}^3$$ represents the position of the world coordinate origin in the camera coordinate system.
- Rotation matrix $$\boldsymbol{R}\in\mathbb{R}^{3\times 3}$$ represents the rotation from the world coordinate system to the camera coordinate system.

$$
\boldsymbol{R} = \boldsymbol{R}_x \boldsymbol{R}_y \boldsymbol{R}_z,\quad \text{where}\;
\boldsymbol{R}_x = \begin{bmatrix}1 & 0 & 0\\0 & \cos\alpha & -\sin\alpha\\0 & \sin\alpha & \cos\alpha\end{bmatrix},\quad
\boldsymbol{R}_y = \begin{bmatrix}\cos\beta & 0 & \sin\beta\\0 & 1 & 0\\-\sin\beta & 0 & \cos\beta\end{bmatrix},\quad
\boldsymbol{R}_z = \begin{bmatrix}\cos\gamma & -\sin\gamma & 0\\\sin\gamma & \cos\gamma & 0\\0 & 0 & 1\end{bmatrix},\quad
$$

### Intrinsic Matrix
- The intrinsic matrix $$\boldsymbol{K}\in\mathbb{R}^3$$ describes the transform from camera coordinate system to 2D imaging plane, including focal length $$f_x, f_y$$ and offsets of the image origin relative to the camera optical center $$c_x,c_y$$.

$$
z_c\begin{bmatrix}
        u\\ v \\1
    \end{bmatrix}=\begin{bmatrix}
 f_x&0&c_x\\ 
 0&f_y&c_y\\ 
 0&0&1
\end{bmatrix}\begin{bmatrix}
x_c \\ y_c \\ z_c\end{bmatrix}
$$

- Note that we need to know the depth $$z_c$$ if we want to re-map pixels $$(u,v)$$ to the camera coordinate system. That's why we need depth estimation. 

> [Kyle Simek](https://ksimek.github.io/2012/08/22/extrinsic/) provide a useful visualization of camera pose.

### Perspective Transform
- Only objects within viewing frustum can be projected into 2D imaging plane.
- The viewing frustum is determined by nera clipping plane (2D imaging plane) and far clipping plane (infinity), including the depth $$\{z_{near}, z_{far}\}$$ and right-top corner of the 2D imaging plane $$(r, t)$$
- Perspective matrix describes the transform a 3D point within viewing frustum to the normalized device coordinates (NDC).

$$
\boldsymbol{p}_{p}=\begin{bmatrix}\frac{z_{nera}}{r} & 0 & 0 & 0 \\0 & \frac{z_{nera}}{t} & 0 & 0 \\0 & 0 & -\frac{z_{far}+z_{nera}}{z_{far}-z_{nera}} & -\frac{2z_{far}z_{near}}{z_{far}-z_{near}}\\0 & 0 & -1 & 0\end{bmatrix}
$$