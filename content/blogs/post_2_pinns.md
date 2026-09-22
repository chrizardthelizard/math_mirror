Title: PINNs
Date: 2026-09-20
Category: blog
Author: Chris Blais
Summary: A simple demonstration of physics informed neural networks

Physics informed neural networks, or PINNs, are a way to coerce a neural network to give answers that agree with physical and chemical constraints. Essentially, they are a neural network with a custom loss function. This is an oversimplification, and excludes existing work where the backwards differentiation is leveraged to build constraints into hidden layers[^fn0], but for this example we will start with the simpler loss function definition. 

To put this into practice, we will use a reaction model, and attempt to predict behavior using a conventional neural network, contrasted with a PINN. 

# Reaction Model
We can start our example with a simple chemistry model, with 2 irreversible reactions and 3 reactants, A, B, and C. 

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

We can code this up in python:

```python3
import numpy as np
import matplotlib.pyplot as plt
t_i = np.arange(0, 10, 0.1)
Ca0 = 2.5
k1 = 0.5
k2 = 0.51
    
def conc_a(t):
    c = Ca0*np.exp(-k1*t)
    return c

def conc_b(t):
    c = k1*Ca0*(np.exp(-k1*t) - np.exp(-k2*t))/(k2 - k1)
    return c

def conc_c(t):
    c = (Ca0/(k2 - k1)) * (k2*(1-np.exp(-k1*t)) - k1*(1-np.exp(-k2*t)))
    return c

ca = conc_a(t_i)
cb = conc_b(t_i)
cc = conc_c(t_i)

plt.plot( t_i, ca, label="$C_a$ true value")
plt.plot( t_i, cb, label="$C_b$ true value")
plt.plot( t_i, cc, label="$C_c$ true value")
plt.xlabel("Time")
plt.ylabel("Concentration")
plt.legend()
```

Running the code above gets us the following profiles. For convenience, We'll leave it unitless. 
![Concentration Profiles](images/Conc_plt.png)

We can add a homoscedastic error to each concentration. We'll assume normally distributed data, with a standard deviation of 0.05:

```python3
# generate homoscedastic noisy data
ca_e = np.random.normal(0, 0.05, ca.shape[0])
ca_n = ca + ca_e
cb_e = np.random.normal(0, 0.05, cb.shape[0])
cb_n = cb + cb_e
cc_e = np.random.normal(0, 0.05, cc.shape[0])
cc_n = cc + cc_e
plt.scatter( t_i, ca_n, label = "$C_a$ with error")
plt.scatter( t_i, cb_n, label = "$C_b$ with error")
plt.scatter( t_i, cc_n, label = "$C_c$ with error")
plt.legend()
```

![Concentration error](images/conc_err.png)

Great! now we have some data to play with. For simplicity, we will focus on concentration A as we build our neural network, but extending the model is as simple as adding more outputs for our training and validation data. Focusing on $C_a$, we will split the data 50/50 for training and validation: 

```python3
# randomly select values using a shuffled boolean array
mask = np.array([True]*50 + [False]*50)
np.random.shuffle(mask)

ti_train = t_i[mask]
ca_train = ca_n[mask]
cb_train = cb_n[mask]
cc_train = cc_n[mask]

ti_val = t_i[~mask]
ca_val = ca_n[~mask]
cb_val = cb_n[~mask]
cc_val = cc_n[~mask]

plt.scatter( ti_train, ca_train, label = "$C_a$ train")
plt.scatter( ti_val, ca_val, label = "$C_a$ validation")
plt.legend()
```
![Concentration error](images/ca_tv_split.png)

We have a final step for inputting these data to a neural network. We need to transform them into tensors. We can use the built-in `Dataset` and `Dataloader` objects in torch (adapted from [^fn4]):

```python3
import torch
import numpy as np
from torch.utils.data import Dataset, DataLoader
import numpy as np
# select parts of data for training

X_train = ti_train
Y_train = ca_train

X_test = ti_val
Y_test = ca_val

# Convert data to torch tensors
class Data(Dataset):
    def __init__(self, X, Y):
        self.X = torch.from_numpy(X.astype(np.float32)).view(len(X),1)
        self.Y = torch.from_numpy(Y.astype(np.float32)).view(len(X),1)
        self.len = self.X.shape[0]
       
    def __getitem__(self, index):
        return self.X[index], self.Y[index]
   
    def __len__(self):
        return self.len
   
batch_size = 64

# Instantiate training and test data
train_data = Data(X_train, Y_train)
train_dataloader = DataLoader(dataset=train_data, batch_size=batch_size, shuffle=True)

test_data = Data(X_test, Y_test)
test_dataloader = DataLoader(dataset=test_data, batch_size=batch_size, shuffle=True)

# Check inputs
for batch, (X, Y) in enumerate(train_dataloader):
    print(f"Batch: {batch+1}")
    print(f"X shape: {X.shape}")
    print(f"y shape: {Y.shape}")
    break
```

output:
```bash
Batch: 1
X shape: torch.Size([50, 1])
y shape: torch.Size([50, 1])
```

Viola! Now our data is ready to be input to our neural network. 

# Non-Regularized NN
Lets start as simple as possible. We will begin with a linear model and make it more complex as we go.

```python3
from torch import nn
from torch import optim

input_dim = 1
hidden_dim = 1
output_dim = 1

model = nn.Linear(1, 1) # Short-cut way to define a 1-to-1 network without a class
criterion = nn.MSELoss()
optimizer = optim.SGD(model.parameters(), lr=0.01)

# Training Loop
for epoch in range(500):
    # Forward pass: Compute predicted y by passing x to the model
    pred_Y = model(X)
    
    # Compute loss
    loss = criterion(pred_Y, Y)
    
    # Zero gradients, perform a backward pass, and update the weights
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

# Test the model
test_input = torch.from_numpy(X_test.astype(np.float32)).view(len(X_test), 1)
predicted = model(test_input).detach().numpy()

plt.scatter(ti_val, predicted, marker="*", color="blue", label="linear nn predicted")      
plt.plot(t_i, ca, color="orange", label="Actual Chemistry")   
plt.legend()
```
![Linear Model](images/linear_nn.png)

A more complex model requires adding more layers. 

```python3
# add ReLU activation layer
model = nn.Sequential(
    nn.Linear(1, 100),
    nn.ReLU(),
    nn.Linear(100, 1)
)
# Define Loss Function and Optimizer
criterion = nn.MSELoss()  # Mean Squared Error loss
optimizer = optim.SGD(model.parameters(), lr=0.01)  # Stochastic Gradient Descent

# Training Loop
for epoch in range(500):
    # Forward pass: Compute predicted y by passing x to the model
    pred_Y = model(X)
    
    # Compute loss
    loss = criterion(pred_Y, Y)
    
    # Zero gradients, perform a backward pass, and update the weights
    optimizer.zero_grad()
    loss.backward()
    optimizer.step()

# We've made a really crappy nn for predicting concentration!
test_input = torch.from_numpy(X_test.astype(np.float32)).view(len(X_test), 1)
predicted = model(test_input).detach().numpy()

plt.scatter(ti_val, predicted, marker="*", color="blue", label="crappy nn predicted")      
plt.plot(t_i, ca, color="orange", label="Actual Chemistry")   
plt.legend()
fig_file = r"C:\Users\chris\Documents\04_writing_blog\math_mirror\content\images\crappy_nn.png"
plt.savefig(fig_file)
```
![Simple NN](images/crappy_nn.png)

This model is fine within the range of the training data, but it fails spectacularly for points that occurr. We can make our model more sane using regularization. 

# Note on data regularization
Physics informed neural networks are a special case of regularization, so we will start by learning that concept. At it's most basic, regularization prevents overfitting. A model with many parameters can fit almost any dataset near perfectly, but this sort of naive approach can lead to a model that is unphysical. To paraphrase Enrico Fermi, with enough arbitrary parameters, you can fit an elephant, and with one more you can make it wiggle it's trunk[^fn1][^fn2]. We would like to avoid overfitting, thus we regularize. 




## References
[^fn0]: H. Chen, G. E. C. Flores, and C. Li, “Physics-informed neural networks with hard linear equality constraints,” Computers & Chemical Engineering, vol. 189, p. 108764, Oct. 2024, doi: 10.1016/j.compchemeng.2024.108764.
[^fn1]: http://neuralnetworksanddeeplearning.com/ (book where I heard fermi quote)
[^fn2]: https://www.nature.com/articles/427297a (fermi quote about four parameter elephant)
[^fn3]: https://maziarraissi.github.io/PINNs/
[^fn4]: https://medium.com/@theo.wolf/physics-informed-neural-networks-a-simple-tutorial-with-pytorch-f28a890b874a
