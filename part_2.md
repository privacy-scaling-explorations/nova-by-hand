# Nova Folding Introduction Part 2

In part(2), I describe, step by step, how to construct the Nova Folding Scheme.

# 1 Nova Folding
As briefly mentioned in the previous part, Nova folding scheme is designed to fold two NP instances (especially R1CS instances) into one.
So let's begin by reviewing the R1CS.

# 2 R1CS
## 2.1 Definition of R1CS
R1CS (Rank-1 Constraint System) addresses tasks in the NP complexity class, which are challenging to solve but relatively easier to verify once a solution is found. In the context of ZKP, R1CS is a mathmatical representation which is converted from  the complex computational tasks. The R1CS is $Az \odot Bz = Cz$, where $A$, $B$, and $C$ are coefficient matrices, and $z$ is the instance vector. This vector $z$ includes the witness $W$, public input/output $x$, and the constant $1$, making $z=[W, x, 1]$.

## 2.2 Constructs the R1CS from equation
### 2.2.1 Statement
Here, we aim to prove that we know a number pair $x$ and $y$ that satisfies the following equation, without revealing the value of $x$:
$$x^3 + x + 5 = y$$
In this scenario, we consider $x$ as the witness and $y$ as the output, thus modifying the equation to:
$$\omega^3 + \omega + 5 = \text{Out}$$

This formula uses almost same one cited in Vitalik's [Quadratic Arithmetic Programs: from Zero to Hero](https://medium.com/@VitalikButerin/quadratic-arithmetic-programs-from-zero-to-hero-f6d558cea649) as an example.


### 2.2.2 Flattening
The Flatting is to break down into the format of $c=a (+\, \text{or} \, \times) b$. Each flattened equation becomes the Constraint Equations.

$$
\begin{aligned}
\text{sym}_1 &= \omega \times \omega \\
y &= \text{sym}_1 \times \omega \\
\text{sym}_2 &= y + \omega \\
\text{Out} &= \text{sym}_2 + 5 \\
\end{aligned}
$$

### 2.2.3 R1CS
Next, we converts the flattened equation to the R1CS matrices. 
The R1CS is represented as $Az \odot B z = Cz$, and $z$ can be represented as $z=[W, x, 1]$, here, $z$ becomes $z = [\omega, \text{sym}_1, y, \text{sym}_2, \text{out}, 1]$. Then, we can find the coeeficient matrices of the R1CS.

The R1CS for $x^3 + x + 5 = y$ is structured as:


$$
\begin{bmatrix}
1 & 0 & 0 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 & 0 & 0 \\
1 & 0 & 1 & 0 & 0 & 0 \\
0 & 0 & 0 & 1 & 0 & 5
\end{bmatrix} \cdot \begin{bmatrix}
\omega \\
\text{sym1} \\
y \\
\text{sym2} \\
\text{out} \\
1
\end{bmatrix} \odot \begin{bmatrix}
1 & 0 & 0 & 0 & 0 & 0 \\
1 & 0 & 0 & 0 & 0 & 0 \\
0 & 0 & 0 & 0 & 0 & 1 \\
0 & 0 & 0 & 0 & 0 & 1
\end{bmatrix} \cdot \begin{bmatrix}
\omega \\
\text{sym1} \\
y \\
\text{sym2} \\
\text{out} \\
1
\end{bmatrix} = \begin{bmatrix}
0 & 1 & 0 & 0 & 0 & 0 \\
0 & 0 & 1 & 0 & 0 & 0 \\
0 & 0 & 0 & 1 & 0 & 0 \\
0 & 0 & 0 & 0 & 1 & 0
\end{bmatrix} \cdot \begin{bmatrix}
\omega \\
\text{sym1} \\
y \\
\text{sym2} \\
\text{out} \\
1
\end{bmatrix}
$$


In this statement, there are many instance vectors $z$ possible, as various pairs of $x, y$ satisfy $x^3 + x + 5 = y$.
For instance, when $(x, y) = (2, 15)$, $z$ is $[2, 4, 8, 10, 15, 1]$. Similarly, for $(x, y) = (3, 35)$, $z$ is $[3, 9, 27, 30, 35, 1]$.

## 2.3 R1CS in IVC
Nova is one of the IVC, so we suppose tha IVC has the function $F$ of the $x^3 + x + 5 = y$ and it's R1CS. This is the IVC, so the output $y$ at step $i$ becomes the input $x$ at the step $i+1$. 

For instance,
Step 1: $(x, y) = (1, 7)$, $z$ is $[1, 1, 1, 2, 7, 1]$
Step 2: $(x, y) = (7, 355)$, $z$ is $[7, 49, 343, 350, 355, 1]$
Step 3: $(x,y)=(355,44739235)$, $z$ is $[355,126025,44738875,44739230,44739235,1]$.

This means the IVC maintains the same R1CS coefficient matrices $A, B, C$ throughout the entire steps, but there will be different instance vectors $z_i$.

Thus, the goal of Nova Folding is to fold two instance vectors $z_1, z_2$ into a single vector $z_{(1,2)}$. After IVC should fold the $z_{(1,2)}$ and $z_3$ into $z_{((1,2),3)}$, continue to fold following until the last.

![r1cs folding](/images/part2_r1csfolding.png)

This illustration is an intuitive representation of Folding. Let's look at the specifics. The actual Nova Folding does Folding of an Augmented Circuit.


# 3 First Attempt: Try Folding to two R1CS
## 3.1 Random linear Combination
The most direct approach to folding $Z_1$ and $Z_2$ is to take a random linear combination with $r$:
$$
Z = Z_1 + r \cdot Z_2
$$

Ideally, we hope that:

$$
\begin{align*}
A \cdot (Z_1 + r \cdot Z_2) \odot B \cdot (Z_1 + r \cdot Z_2) = C \cdot (Z_1 + r \cdot Z_2)
\end{align*}
$$

However, the $Z$ that results from applying the random linear combination does not satisfy the R1CS structure, $A \cdot Z \odot B \cdot Z = C \cdot Z$.

Since $z=[W, x, 1]$, we can express:

\begin{align}
Z_1 &= [W_1, x_1, 1] \\
Z_2 &= [W_2, x_2, 1]
\end{align}

Then $Z = Z_1 + r \cdot Z_2$ becomes:
$$ Z = [W_1 + r \cdot W_2, x_1 + r \cdot x_2, 1 + r \cdot 1] $$

Now, expanding $A \cdot Z \odot B \cdot Z$ is below, but it is not equal to the $C \cdot Z=C \cdot (Z_1 + r \cdot Z_2)$

$$
\begin{align*}
A \cdot Z \odot B \cdot Z &= A \cdot (Z_1 + r \cdot Z_2) \odot B \cdot (Z_1 + r \cdot Z_2) \\
&= A \cdot Z_1 \odot B \cdot Z_1 + r \cdot (A \cdot Z_1 \odot B \cdot Z_2 + A \cdot Z_2 \odot B \cdot Z_1) + r^2 \cdot (A \cdot Z_2 \odot B \cdot Z_2) \\ &= C \cdot Z_1+r^2 (C \cdot Z_2)+r \cdot (A \cdot Z_1 \odot B \cdot Z_2 + A \cdot Z_2 \odot B \cdot Z_1) 
\\
&\neq C \cdot Z.
\end{align*}
$$

These are the causes.:
1. The cross term $r \cdot (A \cdot Z_1 \odot B \cdot Z_2 + A \cdot Z_2 \odot B \cdot Z_1)$ appears unexpectedly, complicating the equation.
2. The presence of the $r^2$ term also adds complexity not accounted for in the original $C \cdot Z$ equation.
3. The scalar multiplication with $r$ alters the constant term, $1$, into $r$ in the vector $Z$, $Z = [W_1 + r \cdot W_2, x_1 + r \cdot x_2, 1 + r \cdot 1]$ and deviating from the standard format expected in a valid R1CS instance.


## 3.2 How can we solve this R1CS isuue?
To overcome the challenges from the first try, we have to adopt a new strategy in the second attempt by introducing two key elements:

- Introduction of an Error Vector $E$: Introduce an error vector $E$ to absorb the cross terms, $r \cdot (A \cdot Z_1 \odot B \cdot Z_2 + A \cdot Z_2 \odot B \cdot Z_1)$ , from folding $Z_1$ and $Z_2$.

- Introduction of a Scalar $u$: To address the issues with the $r^2$ term and the altered constant in $Z$, we introduce a scalar $u$. This scalar adjusts for the factor of $r$ in $CZ_1 + r^2 \cdot CZ_2$ and the modified constant term in $Z = (W, x, 1 + r \cdot 1)$.

This approach leads to a variant of R1CS, termed Relaxed R1CS.

# 4 Relaxed R1CS

## 4.1 Relaxed R1CS
### 4.1.1 Definition of Relaxed R1CS

In Relaxed R1CS, the original R1CS is  adjusted to accommodate the additional terms introduced by $E$ and $u$. The Relaxed R1CS can be expressed as follows:

$$ (A \cdot Z) \odot (B \cdot Z) = u \cdot (C\cdot Z) + E$$

Here, $E$ serves to relax the cross-term effects. The scalar $u$ is introduced to adjust for the $r^2$ term and the altered constant in $Z$, ensuring the equation's balance and alignment.

This modification allows the linear-combinationed vector $Z$, resulting from the folding process, to still satisfy the Relaxed R1CS. Note that any instance of a Relaxed R1CS can be resented as R1CS by adopting $u = 1$ and $E = 0$. So Relaxed R1CS is still  the property of NP-completeness.

### 4.1.2 Mathematical Formulation of the Relaxed R1CS
From the definition of relaxed R1CS, we have:

$$
\begin{align*}
(A \cdot Z_i) \circ (B \cdot Z_i) &= u_i \cdot (C \cdot Z_i) + E_i \\
\text{1st instnace:} Z_1 &= (W_1, x_1, u_1), E_1 \\
\text{2nd instnace:}Z_2 &= (W_2, x_2, u_2), E_2
\end{align*}
$$

By choosing $r$ and performing a linear combination of two instances as follows, we can fold them into a single instance:

$$
\begin{align*}
E &= E_1 + r \cdot (A Z_1 \odot B Z_2 + A Z_2 \odot B Z_1 - u_1 \cdot C Z_2 - u_2 \cdot C Z_1) + r^2 \cdot E_2\\
u &= u_1 + r \cdot u_2 \\
x &= x_1 + r \cdot x_2 \\
W &= W_1 + r \cdot W_2 \\
\end{align*}
$$


Consequently, $Z = (W, \mathsf{x}, u)$ becomes a combination of $Z_1$ and $Z_2$:

$$\begin{align*}
AZ \odot BZ &= A(Z_1 + r\cdot Z_2) \odot B(Z_1 + r\cdot Z_2) \\
&= AZ_1 \odot BZ_1 + r \cdot(AZ_1 \odot BZ_2 + AZ_2 \odot BZ_1) + r^2 \cdot (AZ_2 \odot BZ_2 )\\
&= u_1\cdot CZ_1 + E_1 + r \cdot (AZ_1 \odot BZ_2 + AZ_2 \odot BZ_1) + r^2 \cdot (u_2 \cdot CZ_2 + E_2) \\
&= u_1 \cdot CZ_1 + r \cdot u_2 \cdot CZ_1 - r \cdot u_2\cdot CZ_1 + E_1 + r \cdot (AZ_1 \odot BZ_2 + AZ_2 \odot BZ_1) + r^2 \cdot (u_2 \cdot CZ_2 + E_2) + r \cdot u_1 \cdot CZ_2 - r \cdot u_1 \cdot CZ_2 \\
&= (u_1 + r \cdot u_2) \cdot CZ_1 + E_1 -r \cdot u_2 \cdot CZ_1 + r \cdot (AZ_1 \odot BZ_2 + AZ_2 \odot BZ_1) + (u_1+ r \cdot u_2) \cdot r \cdot CZ_2 + r^2 \cdot E_2 - r \cdot u_1 \cdot CZ_2 \\
&=(u_1 + r \cdot u_2) \cdot C (Z_1 + rZ_2) + r \cdot (AZ_1 \odot BZ_2 + AZ_2 \odot BZ_1 - u_2 \cdot CZ_1 -u_1 \cdot CZ_2) + E_1 + r^2 \cdot E_2 \\
&= u \cdot (CZ) + E
\end{align*}$$

## 4.2 Prover and Verifier Actions in Relaxed R1CS

This section describes how Prover and Verifier interact to build the Relaxed R1CS, although this is a naïve approach.

### 4.2.0 What do they have by begin?
![4.2.0](/images/part2_4.2.0.png)

At first, the Prover has an instance-witness pair for the Relaxed R1CS, denoted as $((E_i,u_i,x_i),W_i)$, and the Verifier knows the only instance $(E_i,u_i,x_i)$ for it.

### 4.2.1 Prover sends the witness to Verifier
![4.2.1](/images/part2_4.2.1.png)

The Prover must send witnesses $W_1, W_2$ to the Verifier so that both have the instance-witness pairs of the Relaxed R1CS.

### 4.2.2 Verifier chooses r and sends it to Prover
![4.2.2](/images/part2_4.2.2.png)

The Verifier chooses $r$ to perform a random linear combination and then send it to the Prover.

### 4.2.3 Prover and Verifier take the random linear combination
![4.2.3](/images/part2_4.2.3.png)

Both the Prover and the Verifier perform the random linear combination using $r$.

### 4.2.4 Prover and Verifier get the new instance-witness pair
![4.2.4](/images/part2_4.2.4.png)

They obtain the new folded instance-witness pair of the Relaxed R1CS, denoted as $(E, u, x, W)$.

### 4.2.5 Prover and Verifier check the satisfacation of the Relaxed R1CS
![4.2.5](/images/part2_4.2.5.png)


Finally, the Prover and the Verifier can check the satisfaction of the Relaxed R1CS.

## 4.3 Challenges in Relaxed R1CS
### 4.3.1 Problems
The Relaxed R1CS approach, while folding two instances into one, remains a few challenges:

- Non-Triviality: In the initial scheme, the prover sends witnesses $(W_1, W_2)$ for the verifier to compute $E$, $(E = E_1 + r \cdot (A Z_1 \odot B Z_2 + A Z_2 \odot B Z_1 - u_1 \cdot C Z_2 - u_2 \cdot C Z_1) + r^2 \cdot E_2)$, which can be very large. This process makes the folding scheme not trivial.

- Lack of Zero-Knowledge: The scheme also fails to provide zero-knowledge properties. Since the witnesses are sent directly to verifier, it compromises the privacy.

### 4.3.2 Solution
To address these issues, the final protocol introduces a new concept: Committed Relaxed R1CS. This variant involves key modifications:

- Use of Homomorphic Commitments: The protocol employs additively homomorphic commitments to both the witness $W$ and the slack vector $E$.

- Treating $W$ and $E$ as the Witness: Both $W$ and $E$ are treated as parts of the witness.

- Single Commitment from the Prover: Instead of sending raw witnesses themselves, the prover sends commitment of the witness. This commitment aids the verifier in computing commitments to the folded witness $(W, E)$ efficiently.

# 5 Committed Relaxed R1CS
Before moving on to the Committed Relaxed R1CS, let's check the what commitemnt scheme is.
## 5.1 Concepts of Commitment Scheme
A commitment scheme allows us to commit to a chosen value while keeping it hidden, emphasizing two core properties: 

1. Hiding: The commitment conceals the original value. It is computationally difficult to deduce the original value from the commitment.
2. Binding: Once a commitment is made, the original value cannot be changed.

### 5.1.1 Pedersen Commitment
The Pedersen Commitment is one of commitment schemes and is used to build Commited Relaxed R1CS.

- Generation Method: Let $P$ and $Q$ be two points on an elliptic curve. The commitment $c(s, r)$, for a secret value $s$ and a random number $r$, is computed using the equation $c(s, r) = sP + rQ$.

- Hiding: It is difficult to derive $s$ from $c$, due to the difficulty of the Elliptic Curve Discrete Logarithm Problem(ECDLP).

- Binding: Changing $s$ is infeasible. Due to the hardness of ECDLP, it is virtually impossible to find different $s'$ and $r'$ such that $c(s, r) = c(s', r')$.

### 5.1.2 Additively Homomorphic Property
Additive homomorphism is a property where arithmetic operations (in this case, addition) performed on encrypted data correspond to the same operations on the original data.

In the Pedersen Commitment scheme:
1. For two values $x$ and $y$, their commitments $c(x, r) = xP + rQ$ and $c(y, s) = yP + sQ$ are created.
2. Adding these commitments results in $c(x, r) + c(y, s) = (x + y)P + (r + s)Q$, forming a new commitment for $x + y$.

## 5.2 Definition of Committed Relaxed R1CS
![5.2](/images/part2_5.2.png)

In the Relaxed R1CS, instance-witness pairs are denoted as $(E_i, u_i, W_i, x_i)$. In terms of the Committed Relaxed R1CS, the witness of the Committed Relaxed R1CS can be defined as $(E_i, r_{E_i}, W_i, r_{W_i})$ and the instance of the Committed Relaxed R1CS can be defined as $(\bar{E}_i, u_i, \bar{W}_i, x_i)$. Here, $r$ is a random number.

# 6 Folding Scheme for Committed Relaxed R1CS

Let's see how to construct Folding scheme for Commited Relaxed R1CS.


## 6.1 Prover and Verifier Actions in the Folding Scheme for Committed Relaxed R1CS

### 6.1.0 What do they have by begin?
![6.1.0](/images/part2_6.1.0.png)

Initially, the Prover has an instance-witness pair for the Relaxed R1CS, denoted as $((E_i,u_i,x_i),W)$, and the Verifier knows the instance $(u_i,x_i)$ because $E_i$ is also treated as witness in addition to $W_i$.

### 6.1.1 Prover commits the witness and sends them to the Verifier
![6.1.1](/images/part2_6.1.1.png)

The Prover has a witness of Commited Relaxed R1CS $(E_i, r_{E_i}, W_i, r_{W_i})$  and takes the Pedersen Commitment of $E_i$ and $W_i$ with random values $r_{E_i}$ and $r_{W_i}$ and then sends the committed witnesses $\bar{E}_i$ and $\bar{W}_i$ to the Verifier. They get the instance of the Committed Relaxed R1CS $(\bar{E}_i, u_i, \bar{W}_i, x_i)$.

### 6.1.2 Prover calculates the cross term and sends it to the Verifier
![6.1.2](/images/part2_6.1.2.png)

The Prover calculates the cross term $T$, commits $T$ with random $r_T$ to get $\bar{T}$, and then sends $\bar{T}$ to the Verifier.

### 6.1.3 Verifier chooses r and sends it to Prover
![6.1.3](/images/part2_6.1.3.png)

The Verifier chooses $r$ for a random linear combination and then sends it to the Prover.

### 6.1.4 Prover and Verifier derive the folded instance of the Committed Relaxed R1CS
![6.1.4](/images/part2_6.1.4.png)

Both Prover and Verifier derive the folded instance of Committed Relaxed R1CS, $(\bar{E}, u, \bar{W}, x)$, by taking random linear combination with $r$.


### 6.1.5 Prover derive the folded witness of the Commited Relaxed R1CS
![6.1.5](/images/part2_6.1.5.png)

Only the Prover needs to derive the folded witness of the Committed Relaxed R1CS, $(\bar{E}, r_E, \bar{W}, r_W)$, by taking random linear combination with $r$.

### 6.1.6 Final Check
![6.1.6](/images/part2_6.1.6.png)

The Prover has the folded witness of Committed Relaxed R1CS and the folded instance of it. The Verifier knows only the folded instance. Since the Pedersen commitment is additively homomorphic, they can check if the Prover folded correctly.

# 7 Non-Interactive Folding Scheme For Committed Relaxed R1CS
Next, I will show you how to convert the interactive Folding Scheme for Committed Relaxed R1CS into a non-interactive one.

## 7.1 Fiat-Shamir Transform
In the Folding Scheme for Committed Relaxed R1CS, the Verifier chooses a random value $r$ and sends it to the Prover. This step is akin to a Public-Coin Protocol, similar to flipping a coin.

This interactive method can be transformed into a non-interactive method using a technique known as the Fiat-Shamir Transformation. Essentially, in the original interactive version, the Verifier sends a random number $r$ to the Prover to ensure unpredictability. 

Conversely, if the protocol can compel the Prover to select a specific value for $r$, then the Prover itself can choose $r$ and carry out the linear combination. 
Intuitively, this means that $r$ is uniquely determined based on the values necessary for the Verifier's verification process.

## 7.2 Apply the Fiat-Shamir Transform in Folding scheme 
In the earlier-described Interactive Folding Scheme, $\bar{T}$ is a value that the Verifier always requires. Therefore, the Prover should uniquely determine $r$ by using a hash function that takes $\bar{T}$ as one parameter.

![7.2](/images/part2_7.2fiatshamir.png)

# 8 Constructing IVC from a Folding Scheme

The previous explanation described a Folding Scheme designed to fold two Committed Relaxed R1CS instance-witness pairs into one. The next section will introduce Nova, a scheme that implements Incremental Verifiable Computation (IVC) using the Folding Scheme. Let's now delve into the review of IVC.

## 8.1 Reviewing the IVC
![review the ivc](/images/part2_reviewingtheivc.png)

In IVC, the Prover aims to demonstrate possession of a function $F$ and an initial input $Z_0$. They want to show that after $n$ steps of applying $F$, where each steps transforms $Z_{i-1}$ into $Z_i$, the final output is $Z_n$.

The verification function can be represented as follows:

$V((n, Z_0, Z_n), \Pi_n) \rightarrow {0, 1}$

This function evaluates to 1 (true) if the proof $\pi$ correctly demonstrates that applying $F$ iteratively $n$ times to $Z_0$ results in $Z_n$, and 0 (false) otherwise.

## 8.2 Augmented Function and Its Constraints
![nova folding](/images/part2_novafolding.png)

In Nova, besides the primary function $F$, the IVC Prover runs an augmented function $F'$. This augmented function comes with an associated augmented constraint system(Augmented Circuit), which is essential for generating the IVC proof $\Pi_i$.

![augmented constraints](/images/part2_augmentedconstraints)


Here, $U_{\text{acc}, i}$ represents the accumulated Committed Relaxed R1CS instances that have been executed correctly from the 1st to the $i-1$ th. $u_i$ indicates that the $i$th step has been executed correctly. Note that $u_i = (\overline{W_i}, \overline{0}, x_i, 1)$ where $\overline{E} = 0$ and $u = 1$ in the committed Relaxed R1CS and $U_i=(\overline{W_i}, \overline {E_i}, x_i, u_i)$.

There are two primary functions of the augmented function $F'$:

1. Incremental Computation: This involves the execution of function $F$, which takes $u_i$ as its input, containing $z_i$, and outputs $Z_{i+1} = F(Z_i)$.

2. Folding Verifier: The NIFS.P folds from $U_{\text{acc}, i}$ and $u_i$ to $U_{\text{acc}, i+1},$. So $F'$ calls the $NIFS.V$ to check it.

Additionally, the Augmented Function $F'$ incorporates Augmented Constraints to generate the IVC Proof.

![augmented instance](/images/part2_augmentedconstraintsinstance.png)


And in IVC, the Prover computes a new instance of $u_{i+1}$, indicating that the computation of $i+1$ in $F'$ has been executed correctly. The component $u_{i+1}.x$ is instantiated by a hash function whose inputs are $i+1$, $Z_0$, $Z_{i+1}$, and $U_{\text{acc}, i+1}$, ensuring it becomes a constant value.

![augmented pic](/images/part2_augmentedconstraintspic.png)


This illustration is a mixture of Figure 4 on P.18 of the Nova paper and [Revisiting the Nova Proof System - Wilson Nguyen](https://youtu.be/h_PU7FZWiQk?si=G5DZYfrlQQi8SG-j).

A hash function serves as a mechanism to ensure the integrity of each step within the computation process. By hashing the result of each step, which includes both the public I/O, we can establish a verifiable link to the subsequent step. This method allows for the confirmation that each phase of the computation is carried out correctly.

Furthermore, it's crucial to avoid the self-referential folding of an instance. To elaborate, when the function $F'$ outputs the accumulated instance $U_{acc, i+1}$, it encounters a nuanced discrepancy. Although $U_{acc, i+1}$ is encapsulated within $u_{i+1}.x$ (i.e., the public I/O of $u_{i+1}$), $F'$ must, in the ensuing step, fold $u_{i+1}.x$ back into $U_{acc, i+1}.x$. This necessitates $F'$ to attempt recursively incorporating $U_{acc, i+1}$ into its subsequent version, engendering a paradox wherein $F'$ is tasked with integrating $U_{acc, i+1}$ into $U_{acc, i+1}.x$, despite $U_{acc, i+1}$ being a fundamental component of $u_{i+1}.x$ already.
To circumvent this issue and prevent direct self-referential folding, $F'$ have to output a collision-resistant hash of its public I/O instead of the I/O directly.


# 9 The role of the Decider
In IVC, a Decider is the verifier who ensures the correctness of computation results by evaluating provided proofs. So here, let's learn how to verify IVC Proof.


## 9.1 IVC verification
![ivc verification](/images/part2_ivcverification.png)

Through the Augmented Constraint System, we obtain the IVC proof $\Pi_i=(u_i, w_i),(U_{\text{acc}, i}, W_{\text{acc}, i})$. The Prover can generate this proof at any desired step. To verify it, follow these steps:

- Check the satisfaction of Relaxed R1CS for $u_i$ and $w_i$.
- Check the satisfaction of Relaxed R1CS for $U_{acc, i}$ and $W_{acc, i}$.
- Check $u_i.(\overline{E}) == \overline{0}$ and $u_i.u == 1$.
- Check $u_i.x == H(i, z_0, z_i, U_{i})$ for consistency.
- Check Pedersen commitments $U_\text{acc,i}.(\overline{E}, \overline{W})$.

## 9.2 Compressing IVC Proofs with zkSNARKs

However, there are challenges with the IVC Proof. The first is the lack of zero-knowledge, as the Prover does not conceal the witness. The second is that the IVC Proof is not succinct. To overcome these issues, it has been proposed that zkSNARKs wraps that prover knows the correct IVC Proof (an IVC proof $\Pi_i$ such that $IVC.V(vk, i, z_0, z_i, \Pi_i) = 1$) within a zkSNARK. The Nova document employs Superspartan for this purpose.

# 10 Reference
- [ZKP MOOC Lecture 10: Recursive SNARKs](https://youtu.be/0LW-qeVe6QI?si=5OHATEeT1EFKNjAS)
- [Revisiting the Nova Proof System - Wilson Nguyen](https://youtu.be/h_PU7FZWiQk?si=A6EgWLesFe3CDSTY)
- [Nova: Recursive Zero-Knowledge Arguments from Folding Schemes](https://eprint.iacr.org/2021/370)
- [R1CS: A Day in the Life of a few Equations](https://learn.0xparc.org/materials/circom/additional-learning-resources/R1CS%20Explainer)
- [(Workshop) [Super] Nova [Scotia]: Unpacking Nova](https://youtu.be/N6RW_YhLMNw?si=7KWZmQ0KOmmIxL49)