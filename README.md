# The Enhanced Modified Kalman Filter (eMKF) tool for small domain estimation

**General disclaimer** This repository was created for use by CDC programs to collaborate on public health related projects in support of the [CDC mission](https://www.cdc.gov/about/organization/mission.htm).  GitHub is not hosted by the CDC, but is a third party website used by CDC and its partners to share information and collaborate on software. CDC use of GitHub does not imply an endorsement of any one particular service, product, or enterprise. 

## Overview

This project contains the SAS code to implement the **enhanced Modified Kalman Filter (eMKF)**. The enhanced MKF procedure enables production of model-based estimates for small populations where direct estimates may lack precision, improving the availability of data for assessing and monitoring health disparities. The enhanced MKF procedure and macro build on the earlier Modified Kalman Filter procedure (Setodji, Lockwood, McCaffrey, Elliott & Adams, 2011) to accommodate nonlinear time trends, irregularly spaced time points, and random sampling variances for the underlying population subgroup means, rates, or proportions. Bayesian estimation in the eMKF macro is implemented adaptably and transparently using PROC MCMC and related SAS 9.4 procedures. Model averaging in the eMKF macro uses a Bayesian mixture prior approach, and renders predictions more robust to polynomial trend misspecification. Various other features in the eMKF macro also improve its functionality, flexibility, and usability relative to the earlier macro; an outline of these improvements is included below. 

This project contains the SAS macro to implement the eMKF, emkf_macro.sas, along with several examples of SAS code to implement the eMKF macro [Testing-and-implementation] with some sample data sets [Sample-data-files]. All data included here are public-use and/or simulated data. 

### Requires

* SAS 9.4

### Running the eMKF macro

[include description here of how to run the macro]

## Related documents

* [Link to Series 2 report]
* [Link to Series 2 report]
* [] (Default eMKF macro parameter settings)

### Outline of methodological differences between eMKF and the original MKF (implemented as of 11-October-2023)

* Time points

     1.   eMKF allows for time points to be unequally spaced.
     2.   In the MLE-based setting, eMKF updates the recursion formulas used to determine the MSE-optimal estimators (BLUPs) of the true states to allow for an arbitrary lag between successive time points instead of lag=1 (see Lockwood et al 2011; DOI: 10.1002/sim.3897).

* Polynomial trends

     3.   eMKF allows for quadratic and cubic time trends to be fitted in both the Bayesian and MLE-based estimation settings.
     4.   By default, eMKF does not allow fitting a degree k polynomial trend (k=0,1,2,3) unless there are k+4 available time points.
     5.   eMKF returns an error if there is only one available time point per group.
     6.   eMKF pre-transforms trend coefficients using an orthogonal polynomial design matrix for comparability of coefficients in linear, quad, and cubic trend models. The coefficients are reverse-transformed before the program exits so the user only sees the "raw" coefficients.

* Sampling variances

     7.   eMKF allows for random sampling variances in the Bayesian setting (see Polettini 2017; DOI 10.1214/16-BA1019, for an overview).
     8.   eMKF uses (effective) sample size (neff) as degrees of freedom in chi-squared distribution of group- and time-specific sampling variance, therefore neff must be supplied. 

* Disparities calculations

     9.   In the Bayesian setting, eMKF estimates all pairwise differences and ratios between groups at the latest time point.
     10. Additional disparities measures (highest and lowest rates, maximal rate difference and ratio, summary rate difference and ratio) are also calculated.
     11. The user can further calculate any other measures he/she desires from the posterior draws, which the user can request to be saved to the workspace. 
      These features differ from MKF where only pairwise differences were calculated and the full MCMC samples from the joint posterior distribution were not available.

* Bayesian estimation setting

     12. eMKF implements "independent" and "common" trend options, which were left out of the RAND version of the macro, in addition to the "full" hierarchical Bayesian model (see RAND User's Guide "TR997_compiled").
     13. eMKF implements Bayesian model averaging using a mixture prior approach, up to linear (3 possible models), quadratic (5 possible models), or cubic (7 possible models).
     14. eMKF replaces the call to the external .exe file (which consisted of pre-compiled C code) with a call to PROC MCMC.
     15. eMKF implements Gibbs sampling in PROC MCMC by calling user defined samplers (UDSs) that are custom-built and precompiled using PROC FCMP.
     16. eMKF replaces z-score-based convergence diagnostic used in MKF with robust version of the Gelman-Rubin diagnostic (see Vehtari et al 2021; DOI 10.1214/20-BA1221).
     17. eMKF applies the Gelman-Rubin diagnostic to all model parameters, not just for the true state predictions (etas) as in the original MKF.
     18. eMKF defaults to more stringent threshold of 1.01 instead of 1.10 for the Gelman-Rubin diagnostic, as per recommendation in Vehtari et al (2021). 
     19. eMKF defaults to 4 chains instead of 3, as per recommendation in Vehtari et al (2021). Each chain is further split in 2 to compute the Gelman-Rubin diagnostic.
     20. eMKF uses closed-form expressions for the determinant, inverse, and Cholesky decomposition of the AR variance-covariance matrix whenever possible to speed up calculations.
     21. eMKF allows the user to select the built-in slice sampler in PROC MCMC to use instead of the traditional random walk MH sampler for sampling AR parameters + SD hyperparameters.

* MLE-based estimation setting

     22. In eMKF, any subset of the seven allowable models (indep_cubic, _quad, _linear; common_cubic, _quad, _linear; and dropped) can be averaged. 
     23. However, the code checks the specified models and adds a common "descendent" if is not already included, so as to have a reference model for Bayes factors.
     24. eMKF increases the maxiter option for PROC NLMIXED to 400 instead of 200 (default) when dealing with two outcomes to improve convergence when k = 2,3.
     25. eMKF initializes parameters to pass to PROC NLMIXED using the appropriate 'by' group stratum/replication (PROC REG). This differs from MKF where only the first stratum/replication was used to initialize the regression coefficients across strata/replications.
     26. eMKF initializes parameters to pass to PROC NLMIXED using the appropriate degree k polynomial regression (PROC REG), including for k=0. This differs from MKF where for k=0 (dropped), the intercept values were initialized at those from the linear regression y=a+b*t instead of y=a.
     27. For the dropped (k=0) case, eMKF only keeps the column vector of 1s in the X matrix (and subsequent matrix calculations for the MSE). This differs from MKF where both the 1s and ts column were kept in the X matrix.

* Macro usability

     28. eMKF includes extensive comments and streamlines the code for readability.
     29. eMKF allows the user additional flexibility in customizing model output and diagnostics, and streamlines the SAS workspace.
     30. eMKF checks for errors in macro parameter specification, including length of character strings for prefix of output datasets.
     31. Std. Error label in output table was replaced with RMSE in eMKF to avoid confusion.


### References

Lockwood JR, McCaffrey DF, Setodji CM, Elliott MN. Smoothing across time in repeated cross-sectional data. Stat Med 30(5):584–94. 2011.

Polettini S. A generalised semiparametric Bayesian Fay–Herriot model for small area estimation shrinking both means and variances. Bayesian Anal 12(3):729–52. 2016.
    
Rossen LM, Talih M, Patel P, Earp M, Parker JD. Evaluation of an enhanced modified Kalman filter approach for estimating health outcomes in small subpopulations. National Center for Health Statistics. Vital Health Stat 2(208). 2024. 

Setodji CM, Lockwood JR, McCaffrey DF, Elliott MN, Adams JL. The Modified Kalman Filter macro: User’s guide. RAND Technical Report No. TR-997-DHHS. 2011. Available from: https://www.rand.org/pubs/technical_reports/TR997.html.

Talih M, Rossen LM, Patel P, Earp M, Parker JD. Technical guidance for using the modified Kalman filter in small-domain estimation at the National Center for Health Statistics. National Center for Health Statistics. Vital Health Stat 2(209). 2024. 

Vehtari A, Gelman A, Simpson D, Carpenter B, Bürkner PC. Rank-normalization, folding, and localization: An improved Ȓ for assessing convergence of MCMC (with discussion). Bayesian Anal 16(2):667–718. 2021.

  
## Public Domain Standard Notice
This repository constitutes a work of the United States Government and is not
subject to domestic copyright protection under 17 USC § 105. This repository is in
the public domain within the United States, and copyright and related rights in
the work worldwide are waived through the [CC0 1.0 Universal public domain dedication](https://creativecommons.org/publicdomain/zero/1.0/).
All contributions to this repository will be released under the CC0 dedication. By
submitting a pull request you are agreeing to comply with this waiver of
copyright interest.

## License Standard Notice
The repository utilizes code licensed under the terms of the Apache Software
License and therefore is licensed under ASL v2 or later.

This source code in this repository is free: you can redistribute it and/or modify it under
the terms of the Apache Software License version 2, or (at your option) any
later version.

This source code in this repository is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the Apache Software License for more details.

You should have received a copy of the Apache Software License along with this
program. If not, see http://www.apache.org/licenses/LICENSE-2.0.html

The source code forked from other open source projects will inherit its license.

## Privacy Standard Notice
This repository contains only non-sensitive, publicly available data and
information. All material and community participation is covered by the
[Disclaimer](DISCLAIMER.md)
and [Code of Conduct](code-of-conduct.md).
For more information about CDC's privacy policy, please visit [http://www.cdc.gov/other/privacy.html](https://www.cdc.gov/other/privacy.html).

## Contributing Standard Notice
Anyone is encouraged to contribute to the repository by [forking](https://help.github.com/articles/fork-a-repo)
and submitting a pull request. (If you are new to GitHub, you might start with a
[basic tutorial](https://help.github.com/articles/set-up-git).) By contributing
to this project, you grant a world-wide, royalty-free, perpetual, irrevocable,
non-exclusive, transferable license to all users under the terms of the
[Apache Software License v2](http://www.apache.org/licenses/LICENSE-2.0.html) or
later.

All comments, messages, pull requests, and other submissions received through
CDC including this GitHub page may be subject to applicable federal law, including but not limited to the Federal Records Act, and may be archived. Learn more at [http://www.cdc.gov/other/privacy.html](http://www.cdc.gov/other/privacy.html).

## Records Management Standard Notice
This repository is not a source of government records, but is a copy to increase
collaboration and collaborative potential. All government records will be
published through the [CDC web site](http://www.cdc.gov).

## Additional Standard Notices
Please refer to [CDC's Template Repository](https://github.com/CDCgov/template) for more information about [contributing to this repository](https://github.com/CDCgov/template/blob/main/CONTRIBUTING.md), [public domain notices and disclaimers](https://github.com/CDCgov/template/blob/main/DISCLAIMER.md), and [code of conduct](https://github.com/CDCgov/template/blob/main/code-of-conduct.md).
