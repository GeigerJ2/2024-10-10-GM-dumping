# Roast my code: Data dumping in AiiDA


<details>
  <summary>A little fun fact about:</summary>

  ![dogfood](./figs/dogfood.png)

  It's got 60g / 100g of protein!!! 💪

  That's why today's GM will be all about dogfooding 🐶

</details>

<details>
  <summary>So let's get started!</summary>

![pr-conversation](./figs/pr-conversation.png)

</details>

## Dumping collection data

Let's first run with the defaults and see where we get:

```shell
verdi storage dump --path /home/geiger_j/aiida_projects/verdi-profile-dump/dev-dumps/storage-mirror/ -o
```

Next, let's check all the CLI options and their default values:

```shell
verdi storage dump -h
```

Some things to consider:
- `--path`: This is the global parent path for dumping. All different collections/entities will be dumped into specific
  subdirectories based on the locally modified `output_path` variable.
- `--overwrite`: Currently only global overwrite
   - Add `--incremental` option which is true by default, to gradually update the directory
- `organize_by_groups`:
    - If not set, only top-level `calculations`, `workflows`, and `data` are created in the dumping directory
      Organization in groups will be ignored in this scheme.
    - If set, entities that are in a certain group `group` will be put in this directory in a nested structure based on
      the group's `type_string` and label
- `--only-top-level-workflows`: With this diabled, also sub-workchains will be dumped into their respective directories.
  Otherwise only top-level workchains.
- `--dump-processes/--dump-data`: Dump `ProcessNode`s and `orm.Data` nodes. `--dump-processes` is `True`, while
  `--dump-data` is `False` by default, as `orm.Data` nodes are implicitly contained in the dumped process data. If it is
  `True`, also a structured dedicated `Data` directory is created. If specified, either one of `--also-raw` or
  `--also-rich` must also be set.
- `--calculations-hidden/--data-hidden`: True by default. If set, dump `CalcJob` data and `orm.Data` entities in a
  hidden directory using their UUIDs and symlink from the structured output directories.
- `--include-inputs`, `--include-outputs`, `--include-attriutes`, `--include-extras`: These are options from the
  `ProcessDumper`, to also dump linked input nodes and output nodes for each `CalcJob`, as well as if the `ProcessNode`
  attributes and extras should be dumped to the `.aiida_node_metadata.yaml` file in every `ProcessNode`s directory.
- `--flat`: This is also from the `ProcessDumper`, and, if activated, removes the `inputs`, `outputs`, `node_inputs` and
  `node_outputs` subdirectories for each `CalcJob`s directory
- `--dump-config-file`: Specify all settings for the dumping via a config file in YAML format:
  `verdi storage dump --config /home/geiger_j/aiida_projects/verdi-profile-dump/git-repos/aiida-core.worktrees/feature/verdi-profile-dump/src/aiida/tools/dumping/test-config-file.yaml`

## `raw` and `rich` data dumping

- `raw` data dumping just dumps the `attributes` of every `orm.Data` node
  - This is currently hard-coded such tat the resulting YAML files are always contained in the associated `CalcJob`
    directories, as I don't see it very valuable to have individual structured `data` directories that contain the
    `Node` attributes -> low-priority TODO

## Dumping processes

### Some examples

```shell
verdi process dump --also-raw --also-rich --include-outputs -o 914
```

#### Directories of the (almost) entire `PwBandsWorkChain`:

```shell
❯ /bin/tree -d -L 4 ./PwBandsWorkChain-914/
```

#### The first `PwCalculation`:

```shell
❯ /bin/tree ./PwBandsWorkChain-914/01-relax-PwRelaxWorkChain/01-iteration_01-PwBaseWorkChain/
```

#### `create_kpoints_from_distance`:

```shell
❯ /bin/tree ./PwBandsWorkChain-914/01-relax-PwRelaxWorkChain/01-iteration_01-PwBaseWorkChain/01-create_kpo
ints_from_distance/
```

#### `seekpath_structure_analysis`:

```shell
❯ /bin/tree ./PwBandsWorkChain-914/02-seekpath-seekpath_structure_analysis/
```

#### The actual band structure calculation:

```shell
❯ /bin/tree ./PwBandsWorkChain-914/04-bands-PwBaseWorkChain/01-iteration_01-PwCalculation/
```


### Dumping of remote data


## More on `raw` and `rich` data dumping


### Customization for `rich` dumping


#### Via the CLI


#### Via a config file


### Of groups


### Of the entire `storage`


## Known TODOs

### P1

- [ ] `incremental` option to properly gradually build up a directory, as more data is created for a profile
- [ ] Allow mixing of config file and command-line options? Should additional command line options overwrite settings in
config file?
    - Or, just ignore additional command-line options, and use the defaults, if values are not set in the file

### P2

- [ ] Allow selecting only certain entities, either via their PKs or by their types (e.g. orm.StructureData)
- [ ] 

## How everything plays together


## Code design: The different Dumpers

![excuse-me-sir-inheritance](/home/geiger_j/aiida_projects/verdi-profile-dump/2024-10-10-GM-dumping/figs/excuse-me-sir-inheritance.png)

