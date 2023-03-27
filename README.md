# aQa-WAVES-Protocol
The WAVES (Witness-Assisted Variational Eigenspectra Solver) protocol employs a witness assisted variational search to find ground and excited states of a general molecule. The protocol is implemented in CIRQ and OpenFermion to investigate the eigenstate manifold of a Hydrogen molecule.

## Abstract
The WAVES protocol was implemented to find ground and excited states of the $\mathrm{H}_2$ molecule. The corresponding energies reached fidelities upwards of 97\%. IPEA (Iterative Phase Estimation Algorithm) was successfully implemented in an attempt to further refine the energy estimates. 

## Introduction
Classical numerical simulation of molecules quickly becomes intractable due to a Hilbert space that grows exponentially with system size. Quantum computers address this by mapping systems of interacting fermions to spin problems. Molecules are then simulated with a series of (parameterized) unitary operations on qubits. 

Ground states energies are easily found through the variational principle. The search for excited states requires more advanced methods. Eigenstate witnessing maps excited state energies to local minima in parameter space. The WAVES protocol employs the von Neumann entropy as an eigenstate witness to search for excited states. 

## Theory
### 1. Hamiltonian Simulation
Excited and ground state energies of molecules can be found by solving the electronic structure Hamiltonian. $$\hat{H} = \Sigma_{i,j}^M J_{ij} \hat{a}^\dagger_i\hat{a_j} + \Sigma_{i,j,k,l}^M V_{ijkl}\hat{a}^\dagger_i\hat{a}^\dagger_j\hat{a}_k\hat{a}_l $$

where $\hat{a}^\dagger_n$ ($\hat{a_n}$) are fermionic creation (annihilation) operators corresponding to the _n_ th mode. The number of terms in this Hamiltonian grows exponentially with system size as ~ $O(M^4)$. In practice the matrix elements of the many-body interaction terms ${V_{ijkl} }$ are small and are often treated as a perturbation on the quadratic Hamiltonian. 

To be able to simulate fermionic Hamiltonians in quantum computers they must be mapped onto the qubit basis with a suitable transformation (e.g. Jordan-Wigner, Bravyi-Kitaev). Time evolution under a given Hamiltonian $\hat{H}$ is described by the unitary $\hat{U}(t) = e^{-i\hat{H}t}$. In practice $\hat{U}(t)$ is Trotter decomposed into a series of multiplicative terms corresponding to Pauli strings from the sum in the Hamiltonian. These are then implemented as rotations, up an overall finite Trotter error. 

### 2. Variational Quantum Eigensolvers
Variational Quantum Eigensolvers (VQEs) are hybrid classical-quantum algorithms that are used to find eigenvalues of a Hamiltonian. A series of parameterised quantum gates are updated via a classical optimizer to minimize a system specific cost function. VQEs are easier to implement on NISQ devices because of this classical processing step, in contrast to fully quantum algorithms like Quantum Phase Estimation (QPE).

The VQE determines an ansatz state via a series of parameterised gates. The number of parameters are determined by the number of modes $n$ in the fermionic system - They can scale like $~O(n^4)$ in the general case. Parameters can be removed through symmetry (e.g. particle number conserving) or cost function specific reasons (e.g. $\hat{a}^\dagger_i\hat{a}^\dagger_i\hat{a_j}\hat{a_j}$ will not affect the energy $E$, hence set $V_{iijj} \rightarrow 0$ ).

Particle number conservation is an important symmetry in quantum chemistry problems. A system of 4 spin-orbitals and 2 electrons can be represented by 4 qubits, where $|0\rangle$  ($|1\rangle$) denote empty (full) spin-orbitals. The only valid states are superpositions of fixed particle number $$|\psi\rangle = c_1|1100\rangle + c_2|1010\rangle + c_3|1001\rangle + c_4|0110\rangle + c_5|0101\rangle + c_6|0011\rangle$$

Givens rotations can be used to excite an electron from one mode to another while conserving particle number. It has been shown that Givens rotations are universal for quantum chemistry circuits. Starting from the Hartree-Fock state $|1100\rangle$, any valid state can be reached by applying the following parameterised (controlled) single and double excitation gates.

| ![givens](https://user-images.githubusercontent.com/98838428/227919765-68b93535-ba91-4d9a-8c0e-8fac39eb7d72.png) |
|:--:| 
| Figure 1: Universal quantum circuit for the H2 molecule with 5 parameters, from [2].|

### The WAVES Protocol

The purpose of the WAVES (Witness-Assisted Variational Eigenspectra Solver) protocol is to find estimates for ground and excited state energies of a general molecule using a witness-assisted variational search. These energy estimates are then further refined using an IPEA (Iterative Phase Estimation Algorithm). 

| ![waves12](https://user-images.githubusercontent.com/98838428/227921303-073d578b-e261-46e8-9ba3-f126a615d0f8.png) |
|:--:| 
| Figure 2: Quantum circuit used to variationally search for eigenstates according to the WAVES protocol, from [1].|


The variational circuit to find ground and excited states is given in Fig. [2]. The algorithm begins by setting the system register to an ''initial guess'' $|\Phi_\mathrm{T}\rangle$. In the case of the ground state search for the $\mathrm{H_2}$ molecule $|\Phi_\mathrm{T}\rangle = |1100\rangle$, the Hartree-Fock state, which minimizes the energy of the corresponding non-interacting (quadratic) Hamiltonian $\hat{H_0}$.

The desired eigenstate manifold (the algorithm cannot distinguish between degenerate states) has a corresponding excitation operator $\hat{E_{p_i}}$ (tab. 1). The ansatz effectively takes the form $\hat{E_{p_i}} \hat{A}(\vec{\theta_i})$, where $\hat{A}(\vec{\theta})$ is the parameterised circuit and $\vec{\theta}$ are the variational parameters. 

| ![ancilla_dm](https://user-images.githubusercontent.com/98838428/227922915-0cfb7835-26a3-41e0-9a52-97219c9858ac.png) |
|:--:| 
| Figure 3: Quantum circuit describing the state of the composite system $\psi_{\mathrm{AS}}\rangle$ after the application of the controlled unitary _U_, from [1].|

Energies are found by performing state tomography on the ancilla qubit. Multiple measurements of the ancilla in the $X$ and $Y$ basis ($\langle Z \rangle = 0$) are used to rebuild the off-diagonal elements of the density matrix given in the system in Fig. 3. The reduced density matrix $\rho_\mathrm{A}$ describing state of the ancilla qubit is found as follows, starting from the composite state of the qubit system after the controlled unitary is applied. $$|\psi_{\mathrm{AS}} \rangle = \Sigma_{n=0}^{2^N-1}c_{n}|\phi_n\rangle \otimes \frac{1}{\sqrt{2}}(|0\rangle + e^{i2\pi\phi_n}|1\rangle)$$ 
Where the state of the system $|\psi_\mathrm{S}\rangle$ has been decomposed into the (complete) set of eigenvectors of the unitary $U$. Notice the eigenphases of $U$ have been _kicked-back_ onto the state of the ancilla qubit. The density matrix of the entire qubit system is given by
$$\hat{\rho} = |\psi_{\mathrm{AS}} \rangle\langle \psi_{\mathrm{AS}}|$$
$$=\Sigma_{n,m}^{2^N-1}c_n c_m^\* |n\rangle \langle m | \otimes \frac{1}{2}(|0\rangle \langle 0 | + e^{-i2\pi \phi_m}|0\rangle \langle 1 | + e^{i2\pi \phi_n}|1\rangle \langle 0 | + e^{i2\pi(\phi_n-\phi_m)}|1\rangle \langle 1 |)$$
The reduced density matrix describing the state of the ancilla qubit is found by tracing over the system indices. Notice that the $\langle k |n\rangle \langle m | k \rangle$ term is only non-zero when $k=n=m$.
$$\hat{\rho_\mathrm{A}}=\mathrm{tr_\mathrm{S}}(\hat{\rho}) = \Sigma_k\langle \phi_k | \hat{\rho} | \phi_k \rangle$$
$$=\frac{1}{2}\Sigma_n^{2^N-1}|c_n|^2(|0\rangle \langle 0 | + e^{-i2\pi \phi_n}|0\rangle \langle 1 | + e^{i2\pi \phi_n}|1\rangle \langle 0 | + |1\rangle \langle 1 |)$$
Recalling that the sum of probabilities $|c_n|^2$ must equal unity for a physical state, the reduced density matrix of the ancilla falls out to be

| ![image](https://user-images.githubusercontent.com/98838428/227923975-9d865940-c85b-40f7-a00b-3bce9d5fcf51.png) |
|:--:| 


It is easy to see that the correct reduced density matrix is recovered when $|\psi_\mathrm{S}\rangle$ starts out as a eigenstate of $U$. Notice that the off-diagonal elements of $\hat{\rho_\mathrm{A}}$ provide an energy estimator $\mathcal{E} = -Arg\[\langle \psi_\mathrm{S}|e^{-i\hat{H}t}|\psi_\mathrm{S}\rangle\]/t$.

The WAVES protocol searches for eigenstates by varying the ansatz parameters according to a classical minimizer. The cost function for this variational search is $$\mathcal{F_\mathrm{cost}} = \mathcal{E} + \mathcal{T}\mathcal{S}(\hat{\rho_\mathrm{A}})$$ where $\mathcal{S}(\hat{\rho_\mathrm{A}})$ is the von Neumann entropy. This terms acts as an eigenstate witness; it is zero if the target state is an eigenstate of the Hamiltonian. The parameter $\mathcal{T}$ trades off between energy and entropy optimization. In practice this cost function can be approximated in terms of the purity $\mathcal{P} = \mathrm{tr(\hat{\rho_\mathrm{A}}^2)}$ as $$\mathcal{F_\mathrm{cost}} = \mathcal{E} - \mathcal{T}\mathcal{P}$$ 

Excited states are found starting from the ground state $|\Phi \rangle = \hat{A}(\theta_\mathrm{gs})|\mathrm{HF}\rangle$ and applying the corresponding excitation operators (Tab. 1) in the high $\mathcal{T}$ limit. Minimize $\mathcal{F_\mathrm{cost}}$ to obtain the unitary for the target excited state $\hat{E_{p_i}} \hat{A}(\vec{\theta_{e_i}})$. The same circuit can be used to find the ground state in the low temperature limit. 

The final step of the WAVES protocol uses the Iterative Phase Estimation Algorithm (IPEA) to further refine the energy estimate (Fig. 4). IPEA is a type of QPE that can be implemented relatively easily on NISQ devices, but needs exponentially many controlled unitaries and repeated measurements. The results of previous measurements (binary bits) are used to update subsequent quantum circuits. The least significant bits are measured first. IPEA can be used on eigenstates or mixed states - If the latter is input the algorithm will return an eigenphase, with probability given by the relative weights of the eigenstates. IPEA can accurately return the correct eigenvalues - It works best when the input state is close to being an eigenstate, hence using this algorithm to refine the energy estimate from the variational search. 

| ![waves3](https://user-images.githubusercontent.com/98838428/227924951-f69c1ff1-79ae-4b4b-8321-941ee2829847.png) |
|:--:| 
| Figure 4: Quantum circuit used to implement IPEA, from [1].|

## Results \& Discussion
The WAVES protocol was implemented in python (using cirq [3] and OpenFermion [4]) to find the ground and excited state energies of the $\mathrm{H_2}$ molecule. The table below collates a representative sample of the results (Tab. 1). Excited and ground state energies obtained via the variational search show very high fidelities with the measured values [4]. Higher energy excited states can be reached by applying the appropriate excitation operator.

| ![image](https://user-images.githubusercontent.com/98838428/227925733-e38d0fd4-8536-49c0-a07e-df1f7f1c2d75.png) |
|:--:| 
| Table 1: Results from the variational search for eigenstate energies, and the corresponding excitation operators.|

The Iterative Phase Estimation Algorithm (IPEA) was then implemented to try and refine the energy estimates. Despite the code working for a test unitary, it was not able to converge to correct eigenvalues under the Trotterised $\mathrm{H_2}$ Hamiltonian. There are two possible reasons for this: 

(1) The precision (number of binary bits) of the eigenphase returned by IPEA requires an exponentially large number of gates. It quickly becomes computationally expensive to repeatedly multiply the 4 qubit ($16\times16$) unitary corresponding to the Trotterised time evolution under $\hat{H}$. This only allowed a low precision implementation of IPEA. 

(2) The eigenstates of the unitary corresponding to the Trotterised time evolution under $\hat{H}$ were not particle conserving. This meant that the state provided by the variational search could not probabilistically converge to the correct eigenstate, as it could not be decomposed into a sum of these eigenstates. 

## Conclusion
The WAVES protocol was implemented to find the ground and excited states of the $\mathrm{H_2}$ molecule. The variational search returned energy estimates with upwards of $97$% fidelity. 

IPEA was implemented correctly on a test unitary but it failed to refine the previous energy estimates. Ideally sparse matrices should be used to speed up computational time. With higher precision the IPEA should converge to the correct eigenphase. Trotter error in the controlled time evolution should also be considered - The number of trotter steps should be increased to mitigate this. 

## References
![image](https://user-images.githubusercontent.com/98838428/227926169-1b6b3f7b-3e6d-4450-b7c6-11a6a4458194.png)
