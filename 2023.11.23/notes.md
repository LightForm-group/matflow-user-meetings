# Set up on the CSF

## Check version

- Check the version of MatFlow with `matflow-dev --version` (should be the latest version: `v0.3.0a101`).

## Check if you need to configure MatFlow on CSF3

- First check if already configured: run `matflow-dev config get --all` and see if "manchester-CSF3" is the loaded config. If not, continue:
  - Normally via the `matflow-configs` repo: https://github.com/hpcflow/matflow-configs
  - Since we still have no internet access on CSF3, we will specify the search directory as a shared location: `/mnt/eps01-rds/jf01-home01/shared/software/matflow_configs/2023.10.27`
  - Run `matflow-dev config init --path /mnt/eps01-rds/jf01-home01/shared/software/matflow_configs/2023.10.27 manchester`
  - Then run `matflow-dev config get --all` and the "manchester-CSF3" config should be loaded
 
## Clear known submissions

- A recent update requires us to clear our submissions history: `matflow-dev manage clear-known-subs`

# Notable recent changes

## Fixes

- fixed `demo-data` CLI
- fixed bug in `LoadStep.plane_strain`
- more accessible show command iconography

## Docs

- List of task schemas ported over: https://github.com/hpcflow/matflow-new/blob/develop/README.md
- Task schemas info on docs website improved (https://docs.matflow.io/stable/reference/template_components/task_schemas.html)
- added task schemas:
  - `sample_texture_from_model_ODF` (MTEX)
  - `generate_volume_element_from_statistics` (Dream3D) - but cannot pass `orientations` or `precipitates` yet
  - `generate_volume_element_extrusion` 
  - `load_microstructure_EBSD/_DIC` (DefDAP)  
  - `modify_VE_add_buffer_zones`
  - `visualise_VE_VTK`
- added workflow demos:
  - `tension_DAMASK_Ti`
  - `damask_input_files`
  - `damask_numerics`
  - `RVE_extrusion_EBSD`
  - `RVE_extrusion_EBSD_DIC`
  - `generate_volume_element_from_statistics`
  - `sample_texture_model_ODF`
- new ported parameter: `damask_solver`: https://docs.matflow.io/stable/reference/template_components/parameters.html#damask-solver
- new ported parameter: `damask_numerics`
  - can also see available options with `matflow demo-data copy numerics_example.yaml .`

# Demos

## New `show` command iconography

- Copy the demo workflow template `tension_DAMASK_Al` to your scratch directory and modify it to run on the short queue.
- You can copy a demo workflow template like this: `matflow-dev demo-workflow copy WORKFLOW_NAME DESTINATION`
- To use the short queue, add the following top-level block to the workflow template YAML file:

  ```yaml
  resources:
    any:
      scheduler_args:
        options: ["-l short"]
  ```

- Submit the workflow and use the `show`/`show -f` command to see the updated iconography!

## Passing input files directly

- Examine the actions of the `simulate_VE_loading` task schema on the [docs site](https://docs.matflow.io/stable/reference/template_components/task_schemas.html#simulate-ve-loading-damask)
- The first four actions are for generating DAMASK input files (geometry, load case, material, numerics)
- If we have these files already, we can pass them directly to MatFlow, and the relevant actions will be skipped
- Copy the demo workflow template `damask_numerics` to your scratch directory and add the same top-level resources block as above
- Look at how we can pass input files to MatFlow.
- Also add a resources block to the simulate task to use 2 cores:

  ```yaml
  resources:
    main:
      num_cores: 2
  ```

- Submit the workflow and notice how there are fewer actions, because the input files have already been generated.
- If you'd like to see what possible numerics parameters can be passed to the `damask_numerics` input parameter of the simulate task, you can copy the demonstration file like this (this file is copied from the DAMASK source code):

  ```
  matflow-dev demo-data copy numerics_example.yaml .
  ```

## Aborting a run

- In this part we will show how it's possible to abort an action that is running without cancelling the whole workflow. This is useful when running simulations; the simulation might have produced sufficient data and might not need to be run to completion.
- Not all actions are *abortable*, as there is a small computational overhead associated with allowing an action to be stopped without killing the whole workflow
- Modify the `tension_DAMASK_Al.yaml` workflow again:
  - Add a sequence on the `VE_grid_size` input of the `generate_volume_element_from_voronoi` task. Use two values: `[8, 8, 8]` and `[32, 32, 32]`.
- Submit the workflow
- Wait for the first element (the smaller grid size) to fully complete.
- Once the first element is completed, you can abort the second simulation like this:

  ```
  matflow-dev workflow WORKFLOW_ID abort-run
  ```
- Wait a few seconds, then use the `show -f` command to check the simulation run was aborted. The remaining post-processing actions should continue as normal.
