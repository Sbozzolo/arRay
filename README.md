<p align="center">
<img src="https://github.com/Sbozzolo/arRay/raw/main/logo.png" height="120">
</p>

`arRay` will be a General-Relativistic Ray-Tracing code in Dyalog APL.

## Why?

[APL](https://en.wikipedia.org/wiki/APL_(programming_language)) (A Programming
Language) is an array programming language. This is a completely different (and
for the most part obscure) paradigm compared to most other languages out there.
APL shines in producing terse and efficient code when used for manipulations of
matrices. I think that problem of General-Relativistic Ray-Tracing can be cast
in these terms, so I wanted to see what it could come out.

Note: I started learning APL last in December 2022, so this code will likely be
terrible.

## General-Relativistic Ray-Tracing

According to General Relativity, light does not travel on straight lines and its
motion is influenced by massive bodies (such as [our
Sun](https://en.wikipedia.org/wiki/Solar_eclipse_of_May_29,_1919)). Light rays
travel on __null geodesics__, which are special curves in spacetime.
General-relativistic ray-tracing consists in finding such curves given the
initial position of the camera.

#### In equations

Let $x^\mu(\lambda)$ be the four-position of a photon with $\lambda$ affine
parameter. We define its momentum as

$$ k^\mu = \frac{d x^\mu}{d \lambda}.$$

$k^\mu$ is also function of $\lambda$. Given that we are describing a null
geodesic, $k^\mu$ has to be null, so $k^\mu k_\mu = 0$. Performing ray-tracing
means solving the geodesic equation:

$$ \frac{d k^\mu}{d \lambda} = -\Gamma^\mu_{\alpha\beta} k^\alpha k^\beta . $$

$\Gamma^\mu_{\alpha\beta}$ are the Christoffel symbols and depend on the
spacetime and the four-position in the spacetime. The geodesic equation is a set
of ordinary differential equations (ODEs) in $\lambda$, which have to be coupled
with the definition of the four-momentum to find the position of the photon. So,
we have to solve eight ODEs (four of which are trivial). We represent each
photon with a 8-element vector, where the first four element is $x^\mu$, and the
last 4 are $k^\mu$. Then, starting from some initial data on our camera, we
solve the ODEs until we reach the black hole or some outer domain.

#### Initial data

First, we need to generate the initial data. Our initial data will consist of a
grid of photons initially on our camera. Then, we integrate the photons backward
in time towards the black hole. Each photon is described by the 8-element state
that contains its four-position and four-momentum.

For the initial data, we need to specify five parameters: `d`, the distance from
the black hole, `i` and `j`, the inclination and azimuthal angle of the camera,
`N` and `F` are the number of pixels and the field-of-view (we assume a square
camera). From these variables, we define a Cartesian coordinate system $(\alpha,
\beta)$ onto the camera.

For a given $(\alpha, \beta)$, the initial four-position will be


#### Evolution

In this first implementation, we solve the ODEs with a forward Euler's method,
which is the simplest way to solve an ODE. With this method, we move from the
$i$-th step $x^\mu(\lambda_i), k^\mu(\lambda_i)$ at $\lambda = \lambda_i$ to the
following at $\lambda_{i+1} = \lambda + \Delta \lambda$ with

$$ x^\mu(\lambda_{i + 1}) = x^\mu(\lambda_{i}) + \Delta \lambda k^\mu(\lambda_{i}), $$

$$ k^\mu(\lambda_{i + 1}) = k^\mu(\lambda_{i}) -\Delta \lambda \Gamma^\mu_{\alpha\beta}(\lambda_{i}) k^\alpha(\lambda_{i}) k^\beta(\lambda_{i}). $$

In this, we defined $ \Gamma^\mu_{\alpha\beta}(\lambda_{i}) =  \Gamma^\mu_{\alpha\beta}(x^\mu(\lambda_{i})) $.

#### Stopping criterion

For each photon, we set two termination conditions: either the photon gets very
close to the horizon, or it goes too far. So, we define two parameters that
control whether to continue the integration or not depending on the spatial
position of the photon with respect to the horizon.
