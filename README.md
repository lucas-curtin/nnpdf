<p align="center">
  <img src="logo.png" alt="T3Net Logo" width="200"/>
</p>

# t3_BSM_Comparison

This repository implements **T3Net**, a compact neural-network framework for non-singlet PDF inference and minimal BSM sensitivity studies. It builds on:
- The NNPDF approach to unbiased PDF determination via neural networks and Monte Carlo replicas [1][2]
- The SimuNET methodology for embedding theory parameters into PDF fits [3]
- Bayesian Gaussian-process priors for PDFs [4]. This is the core inspiration for this analysis, and should be consulted in detail to understand the origin of the methodology.

**T3Net** replaces the GP prior from [4] with a small feed-forward network and focuses on the non-singlet combination  
T3(x) = u⁺(x) − d⁺(x),  
probed by the proton–deuteron structure-function difference. We generate pseudo-data with realistic correlations, perform closure tests, and study the impact of adding a single BSM distortion parameter.

---

## Abstract

Reliable collider predictions require both unbiased parton distribution functions (PDFs) and a clear separation between proton structure and potential effects from new physics. The NNPDF framework removes functional-form bias by fitting neural networks to Monte Carlo replicas of diverse data sets [1][2], and SimuNET embeds additional theory parameters directly into the fit to prevent genuine beyond-Standard-Model (BSM) signals from being absorbed into PDFs [3]. Candido et al. introduced a complementary Bayesian approach, using Gaussian processes as flexible priors and performing full inference over both PDF parameters and hyperparameters [4]. Their benchmarks on deep-inelastic scattering demonstrated rigorous uncertainty quantification and posterior validation. Inspired by that methodology, **T3Net** replaces the Gaussian process prior with a compact neural network and again focuses on the non-singlet combination T3(x) = u⁺ − d⁺, probed by the proton–deuteron structure-function difference. Pseudo-data are generated from this difference with realistic experimental correlations as an input for the model. Closure tests on the fits confirm that fitting only standard QCD inputs recovers a reference distribution within its uncertainty band. Introducing a single extra theory parameter to capture generic BSM distortions uncovers a bias–variance trade-off. PDF uncertainties contract, coverage degrades, and the extra parameter is systematically underestimated. These artifacts trace back to uniform regularisation across all parameters and overly rigid constraint enforcement. By isolating these effects in a minimal setting, **T3Net** investigates pitfalls in this approach and suggests avenues for future research.


---
## Code Structure and Approach

NNPDF and SimuNET are built for large-scale, global PDF fits—frameworks with many moving parts, extensive configuration files, and multi-step workflows. For smaller, focused studies like T3Net, this complexity can obscure the core logic and make rapid prototyping or targeted analyses cumbersome.

In T3_beta.py, we deliberately condense the entire workflow into a single, self-contained script. This design serves three main goals:

1. Clarity of data flow
   All data-loading, preprocessing, model setup, training loops, and plotting routines live in one file. You can trace every variable from its origin (e.g., validphys.API calls) through to its final use (loss computation, output figures).

2. Template for custom analysis
   By stripping away global-fit infrastructure, the script becomes a clear template:
   - Fetch exactly the inputs you need (BCDMS proton/deuteron tables, FK tables, covariance matrices).
   - Build your own simple network (T3Net or T3NetWithC) and training loop.
   - Generate outputs in model_states/, results/, and images/ with no extra scaffolding.

3. Encouraging hands-on understanding
   When everything lives in one script, it’s easy to experiment:
   - Change the definition of the loss function or regularization terms.
   - Swap in different ansatz functions (K1, K2, etc.).
   - Try alternative data splits or a different BSM parameterization.
   No need to rebuild or re-install a large package—just modify the code, rerun, and inspect the results.


---

## Requirements

- Conda (>= 25) or Python 3.9+ with venv  
- ≥ 4 GB free disk space for data/theory ingredients  
- (Optional) GPU + CUDA for faster PyTorch training  

---

## 1. Clone & initialize

git clone https://github.com/yourusername/yourrepo.git  
cd yourrepo  

---

## 2a. Install with Conda

(Optional) freeze your current environment:  
pip freeze > requirements.txt  

Create environment.yml:

name: environment_nnpdf_full  
channels:  
  - conda-forge  
dependencies:  
  - python=3.11  
  - nnpdf=4.0.10  
  - pip  
  - pip:  
    - -r requirements.txt  

Then:

conda env create -f environment.yml  
conda activate environment_nnpdf_full  

---

## 2b. Install with pip + venv

python3 -m venv .env_nnpdf  
source .env_nnpdf/bin/activate  
pip install --upgrade pip  
pip install \  
  git+https://github.com/NNPDF/nnpdf.git@4.0.10 \  
  -r requirements.txt  
# install LHAPDF and pandoc manually if needed  

---

## 3. Running the script

python T3_beta.py  

This will:  
- Create model_states/ and results/  
- Fetch & preprocess BCDMS data via validphys  
- Build FK tables & covariance matrices  
- Train neural nets (standard & BSM)  
- Save results to training_results.pkl  
- Generate plots in images/  

---

## 4. Key files

- T3_beta.py – Main workflow  
- environment.yml & requirements.txt – Env spec  
- training_results.pkl – Pickled fit results  
- images/ – Generated figures  

---

## Downloading resources (theoryID 208, PDFs, fits)

Before running `T3_beta.py`, you should explicitly download the theory and PDF sets your analysis depends on. From your shell:

```bash
# 1) Download the theory definition with ID 208
vp-get theoryID 208

# 2) Download the corresponding PDF set
vp-get pdf NNPDF40_nnlo_as_01180

# 3) (If you want to include the fit itself)
vp-get fit NNPDF40_nlo_as_01180
```
---

## References

1. R. D. Ball et al., “An open-source machine learning framework for global analyses of parton distributions,” Eur. Phys. J. C 81 (2021) 958, doi:10.1140/epjc/s10052-021-09747-9  
2. R. D. Ball et al., “The path to proton structure at 1 % accuracy: NNPDF Collaboration,” Eur. Phys. J. C 82 (2022) 10328, doi:10.1140/epjc/s10052-022-10328-7  
3. S. Iranipour and M. Ubiali, “A new generation of simultaneous fits to LHC data using deep learning,” JHEP 05 (2022) 032, doi:10.1007/JHEP05(2022)032  
4. A. Candido et al., “Bayesian inference with Gaussian processes for the determination of parton distribution functions,” Eur. Phys. J. C 84 (2024) 716, doi:10.1140/epjc/s10052-024-13100-1  
