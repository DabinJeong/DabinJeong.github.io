---
layout: post
title:  "Constraint-based model"
date:   2019-02-28
excerpt: "How to model metabolic network: constraint-based model"
tag:
- metabolism
- constraint-based model
- CBM
comments: true
---

<h1> Constraint-based model for metabolic modeling </h1>
<hr>
What we eat consists us. The chocolate I ate breaks down into carbohydrates when passing through digestive organs, and cell absorb carbohydrates to make energy or to make building blocks for the body. In this sense, metabolism is the way of organisms interacting with the environment. 

Thus, modeling metabolism would improve our understanding on metabolism. Here I will give you brief & easy introduction of one of the modeling method: Constraint-based model.   


<br><br>
<h2> Why should we model metabolism with constraint-based model? </h2>

[Objective] To understand interactions with biological and environmental components
<ul>
    <li>Previous approaches & drawbacks</li>
        <ul>
            <li>Mathematical modeling</li>
                : Lack of dynamics 
            <li>Dynamic modeling data</li>
                : a type of mathematical modeling
                : Lack of experimental data (eg. kinetic parameters, total enzyme concentrations) required for modeling
        </ul>
    <li> Constraint-based modeling </li>
        : Compensate the defects of previous models
        <ul> <li> Only stoichiometric matrix is required (easily retrieved from metabolic network)</li> </ul>
        : Certain constraints and assumptions (eg. steady state operation) are imposed on stoichiometric metabolic network to limit the behavior of the system --> yields the range of solutions (solution space)
</ul>

<br><br>

<h2> What is constraint-based model? </h2>
<figure>
	<img src="https://i.imgur.com/zQb6QF3.jpg">
</figure>

For decades, metabolic network has been constructed from  biological databases, literatures, and experiments. This shows how metabolites interact with each other and the ratio between the amounts of metabolites involved in reactions. From the ratio, stoichiometric matrix can be built. It's just mathematic representation of the network.

<u>To model metabolic network means to figure out the distribution of reactions involved in the network.</u> For example, if we find out that R1 is twice faster than R2 and R3 is consistent, the relationship between the reactions is clarified. The distribution (rate) of reactions is called flux distribution (flux vector). Our goal is to find out the range of flux distribution. From stoichiometric matrix, there are infinite number of combinations of reaction rates that can be represented as N-dimensional space (if there are N reactions in the network). 

Fortunately, there are constraints to reduce the solution space. Those constraints are from experiments, from network topology, or from assumption(eg. steady state assumption, mass balance assumption) in chemistry/physics. Following figure shows how solution space is restricted to give more precise solution space.

<figure>
	<img src="https://i.imgur.com/fEdmOHv.png">
</figure>

However, the number of solutions is still infinite after the restriction of solution space. To retrieve one or few solutions, one more assumptions is adopted: "A cell is striving to achieve a metabolic objective.<sup>2</sup>" Objective function is defined as the linear combination of flux variables. Each reaction has its own meaning (eg.Uptake rates, gene knockout). Assume that reaction R1 refers to "metabolite uptake". If the objective of the cell is to maximize the uptake rates, the cell is going to adjust its metabolism to maximize the rate of R1. Here, objective function is the rate of R1, which can be restated as linear combination of flux variables of all the reactions where all other coefficients of reactions except for R1 are set to 0. Using this method, it is possible to get optimal solutions that maximizes a objective function among the infinite number of solutions in solution space. 

<br><br><br>
<h3> Reference </h3>
1. OD kim. <i>et al.</i> <a href="https://www.frontiersin.org/articles/10.3389/fmicb.2018.01690/full"> A Review of Dynamic Modeling Approaches and Their Application in Computational Strain Optimization for Metabolic Engineering. </a> Front Microbiol. <b>9</b>. 1690. (2018)
<br>
2. A Bordbar. <i>et al.</i> <a href="https://www.nature.com/articles/nrg3643"> Constraint-based models predict metabolic and associated cellular functions.</a> Nat Rev Genet. <b>15</b>, 107-120. (2014)
