Title: PINNs
Date: 2026-09-20
Category: blog
Author: Chris Blais
Summary: A simple demonstration of physics informed neural networks

Physics informed neural networks, or PINNs, are a way to coerce a neural network to give answers that agree with physical and chemical constraints. Essentially, they are a neural network with a custom loss function. This is an oversimplification, and excludes existing work where the backwards differentiation is leveraged to build constraints into hidden layers[^fn0], but for this example we will start with the simpler loss function definition. 

To put this into practice, we will use a reaction model, and attempt to predict behavior using a conventional neural network, contrasted with a PINN. 

# Reaction Model
We can start our example with a simple chemistry model, with 2 irreversible reactions and 3 reactants, A, B, and C. 

\begin{equation}
    Acc = In - Out + Gen - Cons
\end{equation}

\begin{equation*}
A \xrightarrow{k_{1}} B
\end{equation*}

\begin{equation*}
B \xrightarrow{k_{2}} C
\end{equation*}

\begin{align*}
C_{A} &= C_{A,0}e^{-k_{1}t} \\
C_{B} &= k_{1}C_{A,0}\left[\frac{ e^{-k_{1}t} - e^{-k_{2}t} }{k_2 - k_1}\right] \\
C_{C} &=  \frac{C_{A,0}}{k_{2} - k_{1}}[k_{2}[1-e^{-k_{1}t}] - k_{1}[1-e^{-k_{2}t}] ]
\end{align*}

lets make a plot of these functions:
![Concentration Profiles](images/Conc_plt.png)

We can add a homoscedastic error to each concentration:
![Concentration error](images/conc_err.png)

To start, we will focus on concentration A. We will split the data 50/50 for training and validation: 
![Concentration error](images/ca_tv_split.png)

# Non-Regularized NN
Lets start as simple as possible. We will use a linear model to start and make it more complex as we go.

![Linear Model](images/linear_nn.png)

![Simple NN](images/crappy_nn.png)


# Note on data regularization
Physics informed neural networks are a special case of regularization, so we will start by learning that concept. At it's most basic, regularization prevents overfitting. A model with many parameters can fit almost any dataset near perfectly, but this sort of naive approach can lead to a model that is unphysical. To paraphrase Enrico Fermi, with enough arbitrary parameters, you can fit an elephant, and with one more you can make it wiggle it's trunk[^fn1][^fn2]. We would like to avoid overfitting, thus we regularize. 




## References
[^fn0]: H. Chen, G. E. C. Flores, and C. Li, “Physics-informed neural networks with hard linear equality constraints,” Computers & Chemical Engineering, vol. 189, p. 108764, Oct. 2024, doi: 10.1016/j.compchemeng.2024.108764.
[^fn1]: http://neuralnetworksanddeeplearning.com/ (book where I heard fermi quote)
[^fn2]: https://www.nature.com/articles/427297a (fermi quote about four parameter elephant)
[^fn3]: https://maziarraissi.github.io/PINNs/
[^fn4]: https://medium.com/@theo.wolf/physics-informed-neural-networks-a-simple-tutorial-with-pytorch-f28a890b874a
