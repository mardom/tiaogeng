# tiaogeng: recovering organized randoms for galaxy clustering measurements with SOM+HC

## Introduction

This is a python package to calculate the ''organized random'' for galaxy clustering measurement based on the idea from [Johnston et al., 2021](https://arxiv.org/abs/2012.08467) and [Yan et al., 2024](https://arxiv.org/abs/2410.23141).


To measure the two-point function of galaxy distribution, one needs to calculate so-called random-random and data-random pair counts (see https://ui.adsabs.harvard.edu/abs/1993ApJ...412...64L/abstract). However, if the galaxy sample has variable depth due to anisotropic selection effects by various systematics (Galactic extinction, seeing, etc), a uniform random will result in biased estimation of the 2PCF. Therefore, one needs to recover the ''organized random (OR)' to eliminate the selection effect of a synthesis of systematics. In practice, selection functions of systematics could be complicated thus cannot be fit with simple formula. In addition, different systematics might be correlated, which makes the problem more complicated.


The method applies a combination of self-organizing map (SOM) and hierarchical clustering (HC) to identify the clustering of galaxies on high-dimensional systematics space, and then resample them back to the survey footprint to recover the organized random. This method has been tested on mock galaxy catalogs and the KiDS-1000 bright sample and the KiDS-Legacy sample.

## Basic idea

The whole idea can be summarized as follows:

1. Train a SOM with systematics vectors of the galaxy sample;
2. Group the SOM weight with HC. Each group represents a subsample of galaxies that shares similar systematis and therefore selections;
3. Map galaxies from each hierarchical cluster back to the sky. Galaxies from one cluster occuples disjoint sky regions that shares approximately the same selection function;
4. Resample galaxies randomly in each disjoint region according the galaxy number of corresponding hierarchical cluster;
5. Combine the resampled "galaxies" from all the clusters to get the OR catalog.

In practice, the organized randoms are reconstructed as a pixelized weight map that quantifies the likelihood that a galaxie will be kept due to systematics selection in each part of the sky. In this package, we use Healpix scheme to pixelize the organised randoms. We also pixelize the galaxy catalog to speed up 2PCF measurement. For a more detailed technical description of the SOM+HC method, check Section 4 of [Yan et al., 2024](https://arxiv.org/abs/2410.23141).

For galaxy 2PCF, the OR is used as the ``random catalog'' for a Landey-Salay estimator. For galaxy angular power spectra pseudo-$$C_{\ell}$$, the OR weight map is used to generate the galaxy overdensity map and calculate shot noise, as well as serves as the weight map to calculate mode-coupling matrix (see Appendix B of [Yan et al., 2024](https://arxiv.org/abs/2410.23141)).

## Structure of this package

All the source codes are stored in `./code/src`, including:

- OR_weights.py: the file that contains the class to recover organized random weight;
- treecorr_utils.py: contains a function to call `treecorr` to calculate 2PCF;
- plot_som.py: contains a function to make fancy SOM plots;
- glass_mock.py: generates mock catalogs with GLASS;
- generate_mocksys.py: assigns simple mock systematics and depletion functions to a galaxy sample.

In these files, only `OR_weights.py` is essential, and users are welcome to use their own codes to generate mock sample, calculate 2PCF and visualize them.

An example notebook is given in `./code/notebooks` which reads a mock catalog generated by the [GLASS](https://glass.readthedocs.io/en/stable/) package. Then it adds three types of systematics with simple depletion functions and train a SOM+HC to recover the organized random weight. $w(\theta)$'s' are calculated from un-depleted sample + uniform random (unbiased), depleted sample + uniform random (biased), depleted sample + depleted random (true OR), and depleted sample + reconstructed OR (recovered OR). If the idea works well, then $w(\theta)$ with the true OR and recovered OR should agree with each other.

## Required packages

Essential: `numpy`, `scipy`, `matplotlib`, `healpy`, [`somoclu`](https://somoclu.readthedocs.io/en/stable/) (to train a SOM), `sklearn` (to do hierarchical clustering)

Optional (for the example notebook): [`glass`](https://glass.readthedocs.io/en/stable/) (note that we use the version "2023.06" of glass-dev/glass here. other versions might not work well), [`treecorr`](https://rmjarvis.github.io/TreeCorr/_build/html/index.html), [`pyccl`](https://ccl.readthedocs.io/en/latest/index.html), `tqdm` (for showing progress bar)

## What is a "tiaogeng🥄"?


Tiaogeng (调羹) is the Chinese word for “spoon”, more commonly used in southern China. It contains two characters: “tiao(调)”meaning to reconcile and “geng (羹)” referring to a Chinesestyle thick soup. The code reconciles the unevenly observed sky, just as a tiaogeng stirs soup to make it taste more balanced and delicious.

