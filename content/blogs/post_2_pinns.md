Title: PINNs
Date: 2026-06-14
Category: blog
Author: Chris Blais
Summary: A simple demonstration of physics informed neural networks

# Non-Regularized NN



# Data regularization
Physics informed neural networks are a special case of regularization, so we will start by learning that concept. At it's most basic, regularization prevents overfitting. A model with many parameters can fit almost any dataset near perfectly, but this sort of naive approach can lead to a model that is unphysical. To paraphrase Enrico Fermi, with enough arbitrary parameters, you can fit an elephant, and with one more you can make it wiggle it's trunk[^fn1][^fn2]. We would like to avoid overfitting, thus we regularize. 




## References
[^fn1]: http://neuralnetworksanddeeplearning.com/ (book where I heard fermi quote)
[^fn2]: https://www.nature.com/articles/427297a (fermi quote about four parameter elephant)
[^fn3]: https://maziarraissi.github.io/PINNs/
[^fn4]: https://medium.com/@theo.wolf/physics-informed-neural-networks-a-simple-tutorial-with-pytorch-f28a890b874a
