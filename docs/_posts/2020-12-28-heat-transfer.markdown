---
layout: post
title: Python and MATLAB - Heat Transfer
date: 2020-12-28 11:53:32 -0500
---

This class required me to use my Python and MATLAB to find solutions to real world heat transfer scenarios, including 1D and 2D conduction and heat exchangers, using analytical methods. 

The following is Python code I wrote to analyze the 2D conduction behavior in a slab, as well as a sample output:

**Sample Output**: 
![](/assets/img/blog/heattransferpcb.jpg)

**Python Code:** 

```python
# Jonah Offman
# Heat Transfer Computing Project #2
# Due: 4/3/2020

import matplotlib.pyplot as plt
import numpy as np

# Geometry
x = 35e-3  # mm
y = 15e-3  # mm
dx = 0.5e-3  # mm
t = 3e-3   # mm
nx = int(x/dx)  # nodes in x direction
ny = int(y/dx)  # nodes in y direction
n_tot = (nx * ny)  # total amount of nodes

# Properties
t_inf = 23  # Kelvin (23 Celsius)
q_gen = 200e3  # W/m^3
k = 0.5  # W/mK
h_top = 35  # W/m^2K
h_bot = 1.5*h_top  # W/m^2K

# Simplifying Constants
c_corn_top = - (((2 * h_top * dx) / k) + 2)
c_corn_bot = - (((2 * h_bot * dx) / k) + 2)
c_conv_top = - (((2 * h_top * dx) / k) + 4)
c_conv_bot = - (((2 * h_bot * dx) / k) + 4)
# Try without 2
c1_top = - ((2 * h_top * dx * t_inf) / k)
c1_bot = - ((2 * h_bot * dx * t_inf) / k)
c4 = - ((q_gen * (dx ** 2)) / k)

# Initializations Empty Temperature & Constants Matrix Using Numpy
A = np.zeros((n_tot, n_tot))
C = np.zeros((n_tot, 1))

# Iterate Through All Nodes (Note that if statements correspond to boundary conditions)
for i in range(n_tot):
    # Bottom Left Corner
    if i == 0:
        A[i, i] = c_corn_bot
        A[i, i + 1] = 1
        A[i, i + nx] = 1
        C[i] = c1_bot
    # Bottom Right Corner
    elif i == (nx - 1):
        A[i, i] = c_corn_bot
        A[i, i - 1] = 1
        A[i, (i + nx)] = 1
        C[i] = c1_bot
    # Top Left Corner
    elif i == (n_tot - nx):
        A[i, i] = c_corn_top
        A[i, i + 1] = 1
        A[i, i - nx] = 1
        C[i] = c1_top
    # Top Right Corner
    elif i == (n_tot - 1):
        A[i, i] = c_corn_top
        A[i, i - 1] = 1
        A[i, i - nx] = 1
        C[i] = c1_top
    # Bottom Surface
    elif i in range(1, nx):
        A[i, i] = c_conv_bot
        A[i, i - 1] = 1
        A[i, i + 1] = 1
        A[i, i + nx] = 2
        C[i] = c1_bot
    # Top Surface
    elif i in range(n_tot - nx, n_tot):
        A[i, i] = c_conv_top
        A[i, i - 1] = 1
        A[i, i + 1] = 1
        A[i, i - nx] = 2
        C[i] = c1_top
    # Left Surface
    elif (i % nx) == 0:
        A[i, i] = -4
        A[i, i + 1] = 2
        A[i, i + nx] = 1
        A[i, i - nx] = 1
        C[i] = 0
    # Right Surface
    elif ((i + 1) / nx) == 0:
        A[i, i] = -4
        A[i, i + 1] = 2
        A[i, i + nx] = 1
        A[i, i - nx] = 1
        C[i] = 0
    # Interior Nodes
    else:
        A[i, i] = -4
        A[i, i + 1] = 1
        A[i, i - 1] = 1
        A[i, i - nx] = 1
        A[i, i + nx] = 1
        C[i] = c4


# Solve Temperature Matrix
B = np.linalg.inv(A)
T = np.matmul(B, C)


# Turn T into Corresponding Nodes
Temp = T.reshape(ny, nx)


def surface_plot(matrix, **kwargs):
    # Define a function to create a 3D Surface Plot
    # Acquire the cartesian coordinate matrices from the matrix
    # x is cols, y is rows
    (x, y) = np.meshgrid(np.arange(matrix.shape[1]), np.arange(matrix.shape[0]))
    fig = plt.figure()
    ax = fig.add_subplot(111, projection='3d')
    surf = ax.plot_surface(1e3 * (x * dx), 1e3 * (y * dx), matrix, **kwargs)
    return fig, ax, surf


# Plot 3D Surface and Label Axes
(figure, axes, surface) = surface_plot(Temp, cmap=plt.cm.coolwarm)
axes.set_title('Temperature Distribution Across PCB Board')
axes.set_xlabel('X Pos (mm)')
axes.set_ylabel('Y Pos (mm)')
axes.set_zlabel('Temperature (Celsius)')

plt.show()
```

The following is MATLAB code written with my group members in order to analyze a counter flow heat exchanger for a project that was ultimately cut short due to COVID-19. The code allows for calculation of heat transfer over a variable number of heat exchanger passes, allowing for the group to maximum heat transfer:

**Sample Output:**
![](/assets/img/blog/heattransferexchanger.jpg)

```
%% Jonah Offman, Willa Grinsfelder, Owen Friesen, Mohid Khan and Solomon Polansky- 02/24/20
% This code determines relevant values of a heat exchanger. 
clear all 
close all
clc

%% Define Parameters
massHE = 0.025;             %Initalized mass of heat exchanger, kg

%Fluid Properties
thi = 45.0+273.15;      %Kelvin
tci = 20.0+273.15;
vdot = 3.3333E-5;       %m^3/s
ro=997;                 %kg/m^3
cp = 4178.0;            %J/K
mu = 855E-06;           %Ns/m^2
Pr = 5.83;              %Kinematic viscosity/thermal diffusivity
kwater = 0.613;         %W/mK conduction coefficient for water

%Copper Tubing Properties
massperl= 0.36;         %kg/m
d= 0.0127;              %m
kcopper= 401;           %W/mk conduction coefficient for copper
t=0.049/39.3701 ;       %Thickness, m

%Styrofoam Properties
L = (4.75*2.56)/(4*100);           %m
h = (2.00*2.56)/(100);             %m
Per = 2*(h+L);                     %m
A_sq = h*L;                        %m^2
V_box = 0.1524*0.2286*0.2159;      %m^3
%Length of tubing in heat exchanger
L = 0.127;  %L in m 
n=[2 4 6 8];

%% Calculations
mdot=vdot*ro;       %kg/s
A = (1/4)*pi.*d.^2;       %cross-sectional area of tubing, m^2

Re = []; 
%Reynolds Number (circular tube)
Reh = (4.*mdot)./(pi.*d.*mu);
%Reynolds Number (square tube)
dhydraulic = 4*A_sq/Per;
Rec = (4.*mdot)./(pi.*dhydraulic.*mu) ;
%Nusselt (churchill Bernstein)
Nuc = 0.3 + (0.62.*(Rec.^(1/2)).*(Pr.^(1/3))./((1+((0.4./Pr).^(2/3))).^(1/4))).*((1+(Rec/282000).^(5/8)).^(4/5));
%Nusselt, laminar
Nuh = 0.023*(Reh^(0.8))*(Pr^(0.4)); % could be turbulent, might need to implement if statement
%convection coefficient
hh = Nuh*(kwater/d);
hc = Nuc*(kwater/d); 

%Overall Heat Transfer Coefficient, assuming negligent conduction through
%the pipe
U = 1/((1/hh)+(1/hc)); 

%Heat Capacity Rate
C = mdot*cp;

%NTUS
A_Surface=pi.*d.*n.*L
NTU = (U.*A_Surface)./C; % should be surface area 

%Efficiency
cr = 1; % assume that cmin and cmax are equal
eff1 = (1+cr+sqrt(1+cr^2));
eff2 = (1+exp((-NTU).*sqrt(1+cr^2)));
eff3 = (1-exp((-NTU).*sqrt(1+cr^2))); 
eff = 2.*((eff1).*(eff2./eff3)).^(-1); 


%Overall Heat Transfer
q = eff.*C.*(thi - tci);

%Expected output cold/hot water
tco = q./C + tci; 
tho = thi - q./C;

%Log mean temperature --> special operating condition where delT1=delT2
tlm =thi-tco;

% Find F value
P = (tho-thi)/(tci - thi);
R = (tci - tco)/(tho - thi);
F =  1;

%Mass of the heat exchanger and figure of merit
mass = massHE + massperl.*L.*n; 
merit1 = q./mass; 
merit2 = q./V_box; 

L_elbow = 0.3048; 
L_tot = n.*L + (n/2).*L_elbow ; 
lambda = Reh./64;
v = mdot./(ro.*A); 
dP = (lambda.*L_tot.*ro.*(v^2))./(2.*d) ;
merit3 = q./dP ;

%% Generate scatter plot 
figure
scatter(n,merit1,50,'filled');
xlabel('Number of Passes');
ylabel('Figure of Merit (W/kg)'); 

figure
scatter(n,q,50,'filled');
xlabel('Number of Passes');
ylabel('Max Heat Transfer (W)');

figure
scatter(n,merit2,50,'filled');
xlabel('Number of Passes');
ylabel('Figure of Merit (W/m^3)');
 
figure
scatter(n,merit3,50,'filled');
xlabel('Number of Passes');
ylabel('Figure of Merit (W/Pa)');

```


[Back to Projects](/#projects)






