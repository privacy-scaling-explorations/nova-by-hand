# Nova Folding Introduction

These articles were written by Yugo through the [PSE’s acceleration program](https://github.com/privacy-scaling-explorations/acceleration-program). I would like to thank CPerezz and Pierre for their advice as I was studying and writing my article.

# 1 Introduction
In the recent development of Zero-Knowledge Proof technology, efficiency has been crucial for practical applications, such as zkVM and zkML. A major advancement in recent years is the concept of recursion, enabling the verification of a prior SNARK proof while proving additional statements.

However, an innovative alternative has emerged with Nova Folding scheme. Nova Folding scheme uniquely compresses numerous NP constraints(R1CS) and its instance-witness pairs into a single variant R1CS. It even doesn't need to verify a prior SNARK proof at each recursion step.

[Nova: Recursive Zero-Knowledge Arguments from Folding Schemes](https://eprint.iacr.org/2021/370)

The Folding scheme experienced a significant evolution in 2023, marked by the development of SuperNova, HyperNova, Protostar, and similar schemes.

This article aims to demystify why Nova Folding approach is superior to ordinary recursion schemes and to provide an intuitive, yet thorough understanding of how Nova Folding works.

This series consists of three parts.
* [Part 1](/part_1.md): The concept of Incremental Verifiable Computation, the pros and cons of the ordinary recursion scheme, and introduction to the Nova Folding Scheme. 
* [Part 2](/part_2.md): The mechanism of the Folding Scheme and Nova.
* [Part 3](/part_3.md): The foundations of elliptic curves to understand the Cycle of Curves and how it works in Nova.

