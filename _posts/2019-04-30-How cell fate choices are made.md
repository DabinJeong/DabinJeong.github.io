---
layout: post
title:  "How cell fate choices are made"
date:   2019-04-30
excerpt: "[Review] 2019, Nat Biotech, Characterization of cell fate probabilities in single-cell data with Palantir "
tag:

- single cell data
- Markov chain
- cell differentiation
- model of cell fate choice
- Diffusion map

comments: true
---




<h1>[Review] How cell fate choices are made : in a probabilistic perspective</h1>

---

In development, a cell changes into other type of cell with distinct functions and characteristics, which is called cell differentiation. Transition of cell fate is traditionally regarded as a discontinuous process; cell 'magically' turn into the other type. However, it is more appropriate to think that changes in molecular level (eg. epigenetic characteristic) is accumulated in the cell and it makes cell to gradually turn into the other type. As single cell sequencing technologies has been developed, it is possible to detect subtle changes among single-cells so that differentiation can be modeled in a continuous viewpoint. One of the remarkable research is **Palantir algorithm** that model cell differentiation in a continuity, developed by Manu Setty. <i>et al.</i><sup>1</sup>

Here, I will show how the authors built the algorithm in detail.
<br>

***
<h2>1.Construct graph over cells</h2> 
<center><figure>
	<img src="https://i.imgur.com/WClYYvw.jpg">
</figure></center>

As transcriptome represents characteristics of cell, single cell RNA seq (scRNA-seq) data is a powerful tool to identify characteristics of each cell, which is essential for detecting subtle changes among differentiating cells. Firstly, authors built nearest-neighbor graph represented as *G*, where each cell is connected to its most similar cells. However, since scRNA-seq data is high dimensional-low sample data, it is too noisy to detect pattern from graph for further analysis. Here, the author used **diffusion map** to **convert  data into low dimensional data**. This method is called 'diffusion map' because expression vector of each cell is projected onto the diffusion components, which are top eigenvectors of diffusion operator. Diffusion operator is the Laplacian of the affinity matrix. Since eigenvectors of diffusion operator are solutions to the optimization problem which minimize the difference of node attributes of graph, dense region in graph share similar entities and vice versa. Therefore, diffusion components show how dense the graph is. Let's assume that original dimension of gene expression data is *M* which means we measure expression in *M* genes. If we select *L* eigenvectors to represent gene expression such that *L* < *M*, the dimension of expression data for each cell is reduced from *M* to *L* which is even a better representation because density of graph is considered in new embedding.

<center><figure>
	<img src="https://i.imgur.com/ZfCF5XC.jpg">
</figure></center>






***
<h2> 2. Pseudo-time ordering of cells</h2>
<center><figure>
	<img src="https://i.imgur.com/90Lpu9L.jpg">
</figure></center>

Given the initial state cell as a input, **pseudo-time** is defined as **relative distance from the initial state cell**. 

* Firstly, it is necessary to **refine the edge attributes of the graph in the embedded space**. Distance between cells is defined as diffusion distance which use the number of steps, denoted as *t*. Since there is tremendous amount of paths between cells (if network is complicated), it is hard to choose *t* as a parameter. Therefore, **multi-scale distance** is used instead, where *t* is marginalized out.

<center><figure>
	<img src="https://i.imgur.com/UexcRXD.jpg">
</figure></center>

* Another problem is that the noise is accumulated as the distance between cells increases so that it is hard to measure major trends of differentiation. To solve this problem, Palantir **chose waypoint cells which represent major trends of differentiation**, so that pseudo-time is calculated using theses waypoints. To select waypoints for each diffusion component, Palantir iteratively add waypoints to the waypoint set, which "**maximizes the minimum distance to the current waypoints set**"<sup>1</sup>. Firstly, the algorithm randomly choose one cell, called cell *i* for each diffusion component. Distances from cell *i* to all other cells are calculated. Cells with minimum distance from current waypoint is added to the waypoint set. Then, iterate this process. Final waypoints are the union of all waypoints across all diffusion components.

* Finally, let's calculate pseudo-time. Pseudo-time is defined as relative distance from the starting cell. Since waypoints are selected as major trends of differentiation, it is reasonable **to compute distance exploiting the waypoints**. The measure which uses waypoints to compute distance is called waypoint perspective. Waypoint perspective is computed for each waypoint. \\( \tau_{i} \\) represents the distance from the starting cell of pseudo-time to a cell *i*, \\( S^{'} \\) while \\( \tau_{w_{i}} \\) represents the distance to a waypoint \\( w_{i} \\). The distance from a cell *i* to a waypoint \\( w_{i} \\) is represented as \\( D_{w_{i}} \\). Initially, pseudo-time is initialized as  \\( \tau_{i} \\). According to the equation below, perspective of a cell *i* relative to waypoint *w*, denoted as \\( V_{wi} \\) is computed. Pseudo-time is then calculated as a weighted average of waypoint perspectives. These procedure is executed iteratively until the convergence, so that the **psuedo-time is refined**.

  <center><figure>
  	<img src="/_posts/img/post2_Fig2.2_pseudo-time_ordering.jpg">
  </figure></center>

***
<h2> 3. Modeling with Markov Chain </h2>

Markov chain model is a probability based generative model. It is assumed that each data point of sequential data is assigned for certain state and the data can be generated from transition from one state to the other state. Which path to go is chosen based on the transition probability. Since direction of transition is one of the key elements of this model, pseudo-time is used to infer the directionality. 

* Assume that graph constructed using waypoints is denoted as *G'*. Firstly, the graph should be pruned to remove the edges that is not consistent with the pseudo-time order, which means removing de-differentiating path. Considering pseudo-time order and scaling factor, graph *Gd* is constructed where the edges are the distances between nodes and nodes are pruned considering pseudo-time order. Kernel method is then used to convert distances between nodes into affinities and normalized with the sum of affinities of the neighboring nodes. Then, the edge *Pij* of G' represents probability of traversing from cell i to cell j in one step. This term denotes for transition probability of the Markov chain which implies cell state similarity between differentiating cells. If cell i and cell j are distant, the structure of the network affects the overall probability which means that the regulatory process plays a major role in differentiation. If cell i and j are close, the edges between nodes is the key part in calculating overall probability rather than the structure of the network which means the stochastic molecular process plays an important role in differentiation.

<center><figure>
	<img src="https://i.imgur.com/xMDhBMj.jpg">
</figure></center>



* When cell is fully differentiated, the cell became specialized and it loses it's differentiating potential. This state is equivalent to the terminal states in the graph. As terminal states are the cells with little differentiating potential, random walks on graph are converged to the terminal states if differentiation is properly modeled. In this model, terminal state is defined by the nodes that has high probability from steady state distribution. Then, remove the outgoing edges of the terminal states to change the model into the absorbing Markov chain. Absorbing Markov chain is a Markov chain model with some nodes from which it is impossible to leave once entered and these nodes are accessible in every node in the model.  

-----

<h2> 4. Cell fate/differentiation potential characterization </h2>

Once terminal states are defined, it is possible to calculate probability of traversing from certain cell to terminal state cells. As the transition probability is computed in the previous step, probability of traversing from cell i to terminal state (absorbing state) for all k steps can be easily calculated. 

<center><figure>
	<img src="https://i.imgur.com/iy50F4U.jpg">
</figure></center>

After weighted with waypoint perspectives, differentiation potential of each state is defined as the entropy of the probability vector, and it denotes for the degree of uncertainty to reach the final state.

<center><figure>
	<img src="https://i.imgur.com/oY28Xoe.png">
</figure></center>







------


<h3> Reference </h3>

1. Manu Setty. <i>et al.</i> <a href="https://www.nature.com/articles/s41587-019-0068-4"> Characterization of cell fate probabilities in single-cell data with Palantir.</a> Nat Biotech. <b>37</b>. 451-460. (2019)
