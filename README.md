<p align="center">
  <img src="logo.png" alt="T3Net Logo" width="200"/>
</p>

# t3_BSM_Comparison

This repository implements **T3Net**, a compact neural-network framework for non-singlet PDF inference and minimal BSM sensitivity studies. It builds on:
- The NNPDF approach to unbiased PDF determination via neural networks and Monte Carlo replicas [@nnpdf-code], [@Ball_2022]
- The SimuNET methodology for embedding theory parameters into PDF fits [@simunet]
- Bayesian Gaussian-process priors for PDFs [@bayesian]

**T3Net** replaces the GP prior with a small feed-forward network and focuses on the non-singlet combination  
_T3(x) = u⁺(x) – d⁺(x)_,  
probed by the proton–deuteron structure-function difference. We generate pseudo-data with realistic correlations, perform closure tests, and study the impact of adding a single BSM distortion parameter.

---

## Abstract

Reliable collider predictions require both unbiased parton distribution functions (PDFs) and a clear separation between proton structure and potential effects from new physics. The NNPDF framework removes functional-form bias by fitting neural networks to Monte Carlo replicas of diverse data sets [@nnpdf-code; @Ball_2022], and SimuNET embeds additional theory parameters directly into the fit to prevent genuine beyond-Standard-Model (BSM) signals from being absorbed into PDFs [@simunet]. Candido et al. introduced a complementary Bayesian approach, using Gaussian processes as flexible priors and performing full inference over both PDF parameters and hyperparameters [@bayesian]. Their benchmarks on deep-inelastic scattering demonstrated rigorous uncertainty quantification and posterior validation. Inspired by that methodology, **T3Net** replaces the Gaussian process prior with a compact neural network and again focuses on the non-singlet combination _T₃(x) = u⁺ – d⁺_, probed by the proton–deuteron structure-function difference. Pseudo-data are generated from this difference with realistic experimental correlations as an input for the model. Closure tests on the fits confirm that fitting only standard QCD inputs recovers a reference distribution within its uncertainty band. Introducing a single extra theory parameter to capture generic BSM distortions uncovers a bias-variance trade-off. PDF uncertainties contract, coverage degrades, and the extra parameter is systematically underestimated. These artifacts trace back to uniform regularisation across all parameters and overly rigid constraint enforcement. By isolating these effects in a minimal setting, **T3Net** investigates the possible pitfalls in this approach and suggests avenues for future research.

---

## Requirements

- **Conda** (>= 25) or **Python 3.9+** with `venv`  
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

Create `environment.yml`:

    name: environment_nnpdf_full
    channels:
      - conda-forge
    dependencies:
      - python=3.11
      - nnpdf=4.0.10
      - pip
      - pip:
        - -r requirements.txt

Then run:

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
- Create `model_states/` and `results/`
- Fetch & preprocess BCDMS data via `validphys`
- Build FK tables & covariance matrices
- Train neural nets (standard & BSM)
- Save results to `training_results.pkl`
- Generate plots in `images/`

---

## 4. Key files

- `T3_beta.py` – Main workflow  
- `environment.yml` & `requirements.txt` – Env spec  
- `training_results.pkl` – Pickled fit results  
- `images/` – Generated figures  

---

## Downloading resources (theoryID 208, PDFs, fits)

By default, **validphys** auto-downloads resources (PDF sets, fits, theories, outputs) when needed, checking your local cache (`nnprofile`) first and fetching from the remote server if missing. Example runcard:

    pdf:   NNPDF40_nnlo_as_01180
    fit:   NNPDF40_nlo_as_01180
    theoryid: 208
    use_cuts: "fromfit"
    dataset_input:
      dataset: ATLAS_DY_7TEV_36PB_ETA
      cfac:   [EWK]
    actions_:
      - plot_fancy
      - plot_chi2dist

To disable auto-download:

    vp-setupfit --no-net ...

To force network use:

    vp-setupfit --net ...

### Manual download with `vp-get`

    vp-get --list    # list resource types
    vp-get fit NNPDF31_nlo_as_0118_1000

### Programmatic via Loader

    from validphys.loader import FallbackLoader as Loader
    l = Loader()
    l.check_theoryID(208)  # downloads if missing

---

## References

```bibtex
@article{nnpdf-code,
  author  = {Ball, Richard D. and Carrazza, Stefano and … and NNPDF Collaboration},
  title   = {An open-source machine learning framework for global analyses of parton distributions},
  journal = {Eur. Phys. J. C}, volume = {81}, number = {10}, pages = {958}, year = {2021},
  doi     = {10.1140/epjc/s10052-021-09747-9}
}
@article{Ball_2022,
  author  = {Ball, Richard D. and Carrazza, Stefano and … and Wilson, Michael},
  title   = {The path to proton structure at 1% accuracy: NNPDF Collaboration},
  journal = {Eur. Phys. J. C}, volume = {82}, number = {5}, year = {2022},
  doi     = {10.1140/epjc/s10052-022-10328-7}
}
@article{simunet,
  author  = {Iranipour, Shayan and Ubiali, Maria},
  title   = {A new generation of simultaneous fits to LHC data using deep learning},
  journal = {JHEP}, number = {05}, pages = {032}, year = {2022},
  doi     = {10.1007/JHEP05(2022)032}
}
@article{bayesian,
  author  = {Candido, Alessandro and Del Debbio, Luigi and Giani, Tommaso and Petrillo, Giacomo},
  title   = {Bayesian inference with Gaussian processes for the determination of parton distribution functions},
  journal = {Eur. Phys. J. C}, volume = {84}, number = {7}, pages = {716}, year = {2024},
  doi     = {10.1140/epjc/s10052-024-13100-1}
}
