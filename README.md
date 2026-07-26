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

These are just editable variables at the top of `IPR.ipynb` / `VLP.ipynb`, not hardcoded everywhere, so you can swap in your own well data.

## Why Poettmann-Carpenter and not something like Hagedorn-Brown

Mainly because it was the correlation covered in my course and it's simpler to implement — it lumps friction and slippage into a single empirical factor (Ef) instead of tracking flow regimes separately, so there's no flow-pattern map to worry about. It's less accurate for high water-cut or high GLR wells, but fine for the kind of oil-dominated case I was testing.

## Notes on the code itself

- The pressure traverse in `VLP.ipynb` isn't vectorized, it's a plain while loop stepping in 5 psi increments until near total depth, then switching to 1 psi steps for the last 50 ft — copied that logic from how it's usually done by hand/in a spreadsheet, just automated it.
- The spline (`s=9`) has a smoothing factor, not an exact interpolation — needed that because Vogel's curve near Pb has a kink where the saturated/undersaturated parts join, and an exact spline was giving fsolve trouble converging.
- `optimum_tubing()` in `diff_API_tubing.ipynb` walks through a hardcoded list of standard tubing OD/ID sizes and stops once the rate improvement between two consecutive sizes drops below 100 STB/day — that threshold is arbitrary, just something reasonable I picked, not from any textbook.

## What I'd still like to add

- Hagedorn-Brown or Duns-Gros as a second VLP method for comparison
- gas lift analysis (this is currently only for a naturally flowing well)
- clean up the repeated Rs/Bo/Z calculation blocks in `VLP.ipynb`, they're copy-pasted between the two branches of the while loop and should really just be a function

## Disclaimer

Built this with help from Claude for debugging and structuring the code, but I made sure I understand every calculation in here (units, correlations, why the loop is structured the way it is) — this was mainly a learning project to get comfortable applying nodal analysis concepts from class in actual code.
