# [PBRT] Understanding the Differential Equation for Absorption Cross Section in Volume Scattering

1. [There is something confusing in this equation](#intro)
2. [(Background) First order linear differential equation and solution](#diffeq)
3. [(Background) (Background) Summary of relavant variables in Volume Scattering](#volume_scattering)
    1. [Radiance](#radiance)
    2. [Absorption cross section](#absorption_cs)
4. [Solution doesn't fit the differential equation!](#doesnt_fit)
    1. [Assumptions that can make the solution fit](#modify_eq)
    2. [Assumption - Continuity of Radiance](#cont_radiance)
5. [Re-formulating the differential equation](#reformulate)
    1. [Variable definition](#var_def)
    2. [Differential equation formulation](#diff_eq_form)
    3. [The solution](#final_sol)

### 1. There is something confusing in this equation <a name="intro"></a>

I was reading about volume scattering in Physically Based Rendering (aka [PBRT](https://pbrt.org/)) when I encountered this differential equation:

$$d L_o(p, \omega) = -\sigma_a(p, \omega) L_i(p, -\omega) dt \ \ \ \tag{1}$$

with the solution given as

$$\large e^{-\int_{0}^{d} \sigma_a(p+t \omega, \omega) dt} \ \ \ \tag{2}$$

However it seemed unclear how the bottom formula (2) could be a solution of the top differential equation (1). 

In fact, there is no explanation on which function the formula (1) corresponds to, or where this should be substituted into. Assuming it is either of $L_o(p, \omega)$ or $L_i(p, -\omega)$ doesn't lead to equation (1), which made me question what this formula is meant to be.

After figuring out what I have misunderstood, the whole thing looked pretty straightforward. Nevertheless I struggled with this for a whole day so I am writing down how I understood this solution.

Jump to the bottom for conclusion: [5. Re-formulating the differential equation](#reformulate)

### 2. (Background) First order linear differential equation and solution <a name="diffeq"></a>

The solution to first-order linear differential equation is well-known. Given an equation of the form

$$ y' = f(x)y$$

the solution is 

$$ y = e^{\ \int f(x) dx}$$

easily verified using substitution.

### 3. (Background) Summary of relavant variables in Volume Scattering <a name="volume_scattering"></a>

The differential equation (1) explains the absorption of radiance crossing a mass of particles in a probabilistic way. 

#### Radiance <a name="radiance"></a>

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

$$d L_o(p, \omega) = -\sigma_a(p, \omega) L_i(p, -\omega) dt \ \ \ \tag{1}$$

#### Absorption cross section <a name="absorption_cs"></a>

Absorption cross section, $\sigma_a$, is the probability density function of light being absorbed per unit distance travelled. The general idea and formulation of Absorption is describe here ([Absorption - PBRT](https://pbr-book.org/3ed-2018/Volume_Scattering/Volume_Scattering_Processes#Absorption))

<p align="center">
<img src="https://pbr-book.org/3ed-2018/Volume_Scattering/Volume%20absorption.svg" width="700">
</p>

*(Image: https://pbr-book.org/3ed-2018/Volume_Scattering/Volume_Scattering_Processes#Absorption)*

The figure describes absorption in a differential cylinder (or, a hypothetical cylinder with a very short length that is filled with particles). Radiance enters the cylinder from the left (Li works in the opposite direction of the arrow -omega) and leaves on the right.

PBRT describes this process as

$$L_o(p, \omega) - L_i(p, -\omega) = d L_o(p, \omega) = -\sigma_a(p, \omega) L_i(p, -\omega) dt$$

which the solution is given as

$$\large e^{-\int_{0}^{d} \sigma_a(p+t \omega, \omega) dt} \ \ \ \tag{2}$$

## 4. Solution doesn't fit the differential equation! <a name="doesnt_fit"></a>

Recall the problem: The differential equation was given as

$$d L_o(p, \omega) = -\sigma_a(p, \omega) L_i(p, -\omega) dt \ \ \ \tag{1}$$

and the 'solution' provided in PBRT was

> If we assume that the ray travels a distance d in direction w through the medium starting at point p, the remaining portion of the original radiance is given by

$$\large e^{-\int_{0}^{d} \omega_a(p+t \omega, \omega) dt} \ \ \ \tag{2}$$

For convenience let's call this A. Differentiating this solution gives

$$ A = e^{-\int_{0}^{d} \omega_a(p+t \omega, \omega) dt}\ \ \ \tag{2}$$

$$ \frac{dA}{dt} = -\sigma_a(p+d \omega, \omega) \cdot \large e^{-\int_{0}^{d} \sigma_a(p+t \omega, \omega) dt} = -\sigma_a(p+d \omega, \omega) \cdot A \ \ \ \tag{3}$$

(Note that the integral in the exponent cancels out by fundamental theorem of calculus, and variable of integration, t, gets replaced by d)

There are certain points we can compare (3) with equation (1). The left hand side of (1) has $L_o(p, \omega)$ being differentiated, while the left hand side of (2) has $A$ being differentiated. Usually we would expect $A = L_o(p, \omega)$. However on the right hand side of (1) there is $L_i(p, -\omega)$ instead of $L_o(p, \omega)$, and more importantly, the absorption cross section ($\sigma_a$) is evaluated on point $p+d \omega$ instead of point $p$.

### 4.1. Assumptions that can make the solution fit <a name="modify_eq"></a>

As we got the term $\sigma_a(p+d \omega, \omega)$ evaluated at point $p+d \omega$, we could assume two things to make the equation consistent:

1) $L_o(p+d \omega, \omega) = L_i(p + d \omega, -\omega)$ (Radiance continuous on point $p+d \omega$)
2) $A = L_o(p+d \omega, \omega) = L_i(p + d \omega, -\omega)$

and try to evaluate the differential equation on point $p+d \omega$.

Then the derivative (3) with the assumptions above becomes

$$ \frac{d L_o(p+d \omega, \omega)}{dt} = \frac{d L_i(p + d \omega, -\omega)}{dt} = -\sigma_a(p+d \omega, \omega) \cdot \large e^{-\int_{0}^{d} \sigma_a(p+t \omega, \omega) dt} = -\sigma_a(p+d \omega, \omega) \cdot L_i(p + d \omega, -\omega)$$

consistent with (1), as we needed.

### 4.2. Assumption - Continuity of Radiance <a name="cont_radiance"></a>

The first assumption raises the question: Is radiance (L) continuous on point $p+d \omega$?

I do not know if this was mentioned in PBRT, I just learned about radiance 2 days ago and haven't read most of the book. So please don't trust me on this.

I was unsure of this because:
- The section on Radiometry states that radiance is continuous if and only if the point is a free space that is not a surface boundary. Since we are in context of volume scattering, this may be the case (no boundary on point $p+d \omega$) or may not be the case (scattering counts as a probabilistic collision).

I decided that it is continuous because:
- [Wikipedia page on Absorption Cross Section](https://en.wikipedia.org/wiki/Absorption_cross_section) formulates the process as
$$\frac{dN}{dx} = -Nn \sigma$$
    where N: number of photons, n: absorbing molecules per volume, $\sigma$: absorption cross section. Grouping n and $\sigma$ as one thing gives a similar form as equation (1) where $L_o$ and $L_i$ are not distinguished.
- [Wikipedia page on Beer–Lambert_law](https://en.wikipedia.org/wiki/Beer%E2%80%93Lambert_law) formulates attenuation as
$$\frac{d \Phi_e(z)}{dx} = -\mu(z)\Phi_e(z)$$
    where \Phi_e: entering radiant flux, z: direction, and $\mu$: Napierian attenuation coefficient. Considering that radiance is the radiant flux per unit area per unit solid angle, we could see this equation as a similar format of (1), where $\sigma_a$ is replaced by $\mu$, and $L_o$ and $L_i$ are not distinguished.

Seeing similar formats of the given differential equation I decided I could consider the radiance to be continuous on the given point $p+d\omega$, unless it is a surface boundary (which is off the topic of volume scattering).

## 5. Re-formulating the differential equation <a name="reformulate"></a>

While observing the differential equation, I realized that PBRT was describing the differential equation in a simplified way, omiiting the details that lead to the solution. So I tried to come up with a detailed explanation of the situation and the differential equation.

### 5.1. Variable definition <a name="var_def"></a>

Let's define some variables.

#### 1) Point $p$, direction $\omega$, distance $d$

We have 2 points of interest: $p$ and $p+d\omega$. The initial radiance starts from point $p$ with direction $\omega$, continuously decreasing through absorption. We want to figure out how much radiance is after distance $d$.

#### 2) Radiance

Incident radiance ($L_i$) and exitance radiance ($L_o$) are as defined above.

We are assuming the radiance to be continuous on all points (from above: [4.2. Assumption - Continuity of Radiance](#cont_radiance)) so we could temporarily replace both the incident radiance ($L_i$) and exitance radiance ($L_o$) by radiance ($L$), where the direction $\omega$ is negated only for incident radiances.

The radiance at point $p$, or the initial radiance, can be formulated as $L(p, \omega)$, and it is known to us.

The radiance at point $p+d\omega$ is what we want to figure out.

### 5.2. Differential equation formulation <a name="diff_eq_form"></a>

We formulate the differential equation using a differential cylinder on point $p+dt$.

<p align="center">
<img src="https://pbr-book.org/3ed-2018/Volume_Scattering/Volume%20absorption.svg" width="700">
</p>

*(Image: https://pbr-book.org/3ed-2018/Volume_Scattering/Volume_Scattering_Processes#Absorption)*

Let's let the length of this cylinder to be $t$. We will take a limit of t to zero.

Then the incoming radiance is going into direction $\omega$ on point $p+d\omega$, whereas the leaving radiance is leaving for direction $\omega$ on point $p+d\omega+t\omega$. Replacing the two functions makes the difference in radiance to be

$$L(p+d\omega+t\omega, \omega) - L(p+d\omega, \omega)$$

As provided in PBRT, we take the difference in radiance to be a linear function of its initial radiance and $\sigma_a$ evaluated at the point. We formulate this by

$$L(p+d\omega+t\omega, \omega) - L(p+d\omega, \omega) = -\sigma_a(p+d\omega, \omega)L(p+d\omega, \omega) \cdot t$$

To get the difference of radiance per unit distance, we take the limit of t to zero

$$\lim_{t \to 0} \frac{L(p+d\omega+t\omega, \omega) - L(p+d\omega, \omega)}{t} = \frac{dL(p+d\omega, \omega)}{dt}$$

$$= \lim_{t \to 0} \frac{-\sigma_a(p+d\omega, \omega)L(p+d\omega, \omega) \cdot t}{t} \ \ \ \ \ \text{(Substitute the difference of radiance)}$$

$$= \lim_{t \to 0} -\sigma_a(p+d\omega+t\omega, \omega)L(p+d\omega, \omega) \ \ \ \ \ \text{(t cancels out)}$$

$$= -\sigma_a(p+d\omega, \omega)L(p+d\omega, \omega) \ \ \ \ \ \text{(t becomes zero)}$$

$$= \frac{dL(p+d\omega, \omega)}{dt} \ \ \ \ \ \text{(by definition of derivative)}$$

Hence we obtain

$$dL(p+d\omega, \omega) = -\sigma_a(p+d\omega, \omega)L(p+d\omega, \omega) dt$$

On the initial point, distance $d$ is zero, and the result matches the provided equation (1).

### 5.3. The solution <a name="final_sol"></a>

And we're done! The provided solution

$$\large e^{-\int_{0}^{d} \sigma_a(p+t \omega, \omega) dt}$$

works nicely with this result, as shown in [4.1. Assumptions that can make the solution fit](#modify_eq).


