<p align="center">
  <img src="logo.png" alt="T3Net Logo" width="200"/>
</p>

# t3_BSM_Comparison

This repository contains code for performing a closure test and BSM Wilson‐coefficient reconstruction in the non‐singlet PDF channel t3(x). The main entry point is the T3_beta.py script, which:

1. Loads and preprocesses BCDMS F2^p and F2^d data via the validphys API  
2. Builds FK tables and covariance matrices  
3. Constructs pseudo‐data for closure tests  
4. Defines and trains neural‐network models (with and without a single BSM parameter C)  
5. Produces plots comparing data vs. theory, uncertainty bands, sensitivity scans, and fitted Wilson‐coefficient distributions  

---

## Requirements

- Conda (>= 25) or Python 3.9+ with venv  
- At least 4 GB free disk space to download data/theory ingredients  
- (Optional) A GPU and CUDA drivers for faster PyTorch training  

---

## 1. Clone & initialize

git clone https://github.com/yourusername/yourrepo.git  
cd yourrepo  

---

## 2a. Install with Conda

(Optional) Freeze your current setup:  
pip freeze > requirements.txt  

Create environment.yml alongside requirements.txt:

name: environment_nnpdf_full  
channels:  
  - conda-forge  
dependencies:  
  - python=3.11  
  - nnpdf=4.0.10  
  - pip  
  - pip:  
    - -r requirements.txt  

Create & activate the environment:  
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
# Then manually install non-Python deps (LHAPDF, pandoc) as needed  

---

## 3. Running the script

Once your environment is ready:

python T3_beta.py

This will:  
- Create model_states/ and results/ directories  
- Fetch & preprocess BCDMS data  
- Build FK tables and covariance matrices  
- Train neural nets for each replica and configuration  
- Save training results to training_results.pkl  
- Generate plots in images/  

---

## 4. Key files

- T3_beta.py – The main workflow script  
- environment.yml & requirements.txt – Reproducible environment specification  
- training_results.pkl – Pickled DataFrame of fit results  
- images/ – All generated plots  

---

## Downloading resources (theoryID 208 and others)

validphys can automatically download required resources — PDF sets, completed fits, theory definitions, and past validphys outputs — when you run code that needs them. By default it checks your local cache (configured in nnprofile) and if missing, fetches from the remote server.

Example validphys runcard snippet:

    pdf: NNPDF40_nnlo_as_01180  
    fit: NNPDF40_nlo_as_01180  
    theoryid: 208  
    use_cuts: "fromfit"  
    dataset_input:  
      dataset: ATLAS_DY_7TEV_36PB_ETA  
      cfac: [EWK]  
    actions_:  
      - plot_fancy  
      - plot_chi2dist  

When you execute validphys (or vp-setupfit), it will ensure the PDF set NNPDF40_nnlo_as_01180, the fit NNPDF40_nlo_as_01180, and the theory with ID 208 are present. If not found locally, they are downloaded and installed automatically. You rarely need manual intervention.

To disable auto-download, pass `--no-net` to validphys tools. To force-enable, use `--net`.

### What can be downloaded

- **Fits** (via `vp-get fit <name>`)  
- **PDF sets** (from NNPDF or LHAPDF); fits imply their PDF sets  
- **Theories** (by theoryID, e.g. 208)  
- **validphys output files** stored in the server cache  

### The `vp-get` tool

Use `vp-get` to fetch resources manually:

    vp-get --list
    # shows available resource types: fit, pdf, theoryID, vp_output_file

    vp-get fit NNPDF31_nlo_as_0118_1000

If already installed, it reports the local path.

### Programmatic downloads via Loader

In Python, use the FallbackLoader to auto-download:

    from validphys.loader import FallbackLoader as Loader
    l = Loader()
    # downloads theory 208 if missing
    l.check_theoryID(208)

The standard Loader only searches locally and will error if the resource isn’t present.

---

## NNPDF-specific information

Installing NNPDF requires:

- A recent Python (3.9+), Linux or macOS  
- ≥ 4 GB storage  

Conda install (includes LHAPDF & pandoc):

    conda create -n environment_nnpdf nnpdf -c conda-forge
    conda activate environment_nnpdf

Pip install (manual LHAPDF & pandoc):

    python -m venv environment_nnpdf
    source environment_nnpdf/bin/activate
    python -m pip install git+https://github.com/NNPDF/nnpdf.git@4.0.10

Shared data resides under `${CONDA_PREFIX}/share/NNPDF` by default; configure via `nnprofile`. For development installation and contribution guidelines, see the official NNPDF documentation.  
