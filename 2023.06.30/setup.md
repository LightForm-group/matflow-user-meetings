## Setting up on the CSF

* Usually we would use the MatFlow install script to download and install the latest version of MatFlow. However, due to the ongoing cyber incident at UoM, the proxy server is not accessible on the CSF. This means we cannot connect to the internet on the CSF, and so cannot use the normal installation script.
* So instead of this, we have installed a version of MatFlow in a shared space, and to follow along with the session, you will need to add this shared space to your PATH environment variable like this:

  ```bash
  cp ~/.bashrc ~/.bashrc_backup
  echo "export PATH=\"\$PATH:/mnt/eps01-rds/jf01-home01/shared/software/matflow_exes\"" >>~/.bashrc
  source ~/.bashrc
  ```

* Next, we will to tell MatFlow about the environment definitions in the file `envs.yaml`, with this command:

  ```bash
  matflow-dev config append environment_sources /mnt/eps01-rds/jf01-home01/shared/software/matflow_envs/envs.yaml
  ```

## Setting up a local Python environment

* Create a local conda environment on your local machine, so we can inspect the results of the demo workflows:

  ```bash
  conda create -n matflow_new_env python=3.10 jupyter
  pip install matflow-new==0.3.0a28 matplotlib
  ```
