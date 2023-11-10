# Notable recent changes

- added more demonstration workflows: use `matflow-dev demo-workflow --list` to list available workflows
  - demo workflows are listed on the docs site: https://docs.matflow.io/stable/reference/workflows.html 
- added `fit_single_crystal_parameters` workflow

# Set up on the CSF

## Check version

- Check the version of MatFlow with `matflow-dev --version` (should be the latest version: `v0.3.0a90`).

## Check if you need to configure MatFlow on CSF3

- First check if already configured: run `matflow-dev config get --all` and see if "manchester-CSF3" is the loaded config. If not, continue:
  - Normally via the `matflow-configs` repo: https://github.com/hpcflow/matflow-configs
  - Since we still have no internet access on CSF3, we will specify the search directory as a shared location: `/mnt/eps01-rds/jf01-home01/shared/software/matflow_configs/2023.10.27`
  - Run `matflow-dev config init --path /mnt/eps01-rds/jf01-home01/shared/software/matflow_configs/2023.10.27 manchester`
  - Then run `matflow-dev config get --all` and the "manchester-CSF3" config should be loaded

## The `demo-data` CLI

- MatFlow can now use example data files as well as demonstration workflows
- Use this command to list available files: `matflow-dev demo-data --list`
- Demo data files are not included with the "frozen" MatFlow installation (because they might be too large), so to make all demo data files available we need to cache them. This requires internet access, so firstly hop on to a compute node: `qrsh -l short`. Then run this command to download the demo data files:
  `matflow-dev demo-data cache --all`
- This files can now be used within workflow via a reference like: `<<demo_data_file:FILE_NAME>>` (see the fitting workflow we will discuss today)
- There is also a `demo-data copy` command to copy one of the example files to a specified location.

# Demonstration of fitting single-crystal parameters

- Copy the workflow from this repository `fit_single_crystal_parameters_demo.yaml` to your CSF3 scratch space
- Submit the workflow
- We will discuss the structure of the workflow
- On completion, zip up the workflow and download to your computer: `matflow-dev zip WORKFLOW_ID` where `WORKFLOW_ID` is the `ID` provided by the `matflow-dev show` command
- Visualise the results of the workflow using the included Jupyter notebook: `notebook.ipynb`. You will need an environment with `matflow` and `formable` installed.

