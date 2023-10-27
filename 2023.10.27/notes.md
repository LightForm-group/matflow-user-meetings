# Notable recent bugs fixes

- Elements are now iterable, like:
  ```python
  for element in wk.tasks[0].elements:
      print(element.inputs.p1.value)
  ```
- Reusing a workflow template that was previously used to generate a persistent workflow now works (e.g. `AttributeError: '_DummyPersistentWorkflow' object has no attribute 'check_parameters_exist'` is fixed)
- Parameters now do not need to be predefined in the app template components
- Environment definitions are no longer stored within the workflow (just the label is stored); this allows us to move workflows from one machine to another and continue the submission with the environments definitions on the new machine.

# Plan for today

1. Demonstrate submitting a subset of tasks
2. Demonstrate continuation of a workflow on another machine
3. Demonstrate aborting a workflow run (different from cancelling the workflow)
4. Familiarise ourselves with the `demo-workflow` CLI/API

# Set up on the CSF

## Check version

- Check the version of MatFlow with `matflow-dev --version` (should be the latest version: `v0.3.0a80`).

## Demonstration of `config init` on CSF3

- First check if already configured: run `matflow-dev config get --all` and see if "manchester-CSF3" is the loaded config. If not, continue:
- Normally via the `matflow-configs` repo: https://github.com/hpcflow/matflow-configs
- Since we still have no internet access on CSF3, we will specify the search directory as a shared location: `/mnt/eps01-rds/jf01-home01/shared/software/matflow_configs/2023.10.27`
- Run `matflow-dev config init --path /mnt/eps01-rds/jf01-home01/shared/software/matflow_configs/2023.10.27 manchester`
- Then run `matflow-dev config get --all` and the "manchester-CSF3" config should be loaded

# Part 1: run a workflow on multiple machines

## 1.1 set up an MTEX environment locally

> If you don't have MATLAB/MTEX installed, you can skip to 1.3 and use the pre-zipped workflow `tension_DAMASK_texture_CTF_2023-10-27_124344.zip` instead.

- This requires that you have MATLAB and MTEX installed on your laptop.
- Update to the latest version of MatFlow (`v0.3.0a80`, you can check with `matflow --version`)
  - It is better to use the Python package for this
  - If you don't have an existing virtual environment with MatFlow installed, you can set up a new conda environment like this:

    ```bash
    conda create -n matflow_new_env python=3.10 jupyter
    pip install matflow-new
    ```
  - If you do have an existing virtual environment, you can update MatFlow with pip like this: `pip install -U matflow-new`
- Remember that all MatFlow commands you run must be run with this environment activated!
- Check to see if you already have an environment sources files: `matflow config get environment_sources`
- If you do not have an environment sources file:
  - Create a new file called `envs.yaml` in your MatFlow configuration directory: `~/.matflow-new`
  - Tell MatFlow about it with: `matflow config append environment_sources envs.yaml`
- Add the `matlab_env` environment to your environment sources file
  - If on a desktop operating system, you should be able to open this file with you default text editor like this `matflow open env-source`
    - Alternatively, you can get the path to this file (and then open it yourself) like this: `matflow open env-source --path`
  - Example Windows MATLAB env (you can use `get-command matlab` in PowerShell to find the path to MATLAB):
    ```yaml
    - name: matlab_env
      executables:
        - label: run_mtex
          instances:
            - command: |
                & 'C:\Program Files\MATLAB\R2023a\bin\matlab.exe' -batch "<<script_name_no_ext>> <<args>>"
              num_cores: 1
              parallel_mode: null
    ```
  - Example MacOS/Linux MATLAB env (untested; you can use `which matlab` to find the path to MATLAB):
    ```yaml
    - name: matlab_env
      executables:
        - label: run_mtex
          instances:
            - command: |
                /path/to/matlab -batch "<<script_name_no_ext>> <<args>>"
              num_cores: 1
              parallel_mode: null
    ```

## 1.2: submit the first task of the `tension_DAMASK_texture_CTF` example workflow

- This workflow is a simple uniaxial tension simulation with DAMASK, using orientations sampled from a CTF file. 
- We will submit locally just the first task, which runs the texture sampling script in MTEX.
- Download the files: `tension_DAMASK_texture_CTF.yaml` and `texture_Al.ctf`
- Modify the `CTF_file_path` value in the `sample_texture_from_CTF_file_mtex` task so it point to the CTF file
- Submit only the first task of this workflow: `matflow go tension_DAMASK_texture_CTF.yaml --tasks 0`

## 1.3 copy the workflow to the CSF

- First zip it up. You can use the `matflow show` command to get the ID of the workflow you just ran locally. Then do: `matflow zip ID` where `ID` is the integer in the first column of the `matflow show` output that corresponds to your workflow. This will produce a zip file.
- Copy this zip file to the CSF
- Now on the CSF, unzip the workflow with this command: `unzip /path/to/workflow.zip -d /path/to/workflow` (i.e for the `-d` argument, just remove the `.zip` extension)

## 1.4 submit the remainder of the workflow on the CSF

- Submit the rest of the workflow using the `workflow` CLI, like this: `matflow-dev workflow /path/to/workflow submit`
- Use `matflow-dev show` to check on the status of the workflow

# Part 2: aborting a running action

- In this part we will show how it's possible to abort an action that is running without cancelling the whole workflow. This is useful when running simulations; the simulation might have produced sufficient data and might not need to be run to completion.
- Not all actions are *abortable*, as there is a small computational overhead associated with allowing an action to be stopped without killing the whole workflow
  
## 2.1 demonstration of `demo-workflow` CLI/API

- The example workflows will now be bundled with MatFlow and accessible on the command-line via the `demo-workflow` command
- You can list available workflows with the `--list` option: `matflow-dev demo-workflow --list`
- Many more demo workflows will soon be added
- Execute a workflow in the current directory with the `go` sub-command: `matflow-dev demo-workflow go tension_DAMASK_Al`
- Copy a demo workflow to somewhere accessible like: `matflow-dev demo-workflow copy tension_DAMASK_Al /path/to/destination/`

## 2.2 submit the tension_DAMASK_Al workflow but with a sequence on the VE grid size

- Copy the demo workflow to somewhere on your scratch space: `matflow-dev demo-workflow copy tension_DAMASK_Al .`
- Modify the resources in the template:
  - Add a top-level resources block, so that by default, all tasks use the short queue on CSF3:
    ```yaml
    resources:
      any:
        scheduler_args:
          options: ["-l short"]
    ```
  - Add a sequence on the `VE_grid_size` input of the `generate_volume_element_from_voronoi` task. Use two values: `[8, 8, 8]` and `[32, 32, 32]`.
  - Submit the workflow
  - Use `matflow-dev show -f` to check when the simulation task's first element has completed (with the smaller grid size)
  - <strike>Once the first element has completed, use the `abort-run` command to stop the second element simulation. The post-processing should still be performed.</strike>
    - <strike>*Hint*: look at the documentation for the `matflow-dev workflow` command.</strike>
  - There is a bug that means the `abort-run` command will not work; however we can reproduce the same effect by copying the included file `abort_EARs.txt` to the submissions directory: `WORKFLOW_PATH/artifacts/submissions/0` (the `abort-run` command just modifies this file).

# Next meeting (10th November)

- Single-crystal parameter fitting workflow
- More VE generation tasks ported
- Generate pole figures from simulation increments
