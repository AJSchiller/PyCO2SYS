# Should you trust PyCO2SYS?

There are no "certified" results of marine carbonate system calculations against which software like PyCO2SYS can be unambiguously validated.  But we can evaluate its performance by testing its internal consistency and by comparing its outputs with those from other programs.

!!! info "PyCO2SYS v1.3.0"
    All the comparisons, numbers and figures shown on this page were made using PyCO2SYS v1.3.0.

## Internal consistency

### Round robin

PyCO2SYS can solve the marine carbonate system from any valid pair of total alkalinity, dissolved inorganic carbon, pH, partial pressure or fugacity or molinity of aqueous CO<sub>2</sub>, bicarbonate ion molinity, and carbonate ion molinity.  In a "round robin" test, we first determine all of these core variables from one given input pair, and then solve the system again from the results using every possible combination of pairs as the input.  We expect to find exactly the same results from every input pair combination.

!!! success "Internal consistency of PyCO2SYS"
    * The differences arising from using different input pair combinations for the same composition are negligible.
    * Any differences are at least 10<sup>10</sup> times smaller than the best measurement accuracy for any of the variables.

We can conveniently run a round-robin test with PyCO2SYS for any given set of input conditions using `PyCO2SYS.test.roundrobin` ([script available here](https://github.com/mvdh7/PyCO2SYS/blob/master/validate/round_robin.py)):

    :::python
    # Import PyCO2SYS
    import PyCO2SYS as pyco2

    # Define test conditions
    par1 = 2300  # parameter 1, here total alkalinity in μmol/kg-sw
    par2 = 8.1  # parameter 2, here pH on the Total scale
    par1type = 1  # "parameter 1 is total alkalinity"
    par2type = 3  # "parameter 2 is pH"
    sal = 33  # practical salinity
    temp = 22  # temperature in °C
    pres = 1000  # pressure in dbar
    si = 10  # total silicate in μmol/kg-sw
    phos = 1  # total phosphate in μmol/kg-sw
    nh3 = 2  # total ammonia in μmol/kg-sw
    h2s = 3  # total sulfide in μmol/kg-sw
    pHscale = 1  # "input pH is on the Total scale"
    k1k2c = 10  # "use LDK00 constants for carbonic acid dissociation"
    kso4c = 3  # "use D90a for bisulfate dissociation & LKB10 for borate:sal"

    # Run the test
    res, diff = pyco2.test.roundrobin(par1, par2, par1type, par2type,
        sal, temp, pres, si, phos, pHscale, k1k2c, kso4c, NH3=nh3, H2S=h2s,
        buffers_mode="none")

Running the round-robin test with the inputs in the example code above gave the following maximum absolute differences across all input pair combinations:

<div style="text-align:center">
<!-- HTML for table generated with examples/round-robin.py -->
<table><tr>
<th style="text-align:right">Carbonate system parameter</th>
<th style="text-align:center">Mean result</th>
<th style="text-align:center">Max. abs. diff.</th></tr>
</tr><tr>
<td style="text-align:right">Total alkalinity / μmol/kg-sw</td>
<td style="text-align:center">2300.0</td>
<td style="text-align:center">9.09·10<sup>−13</sup></td>
</tr><tr>
<td style="text-align:right">Dissolved inorganic carbon / μmol/kg-sw</td>
<td style="text-align:center">1982.2</td>
<td style="text-align:center">4.55·10<sup>−13</sup></td>
</tr><tr>
<td style="text-align:right">pH (Total scale)</td>
<td style="text-align:center">8.1</td>
<td style="text-align:center">1.78·10<sup>−15</sup></td>
</tr><tr>
<td style="text-align:right"><i>p</i>CO<sub>2</sub> / μatm</td>
<td style="text-align:center">312.0</td>
<td style="text-align:center">1.53·10<sup>−12</sup></td>
</tr><tr>
<td style="text-align:right"><i>f</i>CO<sub>2</sub> / μatm</td>
<td style="text-align:center">311.0</td>
<td style="text-align:center">1.48·10<sup>−12</sup></td>
</tr><tr>
<td style="text-align:right">Carbonate ion / μmol/kg-sw</td>
<td style="text-align:center">218.1</td>
<td style="text-align:center">8.53·10<sup>−14</sup></td>
</tr><tr>
<td style="text-align:right">Bicarbonate ion / μmol/kg-sw</td>
<td style="text-align:center">1754.5</td>
<td style="text-align:center">6.82·10<sup>−13</sup></td>
</tr><tr>
<td style="text-align:right">Aqueous CO<sub>2</sub> / μmol/kg-sw</td>
<td style="text-align:center">9.6</td>
<td style="text-align:center">4.80·10<sup>−14</sup></td>
</tr></table>
</div>

The maximum absolute differences across all the different input pair combinations are negligible in this example, all at least ten orders of magnitude smaller than the accuracy with which any of these variables can be measured.  The differences are not exactly zero because the iterative pH solvers stop once a certain tolerance threshold is reached.  By default, this threshold is set[^1] at 10<sup>−8</sup> (in pH units) in PyCO2SYS.

### Buffer factors

PyCO2SYS offers two independent ways to evaluate the various buffer factors of the marine carbonate system: with explicit equations and by automatic differentation.

!!! success "Independent approaches give the same results"
    Differences between buffer factors calculated with explicit equations and by automatic differentiation are negligible.

The "explicit" approach is taken by functions in `PyCO2SYS.buffers.explicit`.  These use the equations for each buffer factor that have been reported in the literature, corrected for typographical errors where known.  These functions typically only include the effects of carbonate, borate and water alkalinity on each buffer, because the other components of alkalinity complicate the equations but usually only make a small difference to the result.

The "automatic" approach is PyCO2SYS's default behaviour and is taken by the functions in `PyCO2SYS.buffers`.  These functions calculate each buffer factor based on its definition as a derivative in the literature.  The relevant derivative of the appropriate function from `PyCO2SYS.solve.get` is evaluated automatically by [Autograd](https://github.com/HIPS/autograd) ([M16](../refs/#m)).  This has several advantages:

  * If the function in `PyCO2SYS.solve.get` is correct, then its derivatives are also accurate.
  * All equilibrating solutes accounted for in the main alkalinity equation are automatically included in all derivatives too (not just carbonate, borate and water).

Comparing the results of these two independent methods with each other therefore supports that both are being executed correctly.  [For example](https://github.com/mvdh7/PyCO2SYS/blob/master/validate/buffers_auto_explicit.py), under typical open ocean conditions[^2] and with zero nutrients:

<!-- HTML for table generated with examples/buffers-auto-explicit.py -->
<table><tr><th></th>
<th style="text-align:center">Revelle factor</th>
<th style="text-align:center"><i>ψ</i></th>
<th style="text-align:center"><i>γ</i><sub>DIC</sub></th>
<th style="text-align:center"><i>γ</i><sub>Alk</sub></th>
<th style="text-align:center"><i>β</i><sub>DIC</sub></th>
<th style="text-align:center"><i>β</i><sub>Alk</sub></th>
<th style="text-align:center"><i>ω</i><sub>DIC</sub></th>
<th style="text-align:center"><i>ω</i><sub>Alk</sub></th>
<th style="text-align:center"><i>Q</i></th>
</tr><tr>
<th style="text-align:center">Explicit</th>
<td style="text-align:center">13.3336</td>
<td style="text-align:center">0.7714</td>
<td style="text-align:center">0.1612</td>
<td style="text-align:center">−0.1821</td>
<td style="text-align:center">0.1821</td>
<td style="text-align:center">−0.1901</td>
<td style="text-align:center">−0.2090</td>
<td style="text-align:center">0.1990</td>
<td style="text-align:center">1.1291</td>
</tr><tr>
<th style="text-align:center">Automatic</th>
<td style="text-align:center">13.3336</td>
<td style="text-align:center">0.7714</td>
<td style="text-align:center">0.1612</td>
<td style="text-align:center">−0.1821</td>
<td style="text-align:center">0.1821</td>
<td style="text-align:center">−0.1901</td>
<td style="text-align:center">−0.2090</td>
<td style="text-align:center">0.1990</td>
<td style="text-align:center">1.1291</td>
</tr><tr>
<th style="text-align:center">Difference</th>
<td style="text-align:center">1.56·10<sup>−8</sup></td>
<td style="text-align:center">−2.83·10<sup>−7</sup></td>
<td style="text-align:center">3.18·10<sup>−7</sup></td>
<td style="text-align:center">−3.88·10<sup>−7</sup></td>
<td style="text-align:center">3.88·10<sup>−7</sup></td>
<td style="text-align:center">−4.05·10<sup>−7</sup></td>
<td style="text-align:center">−4.89·10<sup>−7</sup></td>
<td style="text-align:center">4.24·10<sup>−7</sup></td>
<td style="text-align:center">1.80·10<sup>−7</sup></td>
</tr></table>

We consider these differences all small enough to be negligible.

Although explicit check values are not available, we can attempt to recreate figures from the literature to check the consistency of PyCO2SYS's calculations.  For example, you can use [buffers_ESM10.py](https://github.com/mvdh7/PyCO2SYS/blob/master/validate/buffers_ESM10.py) to make a passable replicate of Fig. 2 of [ESM10](../refs/#e):

<p style='text-align:center'>
<img src='https://raw.githubusercontent.com/mvdh7/PyCO2SYS/master/examples/figures/buffers_ESM10.png' title="Recreation of ESM10's Fig. 2 with PyCO2SYS"/>
</p>

Switching between `buffers_mode='explicit'` and `buffers_mode='auto'` in PyCO2SYS does not alter this figure sufficiently for any differences to be visible.

## External comparisons

### CO2SYS for MATLAB

PyCO2SYS was originally based on [CO2SYS for MATLAB, version 2.0.5](https://github.com/jamesorr/CO2SYS-MATLAB/releases/tag/v2.0.5).  We should therefore expect that the results of these two programs will agree with each other perfectly, or that differences should be negligible for calculations where PyCO2SYS has since adjusted its calculation approach.

The MATLAB program has itself[^3] been rigorously compared with a suite of similar software packages that have been implemented in several different coding languages by [OEG15](../refs/#o).  Indeed, it was used as the reference against which all other packages were compared, while noting that this does not guarantee it is error-free.  Thanks to the work of [OEG15](../refs/#o), comparisons with CO2SYS for MATLAB allow us to assess the accuracy of PyCO2SYS in the context of all the software packages that they tested.

However, PyCO2SYS now calculates a wider array of properties than CO2SYS for MATLAB, and it has more inputs and options, so not everything can be tested this way.

!!! example "What can't be compared?"
    We cannot compare calculations with carbonate ion, bicarbonate ion or aqueous CO<sub>2</sub> as one of the input marine carbonate system parameters, because these options are not available in CO2SYS v2.0.5 for MATLAB.

    We also cannot test the [PF87](../refs/#p) input option for the hydrogen fluoride [dissociation constant](../co2sys/#settings).

    The PyCO2SYS outputs either not calculated or not returned by CO2SYS v2.0.5 for MATLAB are:

      * All of the [buffer factors](../co2sys/#buffer-factors) except for the Revelle factor.
      * All properties associated with [NH<sub>3</sub> and H<sub>2</sub>S](../co2sys/#alkalinity-and-its-components).
      * [Total calcium](../co2sys/#totals-estimated-from-salinity) molinity.

You can run the comparisons that the following discussion is based on yourself with [compare_MATLABv2_0_5.m](https://github.com/mvdh7/PyCO2SYS/tree/master/validate/compare_MATLABv2_0_5.m) (in MATLAB[^4]) and [compare_MATLABv2_0_5.py](https://github.com/mvdh7/PyCO2SYS/tree/master/validate/compare_MATLABv2_0_5.m) (in Python).

In these tests, the marine carbonate system is solved from every possible combination of [input parameter pair](../co2sys/#carbonate-system-parameters) and [CO2SYS settings](../co2sys/#settings), with non-zero nutrients and pressure.  The results calculated by CO2SYS are then subtracted from those of PyCO2SYS for comparison.

!!! success "PyCO2SYS vs CO2SYS-MATLAB"
    * Every variable computed by `PyCO2SYS.CO2SYS` is negligibly different from its counterpart in CO2SYS v2.0.5 for MATLAB, except for the Revelle factor ([see discussion below](#revelle-factor-discrepancy)).
    * The absolute differences in all compared variables (except the Revelle factor) under the test conditions range from 0 to a maximum on the order of 10<sup>−6</sup>%.
    * The greatest component of these differences arises from the lower tolerance threshold for the iterative pH solvers in PyCO2SYS (10<sup>−8</sup>) compared with their MATLAB counterparts (10<sup>−4</sup>).

We've also compared the [original CO2SYS clone](../co2sys/#the-original-co2sys-clone) in `PyCO2SYS.original.CO2SYS` against CO2SYS for MATLAB.  It is important to remember that this module stands in isolation.  It has its own self-contained set of functions for evaluating equilibrium constants and solving the marine carbonate system that do not interact with the rest of PyCO2SYS.  Agreement between this module and CO2SYS for MATLAB tells us nothing about the performance of `PyCO2SYS.CO2SYS` or any of its underlying functions (and the opposite).

!!! success "The original CO2SYS clone vs CO2SYS-MATLAB"
    * Every variable computed by the [original CO2SYS clone](../co2sys/#the-original-co2sys-clone) in `PyCO2SYS.original.CO2SYS` is virtually identical to its counterpart in CO2SYS v2.0.5 for MATLAB.
    * The absolute differences in all compared variables under the test conditions range from 0 to a maximum on the order of 10<sup>−10</sup>%.

#### Revelle factor discrepancy

Under the fairly typical open-ocean conditions used for these tests (for specifics, see the [*compare_MATLABv2_0_5* scripts](https://github.com/mvdh7/PyCO2SYS/tree/master/validate)), the maximum absolute differences between the Revelle factor under input and output conditions respectively were 0.09 and 0.11, which are almost 1% of the average calculated Revelle factor of ~12.  While probably negligible in most real-world applications, these are too big to be acceptable from a software perspective.

!!! failure "Inconsistency between PyCO2SYS and CO2SYS-MATLAB"
    * The Revelle factor is about 1% different in PyCO2SYS compared with CO2SYS v2.0.5 for MATLAB.
    * We understand why.
    * The PyCO2SYS calculation is more accurate.
    * The difference is probably too small to matter for most real-world applications of the Revelle factor.

There are three separate reasons for the difference, the first two of which could be considered as errors in CO2SYS for MATLAB:

  1. Incorrect reference DIC value used in the final evaluation.
  2. No "Peng correction" in calculation under output conditions.
  3. Relative inaccuracy of the finite-difference approach.

PyCO2SYS corrects errors 1 and 2 above.  By default, PyCO2SYS also evaluates the Revelle factor using automatic differentiation, which is inherently more accurate.

With `buffers_mode='explicit'`, PyCO2SYS does use the finite-difference approach but it reduces the DIC perturbation size from 1 to 0.01 μmol/kg-sw to increase its accuracy.  If we revert both this change and the corrections above, then PyCO2SYS returns Revelle factors that agree with CO2SYS for MATLAB to within 10<sup>−6</sup>%.

[^1]: The pH tolerance threshold for all iterative solvers is controlled by `PyCO2SYS.solve.get.pHTol`.

[^2]: The [default conditions](../co2sys/#using-the-pythonic-api) for `PyCO2SYS.api.CO2SYS_wrap` with total alkalinity = 2300 μmol/kg-sw and DIC = 2150 μmol/kg-sw.

[^3]: The [OEG15](../refs/#o) intercomparison used CO2SYS v1.1 of [HPR11](../refs/#HPR11), but the only difference between v2.0.5 and this in terms of solving the marine carbonate system is the addition of an extra set of carbonic acid dissociation constants.

[^4]: The output of [compare_MATLABv2_0_5.m](https://github.com/mvdh7/PyCO2SYS/blob/master/validate/compare_MATLABv2_0_5.m) is saved as [a CSV file](https://github.com/mvdh7/PyCO2SYS/blob/master/validate/results/MATLAB_CO2SYSv2_0_5.csv) that can be directly imported by [compare_MATLABv2_0_5.py](https://github.com/mvdh7/PyCO2SYS/blob/master/validate/compare_MATLABv2_0_5.py), in case you do not have access to MATLAB.