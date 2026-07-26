# Nodal Analysis Simulator (Oil Wells)

This is a small project I built to understand nodal analysis for a flowing oil well — basically finding the point where the reservoir's inflow (IPR) and the wellbore/tubing's outflow (VLP) meet, and then playing around with that point by changing skin and tubing size.

I'm a 2nd year Petroleum Engineering student, still fairly new to Python, so the code is not the cleanest and there's probably a more efficient way to do half of this. But everything here is something I actually worked through and can explain — Vogel's equation, Poettmann-Carpenter, spline fitting to get a smooth intersection, the works.

## What it actually does

1. Builds the **Inflow Performance Relationship (IPR)** curve using Vogel's correlation (handles both saturated and undersaturated cases depending on where Pb sits relative to Pr).
2. Builds the **Vertical Lift Performance (VLP)** curve using the Poettmann-Carpenter multiphase flow correlation, marching pressure down the tubing from wellhead to bottomhole in small steps.
3. Fits both curves with cubic splines (`scipy.interpolate.splrep/splev`) so they're smooth enough to actually find where they cross.
4. Uses `fsolve` to solve for the intersection — that intersection point is the well's actual operating point (Qo, Pwf).
5. On top of that base case, I added a couple of sensitivity studies:
   - what happens to the operating point if skin is reduced (stimulation effect)
   - what happens if you change the tubing ID
   - a small loop that sweeps through a list of standard tubing sizes and picks the "optimum" one (the biggest tubing size before the rate gain flattens out below a threshold)

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

## Inputs I used (base case)

- Pwh = 100 psi, T = 150°F, tubing ID = 2.441 in
- API = 35°, GLR = 273 scf/STB
- Yg = 0.75, Yo = 0.55, Yw = 1.05
- Depth = 5000 ft
- Pr = 3000 psi, Pb = 2130 psi, test point: Qo = 250 STB/d @ Pwf = 2500 psi
- Skin pressure drop from DST = 200 psi
.
