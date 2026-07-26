# IPR–VLP Nodal Analysis

This project is a Python implementation of **nodal analysis** for an oil well. The goal is to estimate the well's operating point by combining the **Inflow Performance Relationship (IPR)** and **Vertical Lift Performance (VLP)** curves.

The project was built to better understand how reservoir performance and tubing performance work together to determine the actual production rate of a well.


## Project Overview

A producing well is controlled by two systems:

* The **reservoir**, which delivers fluid into the wellbore.
* The **tubing**, which carries the produced fluid to the surface.

The reservoir can only supply a certain flow rate for a given bottomhole pressure, while the tubing requires a certain pressure to lift that flow to the surface. The point where these two conditions match is the well's operating point.

This project generates both curves, finds their intersection numerically, and studies how different operating conditions affect production.


## IPR Model

The IPR curve is generated using the **Vogel correlation**.

The implementation considers both saturated and undersaturated reservoir conditions:

* For saturated reservoirs, the Vogel equation is used directly.
* For undersaturated reservoirs, the model combines a linear Darcy-flow region above the bubble point with the Vogel equation below the bubble point.

Flow Efficiency (FE) is included to represent the effect of near-wellbore damage on productivity.


## VLP Model

The VLP curve is calculated using the **Poettmann–Carpenter correlation**.

Instead of assuming constant fluid properties throughout the well, the pressure is calculated step by step along the tubing. At each step, the model updates important fluid properties such as:

* Solution Gas–Oil Ratio (Rs)
* Oil Formation Volume Factor (Bo)
* Gas Compressibility Factor (Z)
* Gas Formation Volume Factor (Bg)
* Mixture Density

Updating these properties helps capture how the fluid changes as pressure decreases from the bottom of the well to the surface.


## Operating Point

After generating the IPR and VLP curves, the calculated points are interpolated using **SciPy's cubic spline interpolation**. The operating point is then obtained using **`scipy.optimize.fsolve`**, which finds the pressure where the two curves intersect.


## Sensitivity Studies

The notebook includes a few common production engineering cases:

* Effect of removing skin damage (well stimulation)
* Effect of tubing diameter on production
* Effect of changing wellhead pressure
* Selection of an optimum tubing size based on production improvement


## Sample Well

The example well uses the following data:

* Reservoir Pressure: 3000 psi
* Bubble Point Pressure: 2130 psi
* Well Depth: 5000 ft
* Tubing ID: 2.441 in
* API Gravity: 35°
* GLR: 273 scf/STB
* Wellhead Pressure: 100 psi

For this case, the calculated operating point is approximately:

* Production Rate: **940 STB/day**
* Bottomhole Flowing Pressure: **600 psi**


## Observation

One interesting result was observed while testing very small tubing sizes. At higher production rates, the VLP curve was no longer a simple decreasing curve. Friction losses increased rapidly, causing the curve to bend back and creating the possibility of multiple intersections with the IPR curve.

This is something that should be considered when selecting the operating point and could be improved further in future versions of the project.


## Technologies Used

* Python
* NumPy
* SciPy
* Pandas
* Matplotlib
* Jupyter Notebook

## Files

| File | What's in it |
|---|---|
| `IPR.ipynb` | Vogel IPR calculation |
| `VLP.ipynb` | Poettmann-Carpenter VLP calculation (pressure traverse down the tubing) |
| `Interpolation.ipynb` | spline fit for both curves + the intersection solver + base case plot |
| `skin_effect.ipynb` | sensitivity of operating point to skin/drawdown |
| `tubing_effect.ipynb` | sensitivity of operating point to tubing ID |
| `diff_API_tubing.ipynb` | loops over a range of tubing sizes to find the optimum tubing size |
| `all_plots.ipynb` | just runs all the notebooks together in one place |

They're chained together using `%run`, so `IPR.ipynb` feeds into `VLP.ipynb`, which feeds into `Interpolation.ipynb`, and so on. If you just want the full thing end to end, run `all_plots.ipynb` — it pulls in everything else.

(Heads up — `all_plots.ipynb` also tries to `%run Pwf_eff.ipynb` which I hadn't finished/uploaded here, so if you run it as-is that line will throw a file-not-found. Just comment it out, doesn't affect the rest.)

## Future Improvements

* Include water-cut sensitivity analysis.
* Compare results with additional multiphase flow correlations.
* Improve the tubing optimization routine by checking for stable operating points.
* Export pressure traverse calculations to CSV.


## Author

**Divyansh Guptar**
B.Tech, Petroleum Engineering
IIT (ISM) Dhanbad
