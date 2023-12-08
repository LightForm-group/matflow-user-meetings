# Set up on the CSF

## Check version

- Check the version of MatFlow with `matflow-dev --version` (should be the latest version: `v0.3.0a107`).
- If not, check you have this in your `~/.bashrc` file:

  ```
  export PATH="$PATH:/mnt/eps01-rds/jf01-home01/shared/software/matflow_exes"
  ```

## Check if you need to configure MatFlow on CSF3

- First check if already configured: run `matflow-dev config get --all` and see if "manchester-CSF3" is the loaded config. If not, continue:
  - Normally via the `matflow-configs` repo: https://github.com/hpcflow/matflow-configs
  - Since we still have no internet access on CSF3, we will specify the search directory as a shared location: `/mnt/eps01-rds/jf01-home01/shared/software/matflow_configs/2023.10.27`
  - Run `matflow-dev config init --path /mnt/eps01-rds/jf01-home01/shared/software/matflow_configs/2023.10.27 manchester`
  - Then run `matflow-dev config get --all` and the "manchester-CSF3" config should be loaded
 
## Clear known submissions

- A recent update requires us to clear our submissions history: `matflow-dev manage clear-known-subs`
