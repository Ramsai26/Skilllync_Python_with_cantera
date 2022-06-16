# Question

Use the source code as shown in the video, but alter it to satisfy the following conditions

Write a code that takes all the reactions from the GRI mechanism and calculates 10 most sensitive reactions
The sensitivity parameter should be with respect to the Temperature (It should find the sensitivity with respect to the temperature not 'OH' as shown in the video) 
Consider the initial conditions as shown in the video
Your program should show the reaction strings in the order of greatest sensitivity to the least sensitivity
 

Your program should be parametric, It is should be intuitive and easy to obtain 'n' number of sensitive reactions with respect to the asked parameter, by simply changing a number.

# Answer

# Aim: Find the 10 most sensitive reactions out of the first 100 reactions 

As per GRI mechanism there are 325 total reactions present in that were finding 10 most sensitive reactions for first 100 reactions

So here we need to use some new class to acquire the first 100 reactions from the gri

# Context:

In reaction mechanism there are multiple elementary reaction that takes place which results in creation of Intermediate species until it reaches the chemical equilibrum. So depending upon the elementary reaction operating conditions we can know how fastly combustion takes place can be known.

So simply altering the parameters of the operating conditions changes the outcome of the  reactions. So here by doing sensitive analysis we can find what are the sensitive reaction which ate easily manipulated by temperature,pressure or mass fractions etc

```python
import sys
import numpy as np #numpy is used to calculate matlab like functions
import matplotlib.pyplot as plt #Used to plot 

import cantera as ct #used to do reactor network problem


gas=ct.Solution('gri30.xml') #reading the reactions
temp=1500 #defining temperature and pressure
pres=ct.one_atm
#Defining n to find first 10 reactions 
n=10

#Giving variable properties 
gas.TPX=temp,pres,{'CH4':1,'O2':2,'N2':7.52}
#choosing governing equations
r=ct.IdealGasConstPressureReactor(gas)
sim=ct.ReactorNet([r])
R_eq=gas.reaction_equations()
print(R_eq)
#total number of reactions
n_react=325
S_max=[0]*n_react

#adding reactions to sensitivity class to calculate sensititvity 
for i in range(n_react):
    r.add_sensitivity_reaction(i)

sim.rtol = 1e-06 #defining relative tolerance
sim.atol = 1e-15 #defining absolute tolerace

sim.rtol_sensitivity = 1e-06 #defining relative tolerance
sim.atol_sensitivity = 1e-06 #defining absolute tolerance

# creating an states array to store all te gas state properties
states= ct.SolutionArray(gas, extra=['t','s2','s3'])
#Defining total number of reactionss
reac_num=np.arange(0,n_react)

#by using advance reation we are integrating time function
for t in np.arange(0,2e-3,5e-6):
    sim.advance(t)
    
    # Storing the affect of sensitivity with respect to temperature
    for j in range(n_react):
        S=sim.sensitivity('temperature',j)
        if S>S_max[j]: 
            S_max[j]=S
        

#assagning S_max absolute values to
S_abs=np.abs(S_max)
R_eq=[x for y,x in sorted(zip(S_abs,R_eq),reverse=True)]
reac_num=[x for y,x in sorted(zip(S_abs,reac_num),reverse=True)]
S_max=[x for y,x in sorted(zip(S_abs,S_max),reverse=True)]


max_R=R_eq[0:n]
max_s=S_max[0:n]
min_R=R_eq[315:325]
min_s=S_max[315:325]


#plotting Bar graph reactions sensitivity wrt temperature
plt.figure(1)
plt.barh(max_R,max_s)
plt.gca().invert_yaxis()
plt.xlabel('Sensitivity')
plt.ylabel('Reactions')
plt.title('Most sensitive reactions')
plt.tight_layout()
plt.show()
```
