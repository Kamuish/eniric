# ENIRIC - Extended Near InfraRed Information Content
Analysis of near infrared spectra information content.

[![Codacy Badge](https://api.codacy.com/project/badge/Grade/24d3d525a79d4ae493de8c527540edef)](https://www.codacy.com/app/jason-neal/eniric?utm_source=github.com&utm_medium=referral&utm_content=jason-neal/eniric&utm_campaign=badger)
[![Build Status](https://travis-ci.org/jason-neal/eniric.svg?branch=master)](https://travis-ci.org/jason-neal/eniric)[![Coverage Status](https://coveralls.io/repos/github/jason-neal/eniric/badge.svg?branch=master)](https://coveralls.io/github/jason-neal/eniric?branch=master)[![Code Climate](https://codeclimate.com/github/jason-neal/eniric/badges/gpa.svg)](https://codeclimate.com/github/jason-neal/eniric)[![Issue Count](https://codeclimate.com/github/jason-neal/eniric/badges/issue_count.svg)](https://codeclimate.com/github/jason-neal/eniric)[![Test Coverage](https://codeclimate.com/github/jason-neal/eniric/badges/coverage.svg)](https://codeclimate.com/github/jason-neal/eniric/coverage)


## Purpose
To analysis which spectroscopic bands contain the most information for radial velocity measurements.
Model spectra are used to analyze the information content of different bands.
They undergo two convolutions, one for rotational broadening and one for instrumental broadening.
The slope of the spectra are used as a proxy for the radial velocity precision attainable.

The purpose of this work is to:
- Extend the previous analysis to a range of different metallicity values.


## Background
The origin of this code was used in [this paper](https://arxiv.org/abs/1511.07468).

    P. Figueira, V. Zh. Adibekyan, M. Oshagh, J. J. Neal, B. Rojas-Ayala, C. Lovis, C. Melo, F. Pepe, N. C. Santos, M. Tsantaki, 2016,
    Radial velocity information content of M dwarf spectra in the near-infrared,
    Astronomy and Astrophysics, 586, A101

It has a number of efficiency problems which need to be improved upon before the new analysis is performed.

1) Use numpy mapping slicing instead of comprehension lists.  (~>250 times faster)
2) Use Joblib to parallelize the convolutions.


## Outline

The code works in two main stages, "spectral preparation" and "precision calculation".

#### Spectrum preparation

`eniric/nIRanalysis.py`

This stage takes in the raw PHOENIX-ACES spectral models and transforms them, saving the results of this computation as .txt files.

It includes:
- Conversion from flux to photon counts.
- Resolution convolution
- Re-sampling

Some scripts are given in `bin` to run this preparation over all desired parameters automatically. You will have to modify the paths to things.


#### Precision Calculations

`python bin/nIR_precision.py`

This takes in the processed spectra and performs the precision calculations for all 3 conditions outlined in the original paper.
- Cond1. Total information
- Cond2. +/-30km/s telluric line > 2% masking
- Cond3. Perfect telluric correction with variance correction

It also *scales the flux level* to a desired SNR level in a desired band, see below, as this affects the RV precision calculated. By default this is a SNR of 100 in the J band.

## Run-time results:
Comparing the same calculation performed between the old and new code after a series of changes.
On laptop after replacing the comprehension lists:

    new convolution = 27 seconds
    old convolution = 1hr 22 minutes
Ridiculous!


## Bugs:
A number of bugs were found when improving this code. Mainly affecting the condition involving telluric line masking. This alters the RV in this column, sometimes significantly. This, however, does **NOT** alter the conclusions in the published paper.


## Band SNR Scaling.
By default, in accordance with the initial paper, each spectra band is normalized to 100 SNR in the center of the J band.

This now does this automatically by measuring the SNR in 1 pixel resolution (3 points) in the center of the band. And scales accordingly. This adds a spectral model dependent factor on the RV precision.
To get around you can manually specify the SNR level to normalize to and which specific band to normalize to. (it can be itself for instance).

You can do this by ....

The code for the automatic snr detection and application is in `eniric/snr_normalization.py`
