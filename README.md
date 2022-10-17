<a href="https://explosion.ai"><img src="https://explosion.ai/assets/img/logo.svg" width="125" height="125" align="right" /></a>

# spacy-loggers: Logging utilities for spaCy

[![PyPi Version](https://img.shields.io/pypi/v/spacy-loggers.svg?style=flat-square&logo=pypi&logoColor=white)](https://pypi.python.org/pypi/spacy-loggers)

Starting with spaCy v3.2, alternate loggers are moved into a separate package
so that they can be added and updated independently from the core spaCy
library.

`spacy-loggers` currently provides loggers for:

- [Weights & Biases](https://www.wandb.com)
- [MLflow](https://www.mlflow.org/)
- [ClearML](https://www.clear.ml/)

If you'd like to add a new logger or logging option, please submit a PR to this
repo!

## Setup and installation

`spacy-loggers` should be installed automatically with spaCy v3.2+, so you
usually don't need to install it separately. You can install it with `pip` or
from the conda channel `conda-forge`:

```bash
pip install spacy-loggers
```

```bash
conda install -c conda-forge spacy-loggers
```

# Loggers

## WandbLogger

### Installation

This logger requires `wandb` to be installed and configured:

```bash
pip install wandb
wandb login
```

### Usage

`spacy.WandbLogger.v4` is a logger that sends the results of each training step
to the dashboard of the [Weights & Biases](https://www.wandb.com/) tool. To use
this logger, Weights & Biases should be installed, and you should be logged in.
The logger will send the full config file to W&B, as well as various system
information such as memory utilization, network traffic, disk IO, GPU
statistics, etc. This will also include information such as your hostname and
operating system, as well as the location of your Python executable.

**Note** that by default, the full (interpolated)
[training config](https://spacy.io/usage/training#config) is sent over to the
W&B dashboard. If you prefer to **exclude certain information** such as path
names, you can list those fields in "dot notation" in the
`remove_config_values` parameter. These fields will then be removed from the
config before uploading, but will otherwise remain in the config file stored
on your local system.

### Example config

```ini
[training.logger]
@loggers = "spacy.WandbLogger.v4"
project_name = "monitor_spacy_training"
remove_config_values = ["paths.train", "paths.dev", "corpora.train.path", "corpora.dev.path"]
log_dataset_dir = "corpus"
model_log_interval = 1000
```

| Name                   | Type            | Description                                                                                                                                                                                                                     |
| ---------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `project_name`         | `str`           | The name of the project in the Weights & Biases interface. The project will be created automatically if it doesn't exist yet.                                                                                                   |
| `remove_config_values` | `List[str]`     | A list of values to exclude from the config before it is uploaded to W&B (default: `[]`).                                                                                                                                       |
| `model_log_interval`   | `Optional[int]` | Steps to wait between logging model checkpoints to the W&B dasboard (default: `None`). Added in `spacy.WandbLogger.v2`.                                                                                                         |
| `log_dataset_dir`      | `Optional[str]` | Directory containing the dataset to be logged and versioned as a W&B artifact (default: `None`). Added in `spacy.WandbLogger.v2`.                                                                                               |
| `run_name`             | `Optional[str]` | The name of the run. If you don't specify a run name, the name will be created by the `wandb` library (default: `None`). Added in `spacy.WandbLogger.v3`.                                                                       |
| `entity`               | `Optional[str]` | An entity is a username or team name where you're sending runs. If you don't specify an entity, the run will be sent to your default entity, which is usually your username (default: `None`). Added in `spacy.WandbLogger.v3`. |
| `log_best_dir`         | `Optional[str]` | Directory containing the best trained model as saved by spaCy (by default in `training/model-best`), to be logged and versioned as a W&B artifact (default: `None`). Added in `spacy.WandbLogger.v4`.                           |
| `log_latest_dir`       | `Optional[str]` | Directory containing the latest trained model as saved by spaCy (by default in `training/model-latest`), to be logged and versioned as a W&B artifact (default: `None`). Added in `spacy.WandbLogger.v4`.                       |

## MLflowLogger

### Installation

This logger requires `mlflow` to be installed and configured:

```bash
pip install mlflow
```

### Usage

`spacy.MLflowLogger.v1` is a logger that tracks the results of each training step
using the [MLflow](https://www.mlflow.org/) tool. To use
this logger, MLflow should be installed. At the beginning of each model training
operation, the logger will initialize a new MLflow run and set it as the active
run under which metrics and parameters wil be logged. The logger will then log
the entire config file as parameters of the active run. After each training step,
the following actions are performed:

- The final score is logged under the metric `score`.
- Individual component scores are logged under their default names.
- Loss values of different components are logged with the `loss_` prefix.
- If the final score is higher than the previous best score (for the current run),
  the model artifact is additionally uploaded to MLflow. This action is only performed
  if the `output_path` argument is provided during the training pipeline initialization phase.

By default, the tracking API writes data into files in a local `./mlruns` directory.

**Note** that by default, the full (interpolated)
[training config](https://spacy.io/usage/training#config) is sent over to 
MLflow. If you prefer to **exclude certain information** such as path
names, you can list those fields in "dot notation" in the
`remove_config_values` parameter. These fields will then be removed from the
config before uploading, but will otherwise remain in the config file stored
on your local system.

### Example config

```ini
[training.logger]
@loggers = "spacy.MLflowLogger.v1"
experiment_id = "1"
run_name = "with_fast_alignments"
nested = False
remove_config_values = ["paths.train", "paths.dev", "corpora.train.path", "corpora.dev.path"]
```

| Name                   | Type                       | Description                                                                                                                                                                                                             |
| ---------------------- | -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `run_id`               | `Optional[str]`            | Unique ID of an existing MLflow run to which parameters and metrics are logged. Can be omitted if `experiment_id` and `run_id` are provided (default: `None`).                                                          |
| `experiment_id`        | `Optional[str]`            | ID of an existing experiment under which to create the current run. Only applicable when `run_id` is `None` (default: `None`).                                                                                          |
| `run_name`             | `Optional[str]`            | Name of new run. Only applicable when `run_id` is `None` (default: `None`).                                                                                                                                             |
| `nested`               | `bool`                     | Controls whether run is nested in parent run. `True` creates a nested run (default: `False`).                                                                                                                           |
| `tags`                 | `Optional[Dict[str, Any]]` | A dictionary of string keys and values to set as tags on the run. If a run is being resumed, these tags are set on the resumed run. If a new run is being created, these tags are set on the new run (default: `None`). |
| `remove_config_values` | `List[str]`                | A list of values to exclude from the config before it is uploaded to MLflow (default: `[]`).                                                                                                                            |

## ClearMLLogger

### Installation

This logger requires `clearml` to be installed and configured:

```bash
pip install clearml
clearml-init
```

### Usage

`spacy.ClearMLLogger.v1` is a logger that tracks the results of each training step
using the [ClearML](https://www.clear.ml/) tool. To use
this logger, ClearML should be installed and you should have initialized (using the command above).
The logger will send all the gathered information to your ClearML server, either [the hosted free tier](https://app.clear.ml) 
or the open source [self-hosted server](https://github.com/allegroai/clearml-server). This logger captures all the following information (all visible in the web UI):

- The full spaCy config file contents
- Code information such as git repository, commit ID and uncommitted changes.
- Full console output
- Miscellaneous info such as time, python version and hardware information
- Output scalars:
    - The final score is logged under the scalar `score`.
    - Individual component scores are grouped together on one scalar plot, use the web UI to filter.
    - Loss values of different components are logged with the `loss_` prefix.

Then optionally, depending on the setting below, the following can be captured:

- Best model directory (zipped)
- Latest model directory (zipped)
- Dataset used to train (will be versioned using ClearML Data and linked to under Configuration -> User Properties)


**Note** that by default, the full (interpolated)
[training config](https://spacy.io/usage/training#config) is sent over to 
MLflow. If you prefer to **exclude certain information** such as path
names, you can list those fields in "dot notation" in the
`remove_config_values` parameter. These fields will then be removed from the
config before uploading, but will otherwise remain in the config file stored
on your local system.

### Example config

```ini
[training.logger]
@loggers = "spacy.ClearMLLogger.v1"
project_name = "Hello ClearML!"
model_log_interval = 1000
log_best_dir = training/model-best
log_latest_dir = training/model-last
log_dataset_dir = corpus
remove_config_values = ["paths.train", "paths.dev", "corpora.train.path", "corpora.dev.path"]
```

| Name                   | Type            | Description                                                                                                                                                                                                                     |
| ---------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `project_name`         | `str`           | The name of the project in the ClearML interface. The project will be created automatically if it doesn't exist yet.                                                                                                            |
| `task_name`            | `Optional[str]` | The name of the ClearML task. A task is an experiment that lives inside a project. Can be non-unique. (default: `spaCy Training`).                                                                                              |
| `remove_config_values` | `List[str]`     | A list of values to exclude from the config before it is uploaded to ClearML (default: `[]`).                                                                                                                                   |
| `model_log_interval`   | `Optional[int]` | Steps to wait between logging model checkpoints to the ClearML dasboard (default: `None`). Will have no effect without also setting `log_best_dir` or `log_latest_dir`.                                                         |
| `log_best_dir`         | `Optional[str]` | Directory containing the best trained model as saved by spaCy (by default in `training/model-best`), to be logged and versioned as a ClearML artifact (default: `None`)                                                         |
| `log_latest_dir`       | `Optional[str]` | Directory containing the latest trained model as saved by spaCy (by default in `training/model-last`), to be logged and versioned as a ClearML artifact (default: `None`)                                                       |
| `log_dataset_dir`      | `Optional[str]` | Directory containing the dataset to be logged and versioned as a [ClearML Dataset](https://clear.ml/docs/latest/docs/clearml_data/clearml_data/) (default: `None`).                                                             |
