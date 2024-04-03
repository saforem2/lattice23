# MLMC: Machine Learning Monte Carlo
Sam Foreman
2024-04-03

# 

<div style="background-color: rgba(22,22,22,0.75); border-radius: 10px; text-align:center; padding: 0px; padding-left: 1.5em; padding-right: 1.5em; max-width: min-content; min-width: max-content; margin-left: auto; margin-right: auto; padding-top: 0.2em; padding-bottom: 0.2em; line-height: 1.5em!important;">

<span style="color:#939393; font-size:1.5em; font-weight: bold;">MLMC:
Machine Learning Monte Carlo</span>  
<span style="color:#777777; font-size:1.2em; font-weight: bold;">for
Lattice Gauge Theory</span>  
<span style="padding-bottom: 0.5rem;"><br> </span>  
[](https://samforeman.me) Sam Foreman  
<span class="dim-text" style="font-size:0.8em;">Xiao-Yong Jin, James C.
Osborn</span>  
<span style="font-size:0.8em;"><span style="border-bottom: 0.5px solid #00ccff;">[
`saforem2/`](https://github.com/saforem2/)</span>`{`<span style="border-bottom: 0.5px solid #00ccff;">[`lattice23`](https://github.com/saforem2/lattice23)</span>,
<span style="border-bottom: 0.5px solid #00ccff;">[`l2hmc-qcd`](https://github.com/saforem2/l2hmc-qcd)</span>`}`</span>

</div>

<div class="footer">

<span class="dim-text" style="&quot;text-align:left;'">2023-07-31 @
[Lattice
2023](https://indico.fnal.gov/event/57249/contributions/271305/)</span>

</div>

# Overview

1.  [Background:
    `{MCMC,HMC}`](#markov-chain-monte-carlo-mcmc-centeredslide)
    - [Leapfrog Integrator](#leapfrog-integrator-hmc-centeredslide)
    - [Issues with HMC](#sec-issues-with-hmc)
    - [Can we do better?](#sec-can-we-do-better)
2.  [L2HMC: Generalizing MD](#sec-l2hmc)
    - [4D $SU(3)$ Model](#sec-su3)
    - [Results](#sec-results)
3.  [References](#sec-references)
4.  [Extras](#sec-extras)

# Markov Chain Monte Carlo (MCMC)

<div class="columns">

<div class="column" width="50%">

> [!NOTE]
>
> ### Goal
>
> Generate **independent** samples $\{x_{i}\}$, such that[^1]
> $$\{x_{i}\} \sim p(x) \propto e^{-S(x)}$$ where $S(x)$ is the *action*
> (or potential energy)

- Want to calculate observables $\mathcal{O}$:  
  $\left\langle \mathcal{O}\right\rangle \propto \int \left[\mathcal{D}x\right]\hspace{4pt} {\mathcal{O}(x)\, p(x)}$

</div>

<div class="column" width="49%">

![](https://raw.githubusercontent.com/saforem2/deep-fridays/main/assets/normal_distribution.dark.svg)

</div>

</div>

If these were <span style="color:#00CCFF;">independent</span>, we could
approximate:
$\left\langle\mathcal{O}\right\rangle \simeq \frac{1}{N}\sum^{N}_{n=1}\mathcal{O}(x_{n})$  
$$\sigma_{\mathcal{O}}^{2} = \frac{1}{N}\mathrm{Var}{\left[\mathcal{O} (x)
  \right]}\Longrightarrow \sigma_{\mathcal{O}} \propto \frac{1}{\sqrt{N}}$$

<div class="footer">

[ `saforem2/lattice23`](https://saforem2.github.io/lattice23)

</div>

# Markov Chain Monte Carlo (MCMC)

<div class="columns">

<div class="column" width="50%">

> [!NOTE]
>
> ### Goal
>
> Generate **independent** samples $\{x_{i}\}$, such that[^2]
> $$\{x_{i}\} \sim p(x) \propto e^{-S(x)}$$ where $S(x)$ is the *action*
> (or potential energy)

- Want to calculate observables $\mathcal{O}$:  
  $\left\langle \mathcal{O}\right\rangle \propto \int \left[\mathcal{D}x\right]\hspace{4pt} {\mathcal{O}(x)\, p(x)}$

</div>

<div class="column" width="49%">

![](https://raw.githubusercontent.com/saforem2/deep-fridays/main/assets/normal_distribution.dark.svg)

</div>

</div>

Instead, nearby configs are <span class="red-text">correlated</span>,
and we incur a factor of
$\textcolor{#FF5252}{\tau^{\mathcal{O}}_{\mathrm{int}}}$:
$$\sigma_{\mathcal{O}}^{2} =
  \frac{\textcolor{#FF5252}{\tau^{\mathcal{O}}_{\mathrm{int}}}}{N}\mathrm{Var}{\left[\mathcal{O}
  (x) \right]}$$

<div class="footer">

[ `saforem2/lattice23`](https://github.com/saforem2/lattice23)

</div>

# Hamiltonian Monte Carlo (HMC)

- Want to (sequentially) construct a chain of states:
  $$x_{0} \rightarrow x_{1} \rightarrow x_{i} \rightarrow \cdots \rightarrow x_{N}\hspace{10pt}$$

  such that, as $N \rightarrow \infty$:
  $$\left\{x_{i}, x_{i+1}, x_{i+2}, \cdots, x_{N} \right\} \xrightarrow[]{N\rightarrow\infty} p(x)
  \propto e^{-S(x)}$$

> [!TIP]
>
> ### Trick
>
> - Introduce <span class="green-text">fictitious</span> momentum
>   $v \sim \mathcal{N}(0, \mathbb{1})$
>   - Normally distributed **independent** of $x$, i.e. $$\begin{align*}
>     p(x, v) &\textcolor{#02b875}{=} p(x)\,p(v) \propto e^{-S{(x)}} e^{-\frac{1}{2} v^{T}v}
>     = e^{-\left[S(x) + \frac{1}{2} v^{T}{v}\right]}
>     \textcolor{#02b875}{=} e^{-H(x, v)}
>     \end{align*}$$

## Hamiltonian Monte Carlo (HMC)

<div class="columns">

<div class="column" width="55%">

- <span class="green-text">**Idea**</span>: Evolve the
  $(\dot{x}, \dot{v})$ system to get new states $\{x_{i}\}$❗

- Write the **joint distribution** $p(x, v)$: $$
  p(x, v) \propto e^{-S[x]} e^{-\frac{1}{2}v^{T} v} = e^{-H(x, v)}
  $$

</div>

<div class="column" width="45%">

> [!TIP]
>
> ### Hamiltonian Dynamics
>
> $H = S[x] + \frac{1}{2} v^{T} v \Longrightarrow$
> $$\dot{x} = +\partial_{v} H,
> \,\,\dot{v} = -\partial_{x} H$$

</div>

</div>

<img
src="https://raw.githubusercontent.com/saforem2/deep-fridays/main/assets/hmc1.svg"
class="r-stretch" />

## Leapfrog Integrator (HMC)

<div class="columns" style="text-align:center; font-size: 0.9em;"
height="100%">

<div class="column" width="48%">

> [!TIP]
>
> ### Hamiltonian Dynamics
>
> $\left(\dot{x}, \dot{v}\right) = \left(\partial_{v} H, -\partial_{x} H\right)$

> [!NOTE]
>
> ### Leapfrog Step
>
> `input` $\,\left(x, v\right) \rightarrow \left(x', v'\right)\,$
> `output`
>
> $$\begin{align*}
> \tilde{v} &:= \textcolor{#F06292}{\Gamma}(x, v)\hspace{2.2pt} = v - \frac{\varepsilon}{2} \partial_{x} S(x) \\
> x' &:= \textcolor{#FD971F}{\Lambda}(x, \tilde{v}) \, =  x + \varepsilon \, \tilde{v} \\
> v' &:= \textcolor{#F06292}{\Gamma}(x', \tilde{v}) = \tilde{v} - \frac{\varepsilon}{2} \partial_{x} S(x')
> \end{align*}$$

> [!WARNING]
>
> ### Warning!
>
> - Resample $v_{0} \sim \mathcal{N}(0, \mathbb{1})$  
>   at the <span class="yellow-text">beginning</span> of each trajectory

<div style="font-size:0.8em; margin-left:13%;">

<span class="dim-text">**Note**: $\partial_{x} S(x)$ is the
*force*</span>

</div>

</div>

<div class="column" width="49%"
style="text-align:left; margin-left:2%;">

<img src="./assets/hmc1/hmc-update-light.svg" style="width:60.0%" />

</div>

</div>

## HMC Update

<div class="columns">

<div class="column" width="65%">

- We build a trajectory of $N_{\mathrm{LF}}$ **leapfrog steps**[^3]
  $$\begin{equation*}
  (x_{0}, v_{0})%
  \rightarrow (x_{1}, v_{1})\rightarrow \cdots%
  \rightarrow (x', v')
  \end{equation*}$$

- And propose $x'$ as the next state in our chain

$$\begin{align*}
  \textcolor{#F06292}{\Gamma}: (x, v) \textcolor{#F06292}{\rightarrow} v' &:= v - \frac{\varepsilon}{2} \partial_{x} S(x) \\
  \textcolor{#FD971F}{\Lambda}: (x, v) \textcolor{#FD971F}{\rightarrow} x' &:= x + \varepsilon v
\end{align*}$$

- We then accept / reject $x'$ using Metropolis-Hastings criteria,  
  $A(x'|x) = \min\left\{1, \frac{p(x')}{p(x)}\left|\frac{\partial x'}{\partial x}\right|\right\}$

</div>

<div class="column" width="30%">

![](./assets/hmc1/hmc-update-light.svg)

</div>

</div>

## HMC Demo

<iframe data-src="https://chi-feng.github.io/mcmc-demo/app.html" width="90%" height="500" title="l2hmc-qcd">
</iframe>

# Issues with HMC

- What do we want in a good sampler?
  - **Fast mixing** (small autocorrelations)
  - **Fast burn-in** (quick convergence)
- Problems with HMC:
  - Energy levels selected randomly $\rightarrow$ **slow mixing**
  - Cannot easily traverse low-density zones $\rightarrow$ **slow
    convergence**

![HMC Samples with
$\varepsilon=0.25$](https://raw.githubusercontent.com/saforem2/l2hmc-dwq25/main/docs/assets/hmc_traj_eps025.svg)

![HMC Samples with
$\varepsilon=0.5$](https://raw.githubusercontent.com/saforem2/l2hmc-dwq25/main/docs/assets/hmc_traj_eps05.svg)

# Topological Freezing

<div class="columns">

<div class="column" width="45%">

<div style="text-align:left; font-size: 0.9em;">

**Topological Charge**:
$$Q = \frac{1}{2\pi}\sum_{P}\left\lfloor x_{P}\right\rfloor  \in \mathbb{Z}$$

</div>

<span class="dim-text" style="font-size:0.8em;">**note:**
$\left\lfloor x_{P} \right\rfloor = x_{P} - 2\pi
\left\lfloor\frac{x_{P} + \pi}{2\pi}\right\rfloor$</span>

> [!IMPORTANT]
>
> ### Critical Slowing Down
>
> - $Q$ gets stuck!
>   - as $\beta\longrightarrow \infty$:
>     - $Q \longrightarrow \text{const.}$
>     - $\delta Q = \left(Q^{\ast} - Q\right) \rightarrow 0 \textcolor{#FF5252}{\Longrightarrow}$
>   - \# configs required to estimate errors  
>     **grows exponentially**:
>     <span class="red-text">$\tau_{\mathrm{int}}^{Q} \longrightarrow \infty$</span>

</div>

<div class="column" width="45%">

<img
src="https://raw.githubusercontent.com/saforem2/l2hmc-dwq25/main/docs/assets/critical_slowing_down.svg"
style="width:80.0%"
alt="Note \delta Q \rightarrow 0 at increasing \beta" />

</div>

</div>

# Can we do better?

<div class="columns">

<div class="column" width="50%">

- Introduce two (**invertible NNs**) `vNet` and `xNet`[^4]:
  - <span style="font-size:0.9em;">`vNet:`
    $(x, F) \longrightarrow \left(s_{v},\, t_{v},\, q_{v}\right)$</span>  
  - <span style="font-size:0.9em;">`xNet:`
    $(x, v) \longrightarrow \left(s_{x},\, t_{x},\, q_{x}\right)$</span>

 

- Use these $(s, t, q)$ in the *generalized* MD update:
  - <span class="pink-text">$\Gamma_{\theta}^{\pm}$</span>
    $: ({x}, \textcolor{#07B875}{v}) \xrightarrow[]{\textcolor{#F06292}{s_{v}, t_{v}, q_{v}}} (x, \textcolor{#07B875}{v'})$
  - <span class="orange-text">$\Lambda_{\theta}^{\pm}$</span>
    $: (\textcolor{#AE81FF}{x}, v) \xrightarrow[]{\textcolor{#FD971F}{s_{x}, t_{x}, q_{x}}} (\textcolor{#AE81FF}{x'}, v)$

</div>

<div class="column" width="48%">

<img src="./assets/leapfrog-layer-2D-U1-vertical.light.svg"
style="width:85%; text-align:center;" />

</div>

</div>

# L2HMC: Generalizing the MD Update

<div class="columns">

<div class="column" width="50%">

> [!NONE]
>
> ### L2HMC Update
>
> - Introduce $d \sim \mathcal{U}(\pm)$ to determine the direction of
>   our update
>
>   1.  $\textcolor{#07B875}{v'} =$
>       <span class="pink-text">$\Gamma^{\pm}$</span>$({x}, \textcolor{#07B875}{v})$
>       <span class="dim-text" style="font-size:0.9em;">$\hspace{46pt}$
>       update $v$</span>
>
>   2.  $\textcolor{#AE81FF}{x'} =$
>       <span class="blue-text">$x_{B}$</span>$\,+\,$<span class="orange-text">$\Lambda^{\pm}$</span>$($<span class="red-text">$x_{A}$</span>$, {v'})$
>       <span class="dim-text" style="font-size:0.9em;">$\hspace{10pt}$
>       update first **half**: $x_{A}$</span>
>
>   3.  $\textcolor{#AE81FF}{x''} =$
>       <span class="red-text">$x'_{A}$</span>$\,+\,$<span class="orange-text">$\Lambda^{\pm}$</span>$($<span class="blue-text">$x'_{B}$</span>$, {v'})$
>       <span class="dim-text" style="font-size:0.9em;">$\hspace{8pt}$
>       update other half: $x_{B}$</span>
>
>   4.  $\textcolor{#07B875}{v''} =$
>       <span class="pink-text">$\Gamma^{\pm}$</span>$({x''}, \textcolor{#07B875}{v'})$
>       <span class="dim-text" style="font-size:0.9em;">$\hspace{36pt}$
>       update $v$</span>

> [!NONE]
>
> ### 🎲 Re-Sampling
>
> - Resample both $v\sim \mathcal{N}(0, 1)$, and
>   $d \sim \mathcal{U}(\pm)$ at the beginning of each trajectory
>   - To ensure ergodicity + reversibility, we split the
>     <span class="purple-text">$x$</span> update into sequential
>     (complementary) updates
> - Introduce directional variable $d \sim \mathcal{U}(\pm)$, resampled
>   at the beginning of each trajectory:
>   - Note that $\left(\Gamma^{+}\right)^{-1} = \Gamma^{-}$, i.e.
>     $$\Gamma^{+}\left[\Gamma^{-}(x, v)\right] = \Gamma^{-}\left[\Gamma^{+}(x,
>     v)\right] = (x, v)$$

</div>

<div class="column" width="50%">

<img src="./assets/leapfrog-layer-2D-U1-vertical.light.svg"
style="width:85%; text-align:center;" />

</div>

</div>

## L2HMC: Leapfrog Layer

<div class="columns">

<div class="column" width="35%">

<img
src="https://raw.githubusercontent.com/saforem2/l2hmc-dwq25/main/docs/assets/drawio/update_steps.svg"
class="absolute" style="width:40.0%" data-top="30" />

</div>

<div class="column" width="65%">

<img
src="https://raw.githubusercontent.com/saforem2/l2hmc-dwq25/main/docs/assets/drawio/leapfrog_layer_dark2.svg"
style="width:100.0%" />

</div>

</div>

<img
src="https://raw.githubusercontent.com/saforem2/l2hmc-dwq25/main/docs/assets/drawio/network_functions.svg"
class="absolute" style="width:100.0%" data-top="440" />

## L2HMC Update

<div class="columns">

<div class="column" width="50%" style="font-size:0.99em;">

> [!NONE]
>
> ### Algorithm
>
> 1.  `input`: <span class="purple-text">$x$</span>
>
>     - Resample:
>       $\textcolor{#07B875}{v} \sim \mathcal{N}(0, \mathbb{1})$;
>       $\,\,{d\sim\mathcal{U}(\pm)}$
>     - Construct initial state:
>       $\textcolor{#939393}{\xi} =(\textcolor{#AE81FF}{x}, \textcolor{#07B875}{v}, {\pm})$
>
> 2.  `forward`: Generate <span style="color:#f8f8f8">proposal
>     $\xi'$</span> by passing <span style="color:#939393">initial
>     $\xi$</span> through $N_{\mathrm{LF}}$ leapfrog layers  
>     $$\textcolor{#939393} \xi \hspace{1pt}\xrightarrow[]{\tiny{\mathrm{LF} \text{ layer}}}\xi_{1} \longrightarrow\cdots \longrightarrow \xi_{N_{\mathrm{LF}}} = \textcolor{#f8f8f8}{\xi'} := (\textcolor{#AE81FF}{x''}, \textcolor{#07B875}{v''})$$
>
>     - Accept / Reject: $$\begin{equation*}
>       A({\textcolor{#f8f8f8}{\xi'}}|{\textcolor{#939393}{\xi}})=
>       \mathrm{min}\left\{1,
>       \frac{\pi(\textcolor{#f8f8f8}{\xi'})}{\pi(\textcolor{#939393}{\xi})} \left| \mathcal{J}\left(\textcolor{#f8f8f8}{\xi'},\textcolor{#939393}{\xi}\right)\right| \right\}
>       \end{equation*}$$
>
> 3.  `backward` (if training):
>
>     - Evaluate the **loss function**[^5]
>       $\mathcal{L}\gets \mathcal{L}_{\theta}(\textcolor{#f8f8f8}{\xi'}, \textcolor{#939393}{\xi})$
>       and backprop
>
> 4.  `return`: $\textcolor{#AE81FF}{x}_{i+1}$  
>     Evaluate MH criteria $(1)$ and return accepted config,
>     $$\textcolor{#AE81FF}{{x}_{i+1}}\gets
>       \begin{cases}
>       \textcolor{#f8f8f8}{\textcolor{#AE81FF}{x''}} \small{\text{ w/ prob }} A(\textcolor{#f8f8f8}{\xi''}|\textcolor{#939393}{\xi}) \hspace{26pt} ✅ \\
>       \textcolor{#939393}{\textcolor{#AE81FF}{x}} \hspace{5pt}\small{\text{ w/ prob }} 1 - A(\textcolor{#f8f8f8}{\xi''}|{\textcolor{#939393}{\xi}}) \hspace{10pt} 🚫
>       \end{cases}$$

</div>

<div class="column" width="50%">

<img src="./assets/leapfrog-layer-2D-U1-vertical.light.svg"
style="width:75%; text-align:center;" />

</div>

</div>

# 4D $SU(3)$ Model

<div class="columns">

<div class="column" width="50%">

> [!NOTE]
>
> ### Link Variables
>
> - Write link variables $U_{\mu}(x) \in SU(3)$:
>
>   $$ \begin{align*} 
>   U_{\mu}(x) &= \mathrm{exp}\left[{i\, \textcolor{#AE81FF}{\omega^{k}_{\mu}(x)} \lambda^{k}}\right]\\
>   &= e^{i \textcolor{#AE81FF}{Q}},\quad \text{with} \quad \textcolor{#AE81FF}{Q} \in \mathfrak{su}(3)
>   \end{align*}$$
>
>   <span style="font-size:0.9em;">where
>   <span class="purple-text">$\omega^{k}_{\mu}(x)$</span>
>   $\in \mathbb{R}$, and $\lambda^{k}$ are the generators of
>   $SU(3)$</span>

> [!TIP]
>
> ### Conjugate Momenta
>
> - Introduce
>   <span class="green-text">$P_{\mu}(x) = P^{k}_{\mu}(x) \lambda^{k}$</span>
>   conjugate to <span class="purple-text">$\omega^{k}_{\mu}(x)$</span>

> [!IMPORTANT]
>
> ### Wilson Action
>
> $$ S_{G} = -\frac{\beta}{6} \sum
> \mathrm{Tr}\left[U_{\mu\nu}(x)
> + U^{\dagger}_{\mu\nu}(x)\right] $$
>
> where $U_{\mu\nu}(x) = U_{\mu}(x) U_{\nu}(x+\hat{\mu})
> U^{\dagger}_{\mu}(x+\hat{\nu}) U^{\dagger}_{\nu}(x)$

</div>

<div class="column" width="45%">

<img src="./assets/u1lattice.dark.svg" style="width:90.0%" />

</div>

</div>

## HMC: 4D $SU(3)$

Hamiltonian: $H[P, U] = \frac{1}{2} P^{2} + S[U] \Longrightarrow$

<div class="columns">

<div class="column" style="font-size:0.9em; text-align: center;">

> [!NONE]
>
> - <span style="border-bottom: 2px solid #AE81FF;">$U$ update</span>:
>   <span class="purple-text"
>   style="font-size:1.5em;">$\frac{d\omega^{k}}{dt} = \frac{\partial H}{\partial P^{k}}$</span>
>   $$\frac{d\omega^{k}}{dt}\lambda^{k} = P^{k}\lambda^{k} \Longrightarrow \frac{dQ}{dt} = P$$
>   $$\begin{align*}
>   Q(\textcolor{#FFEE58}{\varepsilon}) &= Q(0) + \textcolor{#FFEE58}{\varepsilon} P(0)\Longrightarrow\\
>   -i\, \log U(\textcolor{#FFEE58}{\varepsilon}) &= -i\, \log U(0) + \textcolor{#FFEE58}{\varepsilon} P(0) \\
>   U(\textcolor{#FFEE58}{\varepsilon}) &= e^{i\,\textcolor{#FFEE58}{\varepsilon} P(0)} U(0)\Longrightarrow \\
>   &\hspace{1pt}\\
>   \textcolor{#FD971F}{\Lambda}:\,\, U \longrightarrow U' &:= e^{i\varepsilon P'} U
>   \end{align*}$$

<div class="aside">

<span class="dim-text"
style="font-size:0.8em;">$\textcolor{#FFEE58}{\varepsilon}$ is the step
size</span>

</div>

</div>

<div class="column" style="font-size:0.9em; text-align: center;">

> [!NONE]
>
> - <span style="border-bottom: 2px solid #07B875;">$P$ update</span>:
>   <span class="green-text"
>   style="font-size:1.5em;">$\frac{dP^{k}}{dt} = - \frac{\partial H}{\partial \omega^{k}}$</span>
>   $$\frac{dP^{k}}{dt} = - \frac{\partial H}{\partial \omega^{k}}
>   = -\frac{\partial H}{\partial Q} = -\frac{dS}{dQ}\Longrightarrow$$
>   $$\begin{align*}
>   P(\textcolor{#FFEE58}{\varepsilon}) &= P(0) - \textcolor{#FFEE58}{\varepsilon} \left.\frac{dS}{dQ}\right|_{t=0} \\
>   &= P(0) - \textcolor{#FFEE58}{\varepsilon} \,\textcolor{#E599F7}{F[U]} \\
>   &\hspace{1pt}\\
>   \textcolor{#F06292}{\Gamma}:\,\, P \longrightarrow P' &:= P - \frac{\varepsilon}{2} F[U]
>   \end{align*}$$

<div class="aside">

<span class="dim-text"
style="font-size:0.8em;">$\textcolor{#E599F7}{F[U]}$ is the force
term</span>

</div>

</div>

</div>

## HMC: 4D $SU(3)$

<div class="columns">

<div class="column" width="47%" style="text-align:left;">

- <span style="border-bottom: 2px solid #F06292;">Momentum
  Update</span>:
  $$\textcolor{#F06292}{\Gamma}: P \longrightarrow P' := P - \frac{\varepsilon}{2} F[U]$$

- <span style="border-bottom: 2px solid #FD971F;">Link Update</span>:
  $$\textcolor{#FD971F}{\Lambda}: U \longrightarrow U' := e^{i\varepsilon P'} U\quad\quad$$

- We maintain a batch of `Nb` lattices, all updated in parallel

  - $U$`.dtype = complex128`
  - $U$`.shape`  
    <span style="font-size: 0.95em;">`= [Nb, 4, Nt, Nx, Ny, Nz, 3, 3]`</span>

</div>

<div class="column" width="47%" style="text-align:right;">

<img src="./assets/hmc/hmc-update-light.svg" style="width:60.0%" />

</div>

</div>

# Networks 4D $SU(3)$

<div class="columns">

<div class="column" width="54%" style="font-size:0.9em;">

 <br>

 <br>

<span class="purple-text">$U$</span>-Network:

<span style="font-size:0.9em;">`UNet:`
$(U, P) \longrightarrow \left(s_{U},\, t_{U},\, q_{U}\right)$</span>

 <br>

<div style="border: 1px solid #1c1c1c; border-radius: 6px; padding:1%;">

<span class="green-text">$P$</span>-Network:

<span style="font-size:0.9em;">`PNet:`
$(U, P) \longrightarrow \left(s_{P},\, t_{P},\, q_{P}\right)$</span>

</div>

</div>

<div class="column" width="45%" style="text-align:right;">

<img src="./assets/leapfrog-layer-4D-SU3-vertical.light.svg"
style="width:80.0%" />

</div>

</div>

# Networks 4D $SU(3)$

<div class="columns">

<div class="column" width="54%" style="font-size:0.9em;">

 <br>

 <br>

<span class="purple-text">$U$</span>-Network:

<span style="font-size:0.9em;">`UNet:`
$(U, P) \longrightarrow \left(s_{U},\, t_{U},\, q_{U}\right)$</span>

 <br>

<div style="border: 1px solid #07B875; border-radius: 6px; padding:1%;">

<span class="green-text">$P$</span>-Network:

<span style="font-size:0.9em;">`PNet:`
$(U, P) \longrightarrow \left(s_{P},\, t_{P},\, q_{P}\right)$</span>

</div>

<span class="dim-text">$\uparrow$</span>  
<span class="dim-text" style="padding-top: 0.5em!important;">let’s look
at this</span>

</div>

<div class="column" width="45%" style="text-align:right;">

<img src="./assets/leapfrog-layer-4D-SU3-vertical.light.svg"
style="width:80.0%" />

</div>

</div>

## $P$-`Network` (pt. 1)

<div style="text-align:center;">

![](./assets/SU3/PNetwork.light.svg)

</div>

<div class="columns">

<div class="column" width="50%">

- <span style="border-bottom: 2px solid rgba(131, 131, 131, 0.493);">`input`[^6]:
  $\hspace{7pt}\left(U, F\right) := (e^{i Q}, F)$</span>
  $$\begin{align*}
  h_{0} &= \sigma\left( w_{Q} Q + w_{F} F + b \right) \\
  h_{1} &= \sigma\left( w_{1} h_{0} + b_{1} \right) \\
  &\vdots \\
  h_{n} &= \sigma\left(w_{n-1} h_{n-2} + b_{n}\right) \\
  \textcolor{#FF5252}{z} & := \sigma\left(w_{n} h_{n-1} + b_{n}\right) \longrightarrow \\
  \end{align*}$$

</div>

<div class="column" width="50%">

- <span style="border-bottom: 2px solid rgba(131, 131, 131, 0.5);">`output`[^7]:
  $\hspace{7pt} (s_{P}, t_{P}, q_{P})$</span>

  - $s_{P} = \lambda_{s} \tanh(w_s \textcolor{#FF5252}{z} + b_s)$
  - $t_{P} = w_{t} \textcolor{#FF5252}{z} + b_{t}$
  - $q_{P} = \lambda_{q} \tanh(w_{q} \textcolor{#FF5252}{z} + b_{q})$

</div>

</div>

## $P$-`Network` (pt. 2)

<div style="text-align:center;">

![](./assets/SU3/PNetwork.light.svg)

</div>

- Use $(s_{P}, t_{P}, q_{P})$ to update
  $\Gamma^{\pm}: (U, P) \rightarrow
  \left(U, P_{\pm}\right)$[^8]:

  - <span style="color:#FF5252">forward</span>
    $(d = \textcolor{#FF5252}{+})$:
    $$\Gamma^{\textcolor{#FF5252}{+}}(U, P) := P_{\textcolor{#FF5252}{+}} = P \cdot e^{\frac{\varepsilon}{2} s_{P}} - \frac{\varepsilon}{2}\left[ F \cdot e^{\varepsilon q_{P}} + t_{P} \right]$$

  - <span style="color:#1A8FFF;">backward</span>
    $(d = \textcolor{#1A8FFF}{-})$:
    $$\Gamma^{\textcolor{#1A8FFF}{-}}(U, P) := P_{\textcolor{#1A8FFF}{-}} = e^{-\frac{\varepsilon}{2} s_{P}} \left\{P + \frac{\varepsilon}{2}\left[ F \cdot e^{\varepsilon q_{P}} + t_{P} \right]\right\}$$

# Results: 2D $U(1)$

<div class="columns">

<div class="column" width="50%" style="align:top;">

<img
src="https://raw.githubusercontent.com/saforem2/physicsSeminar/main/assets/autocorr_new.svg"
style="width:90.0%" />

</div>

<div class="column" width="33%"
style="text-align:left; padding-top: 5%;">

> [!IMPORTANT]
>
> ### Improvement
>
> We can measure the performance by comparing $\tau_{\mathrm{int}}$ for
> the <span style="color:#FF2052;">**trained model**</span> vs.
> <span style="color:#9F9F9F;">**HMC**</span>.
>
> **Note**: <span style="color:#FF2052;">lower</span> is better

</div>

</div>

<img
src="https://raw.githubusercontent.com/saforem2/physicsSeminar/main/assets/charge_histories.svg"
class="absolute"
style="margin-bottom: 1em;margin-top: 1em;;width:100.0%" data-top="400"
data-left="0" />

## Interpretation

<div class="columns" style="margin-left:1pt;">

<div class="column" width="36%">

<span class="dim-text"
style="text-align:center; font-size:0.8em">Deviation in $x_{P}$</span>

</div>

<div class="column" width="30%">

<span class="dim-text"
style="text-align:right; font-size:0.8em">Topological charge
mixing</span>

</div>

<div class="column" width="32%">

<span class="dim-text"
style="text-align:right!important; font-size:0.8em;">Artificial influx
of energy</span>

</div>

</div>

<img
src="https://raw.githubusercontent.com/saforem2/physicsSeminar/main/assets/ridgeplots.svg"
style="width:100.0%" />

## Interpretation

![Average plaquette: $\langle x_{P}\rangle$ vs LF
step](https://raw.githubusercontent.com/saforem2/physicsSeminar/main/assets/plaqsf_ridgeplot.svg)

![Average energy:
$H - \sum\log|\mathcal{J}|$](https://raw.githubusercontent.com/saforem2/physicsSeminar/main/assets/Hf_ridgeplot.svg)

# 4D $SU(3)$ Results

- Distribution of $\log|\mathcal{J}|$ over all chains, at each *leapfrog
  step*, $N_{\mathrm{LF}}$ ($= 0, 1, \ldots, 8$) during training:

<div>

</div>

## 4D $SU(3)$ Results: $\delta U_{\mu\nu}$

![](./assets/SU3/pdiff.svg)

## 4D $SU(3)$ Results: $\delta U_{\mu\nu}$

![](./assets/SU3/pdiff-robust.svg)

# Next Steps

- Further code development

  -  [`saforem2/l2hmc-qcd`](https://github.com/saforem2/l2hmc-qcd)

- Continue to use / test different network architectures

  - Gauge equivariant NNs for $U_{\mu}(x)$ update

- Continue to test different loss functions for training

- Scaling:

  - Lattice volume
  - Network size
  - Batch size
  - \# of GPUs

## Thank you!

 <br>

<div>

</div>

> [!NOTE]
>
> ### Acknowledgements
>
> This research used resources of the Argonne Leadership Computing
> Facility,  
> which is a DOE Office of Science User Facility supported under
> Contract DE-AC02-06CH11357.

## 

<div style="text-align:center;">

[![](https://raw.githubusercontent.com/saforem2/l2hmc-qcd/main/assets/logo-small.svg)](https://github.com/saforem2/l2hmc-qcd)

<a href="https://hits.seeyoufarm.com"><img alt="hits" src="https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Fsaforem2%2Fl2hmc-qcd&count_bg=%2300CCFF&title_bg=%23555555&icon=&icon_color=%23111111&title=👋&edge_flat=false"></a>
<a href="https://github.com/saforem2/l2hmc-qcd/"><img alt="l2hmc-qcd" src="https://img.shields.io/badge/-l2hmc--qcd-252525?style=flat&logo=github&labelColor=gray"></a>
<a href="https://www.codefactor.io/repository/github/saforem2/l2hmc-qcd"><img alt="codefactor" src="https://www.codefactor.io/repository/github/saforem2/l2hmc-qcd/badge"></a>

<a href="https://arxiv.org/abs/2112.01582"><img alt="arxiv" src="http://img.shields.io/badge/arXiv-2112.01582-B31B1B.svg"></a>
<a href="https://arxiv.org/abs/2105.03418"><img alt="arxiv" src="http://img.shields.io/badge/arXiv-2105.03418-B31B1B.svg"></a>

<a href="https://hydra.cc"><img alt="hydra" src="https://img.shields.io/badge/Config-Hydra-89b8cd"></a>
<a href="https://pytorch.org/get-started/locally/"><img alt="pyTorch" src="https://img.shields.io/badge/PyTorch-ee4c2c?logo=pytorch&logoColor=white"></a>
<a href="https://www.tensorflow.org"><img alt="tensorflow" src="https://img.shields.io/badge/TensorFlow-%23FF6F00.svg?&logo=TensorFlow&logoColor=white"></a>
[<img src="https://raw.githubusercontent.com/wandb/assets/main/wandb-github-badge-28.svg" alt="Weights & Biases monitoring" height=20>](https://wandb.ai/l2hmc-qcd/l2hmc-qcd)

</div>

## Acknowledgements

<div class="columns">

<div class="column" width="50%">

- **Links**:
  - [ Link to github](https://github.com/saforem2/l2hmc-qcd)
  - [ reach out!](mailto:foremans@anl.gov)
- **References**:
  - [Link to slides](https://saforem2.github.io/lattice23/)
    - [ link to github with
      slides](https://github.com/saforem2/lattice23)
  -  (Foreman et al. 2022; Foreman, Jin, and Osborn 2022, 2021)
  -  (Boyda et al. 2022; Shanahan et al. 2022)

</div>

<div class="column" width="50%">

- Huge thank you to:
  - Yannick Meurice
  - Norman Christ
  - Akio Tomiya
  - Nobuyuki Matsumoto
  - Richard Brower
  - Luchang Jin
  - Chulwoo Jung
  - Peter Boyle
  - Taku Izubuchi
  - Denis Boyda
  - Dan Hackett
  - ECP-CSD group
  - <span class="red-text">**ALCF Staff + Datascience Group**</span>

</div>

</div>

## 

<div class="columns">

<div class="column" width="50%">

### Links

- [ `saforem2/l2hmc-qcd`](https://github.com/saforem2/l2hmc-qcd)

- [📊 slides](https://saforem2.github.io/lattice23) (Github: [
  `saforem2/lattice23`](https://github.com/saforem2/lattice23))

</div>

<div class="column" width="50%">

### References

- [Title Slide Background (worms)
  animation](https://saforem2.github.io/grid-worms-animation/)
  - Github: [
    `saforem2/grid-worms-animation`](https://github.com/saforem2/grid-worms-animation)
- [Link to HMC demo](https://chi-feng.github.io/mcmc-demo/app.html)

</div>

</div>

## References

(I don’t know why this is broken 🤷🏻‍♂️ )

<div id="refs" class="references csl-bib-body hanging-indent"
entry-spacing="0">

<div id="ref-Boyda:2022nmh" class="csl-entry">

Boyda, Denis et al. 2022. “<span class="nocase">Applications of Machine
Learning to Lattice Quantum Field Theory</span>.” In *Snowmass 2021*.
<https://arxiv.org/abs/2202.05838>.

</div>

<div id="ref-Foreman:2021ljl" class="csl-entry">

Foreman, Sam, Taku Izubuchi, Luchang Jin, Xiao-Yong Jin, James C.
Osborn, and Akio Tomiya. 2022. “<span class="nocase">HMC with
Normalizing Flows</span>.” *PoS* LATTICE2021: 073.
<https://doi.org/10.22323/1.396.0073>.

</div>

<div id="ref-Foreman:2021ixr" class="csl-entry">

Foreman, Sam, Xiao-Yong Jin, and James C. Osborn. 2021. “Deep Learning
Hamiltonian Monte Carlo.” In *<span class="nocase">9th International
Conference on Learning Representations</span>*.
<https://arxiv.org/abs/2105.03418>.

</div>

<div id="ref-Foreman:2021rhs" class="csl-entry">

———. 2022. “<span class="nocase">LeapfrogLayers: A Trainable Framework
for Effective Topological Sampling</span>.” *PoS* LATTICE2021 (May):
508. <https://doi.org/10.22323/1.396.0508>.

</div>

<div id="ref-Shanahan:2022ifi" class="csl-entry">

Shanahan, Phiala et al. 2022. “Snowmass 2021 Computational Frontier
CompF03 Topical Group Report: Machine Learning,” September.
<https://arxiv.org/abs/2209.07559>.

</div>

</div>

# Extras

## Integrated Autocorrelation Time

<img
src="https://raw.githubusercontent.com/saforem2/physicsSeminar/main/assets/tint1.svg"
style="width:100.0%" />

## Comparison

<img src="https://saforem2.github.io/anl-job-talk/assets/dQint_eval.svg"
data-ref-parent="fig-comparison" />

<img src="https://saforem2.github.io/anl-job-talk/assets/dQint_hmc.svg"
data-ref-parent="fig-comparison" />

## Plaquette analysis: $x_{P}$

<div class="columns">

<div class="column" width="55%">

<span class="dim-text"
style="text-align:center; font-size:0.9em;">Deviation from
$V\rightarrow\infty$ limit, $x_{P}^{\ast}$</span>

</div>

<div class="column" width="45%">

<span class="dim-text"
style="text-align:right!important; font-size:0.9em;">Average
$\langle x_{P}\rangle$, with $x_{P}^{\ast}$ (dotted-lines)</span>

</div>

</div>

<img
src="https://raw.githubusercontent.com/saforem2/physicsSeminar/main/assets/plaqsf_vs_lf_step1.svg"
style="width:100.0%" />

## Loss Function

- Want to maximize the *expected* squared charge difference[^9]:
  $$\begin{equation*}
  \mathcal{L}_{\theta}\left(\xi^{\ast}, \xi\right) =
  {\mathbb{E}_{p(\xi)}}\big[-\textcolor{#FA5252}{{\delta Q}}^{2}
  \left(\xi^{\ast}, \xi \right)\cdot A(\xi^{\ast}|\xi)\big]
  \end{equation*}$$

- Where:

  - $\delta Q$ is the *tunneling rate*: $$\begin{equation*}
    \textcolor{#FA5252}{\delta Q}(\xi^{\ast},\xi)=\left|Q^{\ast} - Q\right|
    \end{equation*}$$

  - $A(\xi^{\ast}|\xi)$ is the probability[^10] of accepting the
    proposal $\xi^{\ast}$: $$\begin{equation*}
    A(\xi^{\ast}|\xi) = \mathrm{min}\left( 1,
    \frac{p(\xi^{\ast})}{p(\xi)}\left|\frac{\partial \xi^{\ast}}{\partial
    \xi^{T}}\right|\right)
    \end{equation*}$$

## Networks 2D $U(1)$

- Stack gauge links as `shape`$\left(U_{\mu}\right)$`=[Nb, 2, Nt, Nx]`
  $\in \mathbb{C}$

  $$ x_{\mu}(n) ≔ \left[\cos(x), \sin(x)\right]$$

  with `shape`$\left(x_{\mu}\right)$`= [Nb, 2, Nt, Nx, 2]`
  $\in \mathbb{R}$

- $x$-Network:

  - <span class="purple-text">$\psi_{\theta}: (x, v) \longrightarrow \left(s_{x},\, t_{x},\, q_{x}\right)$</span>

- $v$-Network:

  - <span class="green-text">$\varphi_{\theta}: (x, v) \longrightarrow \left(s_{v},\, t_{v},\, q_{v}\right)$</span>
    <span class="dim-text">$\hspace{2pt}\longleftarrow$ lets look at
    this</span>

## $v$-Update[^11]

- <span style="color:#FF5252">forward</span>
  $(d = \textcolor{#FF5252}{+})$:

$$\Gamma^{\textcolor{#FF5252}{+}}: (x, v) \rightarrow v' := v \cdot e^{\frac{\varepsilon}{2} s_{v}} - \frac{\varepsilon}{2}\left[ F \cdot e^{\varepsilon q_{v}} + t_{v} \right]$$

- <span style="color:#1A8FFF;">backward</span>
  $(d = \textcolor{#1A8FFF}{-})$:

$$\Gamma^{\textcolor{#1A8FFF}{-}}: (x, v) \rightarrow v' := e^{-\frac{\varepsilon}{2} s_{v}} \left\{v + \frac{\varepsilon}{2}\left[ F \cdot e^{\varepsilon q_{v}} + t_{v} \right]\right\}$$

## $x$-Update

- <span style="color:#FF5252">forward</span>
  $(d = \textcolor{#FF5252}{+})$:

$$\Lambda^{\textcolor{#FF5252}{+}}(x, v) = x \cdot e^{\frac{\varepsilon}{2} s_{x}} - \frac{\varepsilon}{2}\left[ v \cdot e^{\varepsilon q_{x}} + t_{x} \right]$$

- <span style="color:#1A8FFF;">backward</span>
  $(d = \textcolor{#1A8FFF}{-})$:

$$\Lambda^{\textcolor{#1A8FFF}{-}}(x, v) = e^{-\frac{\varepsilon}{2} s_{x}} \left\{x + \frac{\varepsilon}{2}\left[ v \cdot e^{\varepsilon q_{x}} + t_{x} \right]\right\}$$

## Lattice Gauge Theory (2D $U(1)$)

<div class="columns" layout-valign="top">

<div class="column" width="50%">

<div style="text-align:center;">

> [!NOTE]
>
> ### Link Variables
>
> $$U_{\mu}(n) = e^{i x_{\mu}(n)}\in \mathbb{C},\quad \text{where}\quad$$
> $$x_{\mu}(n) \in [-\pi,\pi)$$

<div>

> [!IMPORTANT]
>
> ### Wilson Action
>
> $$S_{\beta}(x) = \beta\sum_{P} \cos \textcolor{#00CCFF}{x_{P}},$$
>
> $$\textcolor{#00CCFF}{x_{P}} = \left[x_{\mu}(n) + x_{\nu}(n+\hat{\mu})
> - x_{\mu}(n+\hat{\nu})-x_{\nu}(n)\right]$$

<span class="dim-text" style="font-size:0.8em;">**Note**:
$\textcolor{#00CCFF}{x_{P}}$ is the product of links around $1\times 1$
square, called a <span class="blue-text">“plaquette”</span></span>

</div>

</div>

</div>

<div class="column" width="50%">

<img
src="https://raw.githubusercontent.com/saforem2/deep-fridays/main/assets/u1lattice.dark.svg"
style="width:80.0%" alt="2D Lattice" />

</div>

</div>

## 

<iframe data-src="https://nbviewer.org/github/saforem2/l2hmc-qcd/blob/SU3/src/l2hmc/notebooks/l2hmc-2dU1.ipynb" width="100%" height="650" title="l2hmc-qcd">
</iframe>

## Annealing Schedule

- Introduce an *annealing schedule* during the training phase:

  $$\left\{ \gamma_{t}  \right\}_{t=0}^{N} = \left\{\gamma_{0}, \gamma_{1},
  \ldots, \gamma_{N-1}, \gamma_{N} \right\}$$

  where $\gamma_{0} < \gamma_{1} < \cdots < \gamma_{N} \equiv 1$, and
  $\left|\gamma_{t+1} - \gamma_{t}\right| \ll 1$

- <span class="green-text">**Note**</span>:

  - for $\left|\gamma_{t}\right| < 1$, this rescaling helps to reduce
    the height of the energy barriers $\Longrightarrow$
  - easier for our sampler to explore previously inaccessible regions of
    the phase space

## Networks 2D $U(1)$

- Stack gauge links as `shape`$\left(U_{\mu}\right)$`=[Nb, 2, Nt, Nx]`
  $\in \mathbb{C}$

  $$ x_{\mu}(n) ≔ \left[\cos(x), \sin(x)\right]$$

  with `shape`$\left(x_{\mu}\right)$`= [Nb, 2, Nt, Nx, 2]`
  $\in \mathbb{R}$

- $x$-Network:

  - <span class="purple-text">$\psi_{\theta}: (x, v) \longrightarrow \left(s_{x},\, t_{x},\, q_{x}\right)$</span>

- $v$-Network:

  - <span class="green-text">$\varphi_{\theta}: (x, v) \longrightarrow \left(s_{v},\, t_{v},\, q_{v}\right)$</span>

## Toy Example: GMM $\in \mathbb{R}^{2}$

<img
src="https://raw.githubusercontent.com/saforem2/l2hmc-dwq25/main/docs/assets/iso_gmm_chains.svg"
id="fig-gmm" class="r-stretch" />

## Physical Quantities

- To estimate physical quantities, we:
  - Calculate physical observables at **increasing** spatial resolution
  - Perform extrapolation to continuum limit

![](https://raw.githubusercontent.com/saforem2/physicsSeminar/main/assets/static/continuum.svg)

# Extra

<span class="preview-image"
style="text-align:center; margin-left:auto; margin-right: auto;">![](./assets/thumbnail.png)</span>

[^1]: Here, $\sim$ means “is distributed according to”

[^2]: Here, $\sim$ means “is distributed according to”

[^3]: We **always** start by resampling the momentum, $v_{0} \sim
    \mathcal{N}(0, \mathbb{1})$

[^4]: [L2HMC:](https://github.com/saforem2/l2hmc-qcd) (Foreman, Jin, and
    Osborn 2021, 2022)

[^5]: For simple $\mathbf{x} \in \mathbb{R}^{2}$ example,
    $\mathcal{L}_{\theta} =
    A(\xi^{\ast}|\xi)\cdot \left(\mathbf{x}^{\ast} - \mathbf{x}\right)^{2}$

[^6]: $\sigma(\cdot)$ denotes an activation function

[^7]: $\lambda_{s},\, \lambda_{q} \in \mathbb{R}$ are trainable
    parameters

[^8]: Note that $\left(\Gamma^{+}\right)^{-1} = \Gamma^{-}$,
    i.e. $\Gamma^{+}\left[\Gamma^{-}(U, P)\right] = \Gamma^{-}\left[\Gamma^{+}(U, P)\right] = (U, P)$

[^9]: Where $\xi^{\ast}$ is the *proposed* configuration (prior to
    Accept / Reject)

[^10]: And $\left|\frac{\partial \xi^{\ast}}{\partial \xi^{T}}\right|$
    is the Jacobian of the transformation from
    $\xi \rightarrow \xi^{\ast}$

[^11]: <span style="font-size:0.8em;">Note that
    $\left(\Gamma^{+}\right)^{-1} = \Gamma^{-}$,
    i.e. $\Gamma^{+}\left[\Gamma^{-}(x, v)\right] = \Gamma^{-}\left[\Gamma^{+}(x, v)\right] = (x, v)$</span>
