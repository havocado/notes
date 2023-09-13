## Understanding Absorption cross section

I was reading about volume scattering in Physically Based Rendering (aka [PBRT](https://pbrt.org/)) when I encountered this differential equation:

$$ d L_o(p, \omega) = -\sigma_a(p, \omega) L_i(p, -\omega) dt \ \ \ \tag{1}$$

with the solution given as

$$e^{-\int_{0}^{d} \omega_a(p+t \omega, \omega) dt} \ \ \ \tag{2}$$

However it seemed unclear how (2) could work as the solution of the differential equation (1). 

In fact, there is no explanation on which function (1) corresponds to within the differential equation, and substituting (2) to either of $L_o(p, \omega)$ or $L_i(p, -\omega)$ doesn't fit the equation (1), which made me question what this solution was meant for.

### (Background) First order linear differential equation and solution

The solution to first-order linear differential equation is well-known. Given an equation of the form

$$ y' = f(x)y$$

the solution is 

$$ y = e^{\ \int f(x) dx}$$

easily verified using substitution.

### (Background) Summary of relavant variables in Volume Scattering

The differential equation (1) explains the absorption of radiance crossing a mass of particles in a probabilistic way. 

#### Radiance

Radiance is defined using Energy, Radiant Flux (Power), Irradiance and Radiant Exitance (explained well here in [Radiometry - PBRT](https://pbr-book.org/3ed-2018/Color_and_Radiometry/Radiometry))

Flux is defined as a total amount of light energy passing through a volume per unit time, formulated as

$$\Phi = \lim_{\Delta t \to 0} \frac{\Delta Q}{\Delta t} = \frac{dQ}{dt} \ \ \[joules / second = watts \]$$

Irradiance (E)/Radiant Exitance(M) is defined as flux per unit area, where irradiance measures arriving light and radiant exitance measures leaving light. They are formulated as

$$E(p) = \lim_{\Delta A \to 0} \frac{\Delta \Phi(p)}{\Delta A} = \frac{d \Phi(p)}{dA} \ \ \[watts/m^2 \]$$

Radiance is irradiance/radiant exitance per direction, or "solid angles". 

Solid angle is a generalization of angles in 3 dimensions. In 2D, angles can be parameterized by the length of the arc in the unit circle (called radians). In 3D, angles can be parameterized by area on the unit sphere, and they are called steradians.

Here we consider irradiance/radiant exitance using area A perpendicular to the direction omega (denoted E_omega), and differentiate by direction.

$$L(p, \omega) = \lim_{\omega \to 0} \frac{\Delta E_\omega(p)}{\Delta \omega} = \frac{d E_\omega(p)}{d \omega}$$

So radiance is the flux density per unit area per unit solid angle, and can be formulated as

$$L = \frac{d \Phi}{d \omega \ d A^ㅗ}$$

Since Radiance can be discontinuous on points (especially on surface boundaries) we define Incident and Exitance Randiance using limits on opposite directions:

$$L^+(p, \omega) = \lim_{t \to 0^+} L(p+t n_p, \omega)$$

$$L^-(p, \omega) = \lim_{t \to 0^-} L(p+t n_p, \omega)$$

where $n_p$ is the surface normal at point p.

And for convenience we define the Incident radiance function (Li(p, w)) and exitant radiance function (Lo(p, w)) as

$$L_i(p, \omega) =
\begin{cases} L^+(p, -\omega), \ \ \omega \cdot n_p > 0 \\
L^-(p, -\omega), \ \ \omega \cdot n_p < 0
\end{cases}$$

$$L_o(p, \omega) =
\begin{cases} L^+(p, \omega), \ \ \omega \cdot n_p > 0 \\
L^-(p, \omega), \ \ \omega \cdot n_p < 0
\end{cases}$$

defines the two functions in (1):

$$(Recall) \ \ d L_o(p, \omega) = -\sigma_a(p, \omega) L_i(p, -\omega) dt \ \ \ \tag{1}$$


##### Absorption cross section

todo
