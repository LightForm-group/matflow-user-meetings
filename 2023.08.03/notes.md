# 2023.08.03 MatFlow User Meeting

## Setting up on the CSF

### If you did not attend last time

* Usually we would use the MatFlow install script to download and install the latest version of MatFlow. However, due to the ongoing cyber incident at UoM, the proxy server is not accessible on the CSF. This means we cannot connect to the internet on the CSF login nodes, and so cannot use the normal installation script.
* So instead of this, we have installed a version of MatFlow in a shared space, and to follow along with the session, you will need to add the following shared space directory to your `PATH` environment variable:

  ```bash
  /mnt/eps01-rds/jf01-home01/shared/software/matflow_exes
  ```

* If you are unsure how to modify your `PATH` variable, then you can instead copy and paste this code into your terminal:

  ```bash
  cp ~/.bashrc ~/.bashrc_backup
  echo "export PATH=\"\$PATH:/mnt/eps01-rds/jf01-home01/shared/software/matflow_exes\"" >>~/.bashrc
  source ~/.bashrc
  ```

### For everyone

* Check you have the latest version (0.3.0a39) with: `matflow-dev --version`
* Some new features (`matflow show`) require a modified configuration file, to ensure that the multiple login nodes on the CSF appear as the same "machine" to MatFlow.
* Run this command to overwrite your current config file with one we have prepared:

  ```bash
  cp /mnt/eps01-rds/jf01-home01/shared/software/matflow_configs/2023.07.20_config.yml ~/.matflow-new/config.yml
  ```
* Now check this worked by running this command:

  ```bash
  matflow-dev config get --all
  ```

* Look at the output from this command. Under the `meta-data` heading, you should see an entry called: `config_invocation_key`. This should identify as `CSF3`.

## Example workflow 1: `fit_yield_functions.yml`

* Copy the workflow from the shared location to somewhere on your scratch directory:

  ```bash
  cp /mnt/eps01-rds/jf01-home01/shared/software/matflow_workflows/fit_yield_functions.yaml ~/scratch
  ```

* Submit the workflow:

  ```bash
  matflow-dev go fit_yield_functions.yaml
  ```
