# SNUS_WQAI_Project
This is the project work under Womanium Quantum+AI 2024 program  
In this experiment we will explore utility scale work by simulating the dynamics of a large Heisenberg spin chain. The goal is to measure the dynamics of $Z_i$ for a given site as a function of time and external field $h$ for two different phases of the spin chain.

This lab will be broken into sections matching the Qiskit Patterns framework which include the following steps:

1. Map the system to quantum circuits and operators
2. Optimize the circuits to be executed
3. Execute the time evolution circuits
4. Analyze or post-process the results

This lab also divided into 8 excercises:  

### Introduction
The Heisenberg model, introduced in the late 1920s, is a popular model used to study magnetic phenomena and phase transitions in many-body systems. It is related to its more simplified cousin, the Ising model, and examines the dynamics that emerge from what is known as the *exchange interaction*. This interaction arises from a combination of the Pauli exclusion principle and the Coulomb interaction [[1]](https://doi.org/10.1119/1.4798343) and has a Hamiltonian of the form:

$$ H = \sum_{i=1}^N\left(J_x X_iX_{i+1} + J_y Y_iY_{i+1} + J_z Z_iZ_{i+1}\right). $$

Here $N$ is the number of sites in our chain and $X_i$, $Y_i$, and $Z_i$ are the Pauli operators which act on the $i^{th}$ site. The parameters $J_x$, $J_y$, and $J_z$, represent the coupling strength for the Coulomb ($J_x$ and $J_y$) and Ising ($J_z$) interaction. For the rest of the lab we'll consider $J_x=J_y=1$ for simplicity. 

In general this model has a few phases based on the ratio $\Delta = J_z/J$ (also known as the anisotropy). In this lab we will explore two of them:
- The **istropic** phase where $\Delta = 1$. This is also known as the **XXX** phase.
- The **anisotropic** phase when $\Delta \neq 1$.

To measure some interesting dynamics of this system, we will also introduce a transverse magnetic field with a strength $h$ which will interact with each site through the Pauli $X$ operator. The Hamiltonian of our spin chain will now take the form:

\begin{align} 
H =& \sum_{i=1}^N\left(J X_iX_{i+1} + J Y_iY_{i+1} + J_z Z_iZ_{i+1} + hX_i\right) \\
    =& \sum_{i=1}^N\left(X_iX_{i+1} + Y_iY_{i+1} + \Delta Z_iZ_{i+1} + hX_i\right)
\end{align}

where we have substituted $\Delta = J_z/J$.  

## Step 0: Setup

The code cells including in the step 0 will install all the required packages needed for this experiment.  

## Step 1: Map the system to quantum circuits and operators


In this lab we will examine the dynamics of the expectation value $\langle Z_i \rangle$ averaged over each site as a function of the field strength $h$. In the experiments below, you will prepare a circuit which implements the time evolution operator acting on the state $|000...0\rangle$ according to the system parametersFor simplicity we will should set $\delta t = \frac{\pi}{4}$. This ensures that $\theta_J = -2J_z\delta t = -\frac{\pi}{2}$ which makes the time evolution of our system much simpler for the isotropic phase.

In this first exercise, you will create a function to generate the Hamiltonian in the form of a `SparsePauliOp` object. We'll introduce the modules you'll need, define some system parameters, and demonstrate a quick example of how you might want to accomplish this task.  

We'll first begin by defining the parameters for our system. This includes the number of sites, anisotropy $\Delta$, external field $h$, and evolution time $\delta t$.  
Next we can define our chain using some of the built-in methods of the `CouplingMap` object. Let's use the `from_ring()` method to create a system with periodic boundary conditions.  
With our lattice defined, we'll next create our Hamiltonian using the `PauliOp.from_sparse_list()` method. This function takes in a list of 3-tuples containing information about the operator, qubit index, and coefficient. To create this list we'll iterate over each edge in our lattice map.  

### Exercise 1. Write a function to generate a Hamiltonian

In this first exercise you'll create a function to build a Hamiltonian for a system with **closed** boundary conditions which takes the system parameters and system size as arguments.

*Hint: To create a coupling map with closed boundary conditions, use the [`CouplingMap.from_line()`](https://docs.quantum.ibm.com/api/qiskit/qiskit.transpiler.CouplingMap#from_line) method.*  

### Generate the time evolution circuit

Now that we have the ability to quickly define a Hamiltonian, we next need to consider how to generate the time evolution operator and execute it using a `QuantumCircuit` object. First, recall that the time evolution operator of a static Hamiltonian has the form:

\begin{align} 
    U(t) =& e^{-iHt},
\end{align}

and if we plug in the Hamiltonian of *our* system, the time evolution operator then appears as
$$ U(t) = \exp\left(-it\sum_{i=1}^NX_iX_{i+1} + Y_iY_{i+1} + \Delta Z_iZ_{i+1} + hX_i\right) $$

Next we need to consider how to decompose this into a linear combination of gates which can be executed using a quantum computer. In general, the time evolution operator must be approximated and, conveniently, the Qiskit SDK has some built-in functionality to assist us in generating this. We will use the `PauliEvolutionGate` object and `LieTrotter` synthesis class to use the Li-Trotter approximation for $U(t)$.

The code below accomplishes a few steps to help understand the workflow:
1. Generate the `SparsePauliOp` from the function you created above.
2. Create a `PauliEvolutionGate` representing the action of $U(t)$ for the Hamiltonian and specified time parameter.
3. Instantiate a new `LieTrotter` object in order to synthesize the approximated time evolution.
4. Apply the `synthesize()` mtehod on the `PauliEvolutionGate` defined and draw the resulting circuit




## Step 2: Optimize your circuits

Now that we have a way to generate the time evolution circuits, we should consider how we can optimize them in order to minimize the amount of noise we will observe when running on real hardware.

### Exercise 2. Refactor time evolution operators into entangling layers

Upon examining the time evolution circuits that are generated using the `LieTrotter` or `SuzukiTrotter` synthesis functions, you may notice that they produce a sequence of gates which leave the qubits idle for long periods of time. To optimize the execution of these circuits, we can reorder the two-qubit rotations into two entangling layers acting on even and odd pairs of qubits. An example of what a single layer of this circuit looks like is shown below:  


In this next exercise, create a new function called `build_layered_hamiltonian` which will create a `SparsePauliOp` objects which is a sum of operators acting on even pairs of qubits and odd pairs of qubits. do so 

*(Hint: follow the same steps as before, but include a check to see if you are inspecting an even or odd qubit pair when iterating over the graph.)*  

### Exercise 3. Transpile your circuit

Now that the time evolution circuit has been optimized, we'll next prepare it for execution on quantum hardware. For this exercise, build a staged passmanager and transpile the time evolution circuit using the backend obtained from the following code cell. Once you've obtained a layout for the circuit, apply it to the observables we created above using the `apply_layout()` method. 

*Note: You'll need to select a backend using a `QiskitRuntimeService()` object and use if for the pass manager.*  



## Step 3: Execute on a backend

Now we'll put our transpiled circuit together alongside the operators we have defined. We will prepare a set of circuits to run in a `Batch` where we will measure $Z$ as a function of the external field $h$.

Also recall that we will use $\delta t = 5\pi/4$ in order for the $ZZ$ gates to rotate by $-\pi/2$.

We should also try to execute this for a few different phases of the chain/lattice.

First we will instantiate a set of error mitigation options for the estimator primitive:  

### Exercise 4. Prepare the circuits to execute

Below a set of system parameters has been provided for you to perform your time evolution experiment. You will simulate the time evolution of a 50-site Heisenberg chain and measure the expectation value $\langle Z\rangle$ of each site. You'll need to create a set of circuits to accomplish this for the two different phases specified in the `anisotropies` dict defined below. These two phases to simulate are the *Anisotropic* phase ($\Delta = -5$) and the *XXX* phase ($\Delta = 1$). Also recall that we will use $\delta t = 5\pi/4$ in order for the $ZZ$ gates to rotate by $-\pi/2$.

To complete this exercise, you'll need to create several dictionaries containing the Hamiltonians, time evolution operators, transpiled time evolution circuits, and the observables to measure. The lab will assume that each of these dictionaries has two keys: `Anisotropic` and `XXX` and that the value associated with each key will store a list containing elements of the associated data type.

The following dictionaries to create are:
1. A dictionary of hamiltonians built using the `build_layered_hamiltonian` function you created earlier for each value of external field $h$ specified in the variable `h_vals`. The list for each key should contain the Hamiltonians for each value from the $h_{vals}$ list.
  - The format of this data structure should be `Dict{ key_corresponding_to_phase : List_over_hvals[ SparsePauliOp ]}`
3. A dictionary containing lists of time evolution operators generated from the Hamiltonians you created in the previous step.
  - The format of this data structure should be: `Dict{ key_corresponding_to_phase : List_over_hvals[ PauliEvolutionGate ]}`
4. A dictionary containing lists of quantum circuits synthesized using the `LieTrotter.synthesize()` method, passing in the time evolution operators defined above as well as the `Parameter` value, `dt`.
  - The format of this data structure should be: `Dict{ key_corresponding_to_phase : List_over_hvals[ QuantumCircuit ]}`
5. A dictionary containing lists of the transpiled circuits
  - The format of this data structure should be: `Dict{ key_corresponding_to_phase : List_over_hvals[ QuantumCircuit ]}`
6. A dictionary containing lists of `SparsePauliOp` operators to use as the set of observables to measure.
   - The format of this data structure should be:
   - `Dict{ key_corresponding_to_phase : List_over_hvals[ List_over_sites [ SparsePauliOp ] ]}`


### Exercise 5. Build the PUBs to execute on hardware

Now that the circuits to execute and observables to measure have been prepared, we'll next need to create the Primitive Unified Blocs (PUBs) in order to submit these jobs to quantum hardware. Each PUB is a tuple containing the circuit to execute in the first element, a list of observables to measure in the second, and a list of parameter values to set (in our case this is $\delta t$.) You can read more about the format of PUBs in the [documentation](https://docs.quantum.ibm.com/run/primitives#interface-changes).  

### Exercise 6: Use the `Batch` execution mode to execute your circuits

Since the circuits we will run do not depend on one another, we can submit these to the backend using the `Batch` execution mode. Here you will submit each circuit as its own job within the Batch context and save the associated job id to a dictionary to be stored as a JSON file for later parsing.

To accomplish this, I suggest iterating over the list of pubs for each phase and submitting each pub individually.  

Once these jobs are submitted, you'll then want to save the job ids to a file so that you can retrieve the data once they are complete.  

## Step 4: Post-Process

In these last two exercises you will retrieve and post-process your results.

### Exercise 7: Retrieve your job results
For this exercise you'll use the `service.job(job_id)` method in order to retrieve your jobs by job id. You can obtain these ids either from the dictionary you defined earlier or by loading the json file which you saved them to. The lab assumes that the data you retrieve will be in the form of a dictionary whose keys associate with each phase you simulated and corresponding values contain a list of the job data.  

That's It ! 
Good Luck !



