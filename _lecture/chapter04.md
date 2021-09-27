---
layout: default
title: "Chapter 04"
parent: Lecture
date: 2021-08-18
categories: lecture
author: Lars Pastewka
nav_order: 04
---
---

<h2 class='chapterHead'><span class='titlemark'>Chapter 4</span><br />
<a id='x1-10004'></a>Temperature control</h2>
<div id='shaded*-1' class='framedenv'><!--  l. 3  -->
<p class='noindent'><span class='underline'><span class='cmbx-12'>Context:</span></span> Most molecular dynamics calculations are carried out in <span class='cmti-12'>thermal</span> <span class='cmti-12'>equilibrium</span>. Equilibrium is typically maintained by coupling the molecular calculation to a virtual <span class='cmti-12'>heat bath</span>, with which it exchanges energy but no particles. This chapter discusses properties of thermal equilibrium and introduces simple algorithms for heat-bath
coupling.</p>
</div>
<!--  l. 7  -->
<p class='noindent'><a href='https://uni-freiburg.cloud.panopto.eu/Panopto/Pages/Embed.aspx?id=2adb1988-1575-473f-8b8b-ad230158c41c' class='url'><span class='cmtt-12'>https://uni-freiburg.cloud.panopto.eu/Panopto/Pages/Embed.aspx?id=2adb1988-1575-473f-8b8b-ad230158c41c</span></a></p>
<h3 class='sectionHead'><span class='titlemark'>4.1</span> <a id='x1-20004.1'></a>Introduction</h3>
<!--  l. 11  -->
<p class='noindent'>In order to talk about temperature control, we need to discuss the properties of thermal equilibrium. This is the realm of <span class='cmti-12'>statistical mechanics</span> or <span class='cmti-12'>statistical</span> <span class='cmti-12'>thermodynamics</span> that is discussed in more detail in Chapter <span class='cmbx-12'>??</span> and Appendix <span class='cmbx-12'>??</span>. A key outcome is that the velocity components are distributed according to a Boltzmann
distribution. The velocity magnitude is then distributed according to a Maxwell-Boltzmann distribution.</p>
<!--  l. 13  -->
<p class='indent'>A thermostat implicitly models the coupling to a heat bath much larger than the atomistic system under investigation. Because it is much larger, its temperature will not change when energy flows from and to the heat bath. The atomistic system becomes canonical and its statistics follows the <span class='cmti-12'>canonical</span> <span class='cmti-12'>ensemble</span>. An ensemble here describes which parameters are constrained, and the canonical ensemble is often also called the
<span class='cmti-12'>NVT-ensemble</span>, because particle number \(N\), volume \(V\) and temperature \(T\) are constrained (fixed). An ideal thermostat guarantees relaxation of the distribution of atomic degrees of freedom to the <span class='cmti-12'>canonical distribution function</span> (see Chapter <span class='cmbx-12'>??</span>).</p>
<!--  l. 15  -->
<p class='indent'>We will here start with a mechanistic treatment of thermostats and underpin it with more rigorous theory in Chapter <span class='cmbx-12'>??</span>. The present chapter teaches the basic concepts required for an implementation of simple thermostatting schemes. Thermostats can be roughly categorized into constraint methods (velocity rescaling and Berendsen), stochastic methods (Andersen, Langevin and dissipative particle dynamics) and extended system methods (Nosé-Hoover). Constraint
and extended system methods are deterministic, i.e. they follow the same path when starting from the same initial state. In this chapter we will only discuss the simple constraint methods. We will come back to more advanced methods for temperature control later in these notes.</p>
<!--  l. 18  -->
<p class='noindent'></p>
<h3 class='sectionHead'><span class='titlemark'>4.2</span> <a id='x1-30004.2'></a>Simple themostatting schemes</h3>
<!--  l. 20  -->
<p class='noindent'></p>
<h4 class='subsectionHead'><span class='titlemark'>4.2.1</span> <a id='x1-40004.2.1'></a>Velocity rescaling</h4>
<!--  l. 22  -->
<p class='noindent'>The crudest (and simplest) form of fixing the temperature in a molecular dynamics simulation to a value of \(T_0\) is by velocity rescaling. Since the instantaneous temperature is \begin{equation} \frac{3}{2} k_BT = \sum _i \frac{1}{2} mv_i^2, \end{equation} we obtain a temperature of \(T_0\) if we rescale all velocities by \begin{equation} \label{eq:velocity-rescaling} \vec{v}_i \to \lambda \vec{v}_i\ \text{ with } \ \lambda =\sqrt{\frac{T_0}{T}} \end{equation} after every time step.
This is a very intrusive way of setting the temperature and should not be used in any practical situations, but it is a good illustration of how a simple constraint method works.</p>
<!--  l. 33  -->
<p class='noindent'></p>
<h4 class='subsectionHead'><span class='titlemark'>4.2.2</span> <a id='x1-50004.2.2'></a>Berendsen thermostat</h4>
<!--  l. 35  -->
<p class='noindent'>The <a href='#Xberendsen_molecular_1984'>Berendsen et al.</a> (<a href='#Xberendsen_molecular_1984'>1984</a>) thermostat uses a damping or acceleration term to control the temperature. The governing equations of motion of the Berendsen thermostat are \begin{equation} m \dot{\vec{v}}_i = \vec{f}_i + \frac{m}{2\tau } \left ( \frac{T_0}{T} -1 \right ) \vec{v}_i \label{eq:berendsen} \end{equation} where \(\tau \) is a relaxation time constant. The factor in front of the velocity
is a damping coefficient. The coefficient vanishes for \(T = T_0\), Eq. \eqref{eq:berendsen} then reduces to Newton’s equation of motion. However, it has a positive sign (=speeds up particles) for \(T &lt; T_0\) and has negative sign (=slows down particles) for \(T &gt; T_0\). From Eq. \eqref{eq:berendsen} we can easily derive a differential equation for the evolution of the temperature: \begin{align} 3k_B \frac{dT}{dt} &amp;= \sum _i m \vec{v}_i \cdot \dot{\vec{v}}_i \\ &amp;= \sum _i \left [
\vec{v}_i \cdot \vec{f}_i + \frac{1}{2\tau } \left ( \frac{T_0}{T} - 1 \right ) m v_i^2 \right ] \\ &amp;= - \frac{dE_\text{pot}}{dt} + \frac{3k_B(T_0 - T)}{\tau } \end{align}</p>
<!--  l. 52  -->
<p class='indent'>This can be written as \begin{equation} \label{eq:berendsen-temperature-evolution} \frac{dT}{dt} = -\frac{T-T_0}{\tau } + S \end{equation} where \(S=-\frac{1}{3k_B} \frac{dE_\text{pot}}{dt}\) is the change of <span class='cmti-12'>potential</span> energy and constitutes an additional temperature (energy) source.</p>
<!--  l. 59  -->
<p class='indent'>For \(S = 0\), this equation is solved by \begin{equation} \label{eq:berendsen-explicit-temperature-evolution} T(t) = T_0 + (T_1-T_0) e^{-t/\tau } \end{equation} The temperature relaxes exponentially from the intial value \(T_1\) towards \(T_0\). We directly see that \(\tau \) in Eq. \eqref{eq:berendsen} is indeed the relaxation time constant.</p>
<!--  l. 66  -->
<p class='indent'>Note that Eq. \eqref{eq:berendsen-explicit-temperature-evolution} suggests an implementation of the Berendsen thermostat in terms of velocity rescaling. During at single time step \(\Delta t \ll \tau \), the temperature changes from \(T\) to \(T_0 + (T-T_0)e^{-\Delta t / \tau }\). We can implement this as velocity rescaling, Eq. \eqref{eq:velocity-rescaling}, with \begin{equation} \lambda = \sqrt{\frac{T_0}{T} + \left ( 1- \frac{T_0}{T} \right ) e^{-\frac{\Delta t}{\tau }}}
\approx \sqrt{1+ \left ( \frac{T_0}{T} - 1 \right ) \frac{\Delta t}{\tau }} \end{equation} where \(T\) is the current (measure) temperature and \(T_0\) is the target temperature.</p>
<!--  l. 73  -->
<p class='indent'>A Berendsen thermostat therefore constitutes a gentle way of rescaling velocities. The relaxation time \(\tau \) determines the strength of the coupling between thermal bath and atomistic system. The velocity rescaling limit \(\lambda \to \sqrt{T_0/T}\) is obtained as \(\tau \to 0\). Thermostats should be tuned as weak as possible and as strong as necessary to disturb the system the least while still allowing it to reach the target temperature within the simulation time. There is the
additional requirement \(\tau \gg \Delta t\) (where \(\Delta t\) is the time step), otherwise equation Eq. \eqref{eq:berendsen-temperature-evolution} will not be sampled properly numerically. The velocity rescaling thermostat discussed above is bad because it is very strong, but also because it violates \(\tau \gg \Delta t\).</p>
<!--  l. 75  -->
<p class='noindent'></p>
<h3 class='sectionHead'><span class='titlemark'>4.3</span> <a id='x1-60004.3'></a>Equilibrating a molecular simulation</h3>
<!--  l. 77  -->
<p class='noindent'>A “happy” molecular dynamics simulation will nicely run at constant temperature. Simulations are only this happy once they are <span class='cmti-12'>equilibrated</span> and this equilibration implies that the positions \(\{\vec{r}_i\}\) are such that the system resides somewhere near a (potentially local) minimum in the potential energy landscape. When we set up a new simulation, we have to guess a set of \(\{\vec{r}_i\}\) that are often far away from this minimum. (For crystalline
solids this guess is simple, since we typically know the crystal structure that we are interested in. For liquids, the guess is more difficult since the overall structure is disordered.) Since the forces \(\{\vec{f}_i\}\) point towards the minimum, the system will evolve in this direction and the potential energy \(E_\text{pot}\) will decrease over time, \(d E_\text{pot}/dt &lt; 0\). Equation \eqref{eq:berendsen-temperature-evolution} tells us, that this leads to an increase in temperature since
\(S&gt;0\).</p>
<!--  l. 79  -->
<p class='indent'>A common problem is that this temperature can be large enough to vaporize the system, i.e. the temperature increases above the vaporization point. The first step in any molecular dynamics simulation is hence to <span class='cmti-12'>equilibrate</span> the system while avoiding a temperature rise above the point of vaporization (or melting if you are setting up a solid). This can be achieved by running a calculation with a Berendsen thermostat and a strong coupling (i.e. a small \(\tau
\)). Once the system has equilibrated, the value of \(\tau \) can be adjusted to a more reasonable relaxation time that does not disturb the calculation too much. Good values for \(\tau \) are between \(1\) ps and \(10\) ps.</p>
<!--  l. 81  -->
<p class='indent'>Note that if we <span class='cmti-12'>continuously</span> pump energy into our system, for example because we deform it externally, then Eq. \eqref{eq:berendsen-temperature-evolution} acquires a non-zero source term, \(S &gt; 0\). Assuming \(S\) is constant over time, the final temperature is shifted to \(T_0\,' = T_0 + S\tau \). This temperature offset gets smaller with increasing coupling strength \(1/\tau \).</p>
<h2 class='likechapterHead'><a id='x1-70004.3'></a>Bibliography</h2>
<div class='thebibliography'>
<p class='bibitem'><span class='biblabel'><a id='Xberendsen_molecular_1984'></a><span class='bibsp'>   </span></span>H. J. C. Berendsen, J. P. M. Postma, W. F. van Gunsteren, A. DiNola, and J. R. Haak. Molecular dynamics with coupling to an external bath. <span class='cmti-12'>J. Chem. Phys.</span>, 81(8):3684–3690, 1984. URL <a href='https://doi.org/10.1063/1.448118' class='url'><span class='cmtt-12'>https://doi.org/10.1063/1.448118</span></a>.</p>
</div>
