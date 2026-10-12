This directory contains three groups of experiments related to FGCR and ROS: conditional probabilistic inference, structural and runtime comparisons on standard/random Bayesian networks, and Target-Order experiments on CHILD/ALARM. Each subdirectory contains the corresponding Notebooks, input data, and saved results. Please launch Jupyter from the corresponding experiment directory to ensure that relative paths are resolved correctly.

#Directory Structure#

* `Probabilistic Inference/`: known-parameter inference, finite-sample inference, and interventional inference on the ALARM network.
* `runtime/`: structural-consistency/runtime experiments on six standard networks, together with runtime experiments on randomly generated DAGs.
* `Target-Order/`: numerical experiments, tests, plotting, and saved-result verification on the CHILD and ALARM networks.
* The `data/`, `result/`, `results_*/`, and `figures/` directories within each experiment folder store input data and parameters, result tables/logs, and figures, respectively. Workbooks and repetition-level CSV files typically also contain summary and audit information.


#Environment Setup#

Python 3.11 or later and Jupyter are recommended. Dependencies vary across experiment directories and mainly include NumPy, Pandas, OpenPyXL, Matplotlib, and NetworkX. The standard-network loading code contains a fallback parser for BIF structures, while pgmpy may be used for BIF loading. The core network and inference code of the Target-Order experiments uses NumPy. Existing runtime environment records can be found in `Target-Order实验/results/environment.json` and in the environment sheet of the conditional-probability result workbooks. The recorded environment is Python 3.11.15, NumPy 2.4.6, and Windows 10.

To install the basic dependencies in a new environment and launch Jupyter:

```bash
python -m pip install jupyter numpy pandas openpyxl matplotlib networkx
jupyter notebook
```

If the required BIF reader is unavailable in the runtime environment, the fallback parsing implementation in the standard-network Notebook can be used to load the network structure. The PNG reconstruction workflow of the Target-Order experiments also invokes the system program `pdftoppm`; make sure Poppler is installed and that this command can be executed from the terminal. Execution order:


#1. Probabilistic Inference Experiments#

In `Probabilistic Inference实验/`, first run `00_FGCR_Common_Core.ipynb`, and then run:

1. `01_Known_Parameter_Inference.ipynb`: known-parameter inference.
2. `02_Finite_Sample_Inference.ipynb`: finite-sample estimation, Bias/RMSE, and repeated experiments.
3. `03_Do_Intervention_Inference.ipynb`: conditional inference after intervention.

The formal experiment scale is controlled by the Notebook and `data/alarm_frozen_protocol.json`; before execution, confirm that `FGCR_RUN_MODE=formal`. The finite-sample Notebook supports resuming from checkpoints through `FGCR_RESUME=1`. The parameter bank is located at `data/alarm_parameter_sets.xlsx`, and its content hash should match the value recorded in the frozen JSON protocol. Formal execution writes results to `result/`, while the finite-sample experiment also uses `checkpoints/`; if the original packaged results need to be preserved, copy these directories before running.

The current machine-readable protocol and the frozen settings in the Notebooks are the basis for reproducing the actual results. This archive does not include a separate legacy protocol-description document.


#2. runtime Experiments#

In `计时实验/`, run:

* `FGCR_ROS_standard_networks.ipynb`: six standard networks—Asia, Insurance, Alarm, Win95pts, Andes, and Munin. The structural-consistency comparison uses 30 three-node conditioning sets and 4 fixed topological orders for each network, giving 720 instances in total; the runtime experiment uses one fixed order for each of the 30 conditioning sets per network, giving 180 paired instances in total. Each instance uses 3 warm-up runs and 10 formal repetitions.
* `FGCR_ROS_random_networks.ipynb`: the main experiment uses 50, 100, 200, and 500 nodes with target average in-degrees of 2, 4, and 8; the stress test separately uses 1,000 nodes with average in-degrees of 2 and 4. Conditioning-set sizes are generated from the fixed size 3 and proportions 0.05, 0.10, 0.20, and 0.30, with duplicate sizes removed. Each instance uses 4 fixed topological orders, 3 warm-up runs, and 10 formal repetitions.

The `STANDARD_DATA_DIR` in the standard-network Notebook is currently an absolute Windows path from the author's machine. Before running on another machine, change this variable to the actual local path of `计时实验/data`. Results are saved by default in `results_standard_networks/` and `results_random_networks/`.

The timing interval contains only the algorithmic core specified in the Notebook. FGCR/ROS output-consistency checks, input generation, result aggregation, and result saving are excluded from the algorithm runtime. Timing results are affected by hardware, operating system, dependency versions, and system load. Cross-machine reproduction should therefore focus on verifying the configuration, timing boundaries, and output consistency rather than requiring identical absolute runtimes.


#3. Target-Order Experiments#

In `Target-Order实验/`, launch Jupyter from this directory and run the following in order:

1. `00_bn_core.ipynb`: shared discrete Bayesian-network, inference, and sampling implementations.
2. `01_test_core.ipynb`: correctness checks.
3. `02_run_experiments.ipynb`: run the CHILD and ALARM numerical experiments.
4. `03_make_outputs_png.ipynb`: generate figures from the saved results.
5. `04_verify_saved.ipynb`: replay random samples and verify the saved records.

`config.json` fixes the common seed at 20261007, 100 topological orders, 20 repetitions, a pilot sample size of 500, a fitting pool of 5,000, a validation sample size of 1,000, fitting sample sizes of 100/1,000/5,000, BDeu ISS=10, and `delta=0.05`. BIF networks, parameters, topological orders, and conditional samples are stored in `data/`; repetition-level data and audit materials are stored in `results/`; PNG files are stored in `figures/`.

The packaged `results/tests.json` and `results/replay_verification.json` record successful test and replay verification results. The complete numerical experiments may take considerable time; before running them, back up `data/`, `results/`, and `figures/`, because the corresponding Notebooks may generate or update these files.


#Results and Reproducibility Notes#

* With fixed random seeds, input files, frozen protocols, and experiment configurations, structural and non-timing numerical results should be reproducible within the floating-point tolerances specified by the code; bitwise-identical floating-point results are not guaranteed across different platforms.
* In the Target-Order experiments, different sample sizes use nested prefixes of the same fitting-sample pool; therefore, rows in the experimental details are not necessarily mutually independent. Seeds for the randomized Kahn algorithm allow the corresponding topological orders to be replayed, but this does not imply uniform sampling from the set of all topological orders.
* The Probabilistic Inference and Target-Order experiments use finite networks, parameterizations, and query settings; the scope of the conclusions is determined by the corresponding configurations.
* Random-network seeds, graph inputs, conditioning sets, and topological orders are stored in the result workbooks. When comparing reproduced results, first verify these inputs, configurations, algorithm-output consistency, and result hashes.
