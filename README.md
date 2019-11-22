# experiment-impact-tracker

The experiment-impact-tracker is meant to be a simple drop-in method to track energy usage, carbon emissions, and compute utilization of your system. Currently, on Linux systems with Intel chips (that support the RAPL powercap interface) and NVIDIA GPUs, we record: power draw from CPU and GPU, hardware information, python package versions, estimated carbon emissions information, etc. In California we even support realtime carbon emission information by querying caiso.org!

Once all this information is logged, you can generate an online appendix which shows off this information like seen here:

https://breakend.github.io/RL-Energy-Leaderboard/reinforcement_learning_energy_leaderboard/pongnoframeskip-v4_experiments/ppo2_stable_baselines,_default_settings/0.html

## Installation

To install:

```bash
git clone https://github.com/Breakend/experiment-impact-tracker.git
cd experiment-impact-tracker
pip install -e .
```

## Usage

Please go to the docs page for detailed info on the design, usage, and contributing: https://breakend.github.io/experiment-impact-tracker/ 

If you think the docs aren't helpful or need more expansion, let us know with a Github Issue!

Below are some short snippets that might help.

### Tracking
You just need to add a few lines of code!

```python
from experiment_impact_tracker.compute_tracker import ImpactTracker
tracker = ImpactTracker(<your log directory here>)
tracker.launch_impact_monitor()
```

This will launch a separate process (more like thread) that will gather compute/energy/carbon information in the background.

**NOTE: Because of the way python multiprocessing works, this process will not interrupt the main one UNLESS you periodically call the following. This will read the latest info from the log file and check for any errors that might've occured in the tracking process. If you have a better idea on how to handle exceptions in the tracking thread please open an issue or submit a pull request!!!** 

```python
info = tracker.get_latest_info_and_check_for_errors()
```


### Generating an HTML appendix

After putting all your experments into a folder, we can automatically search for the impact tracker's logs and generate an HTML appendix using the command like below.

First, create a json file with the structure of the website you'd like to see (this lets you create hierarchies of experiment as web pages).

For an example of all the capabilities of the tool you can see the json structure here: https://github.com/Breakend/RL-Energy-Leaderboard/blob/master/leaderboard_generation_format.json

Basically, you can group several runs together and specify variables to summarize.

```bash
"Comparing Translation Methods" : {
  "filter" : "(translation)", # this regex we use to look through the directory you specify and find experiments with this in the directory structure,
  "description" : "An experiment on translation.", # Use this to talk about your experiment
  # executive_summary_variables: this will aggregate the sums and averages across these metrics.
  # you can see available metrics to summarize here: 
  # https://github.com/Breakend/experiment-impact-tracker/blob/master/experiment_impact_tracker/data_info_and_router.py
  "executive_summary_variables" : ["total_power", "exp_len_hours", "cpu_hours", "gpu_hours", "estimated_carbon_impact_kg"],   
  "executive_summary_ordering_variable" : "estimated_carbon_impact_kg", # the variable used to rank experiments
  "child_experiments" : 
        {
            "Transformer Network" : {
                                "filter" : "(transformer)",
                                "description" : "A subset of experiments for transformer experiments"
                            },
            "Conv Network" : {
                                "filter" : "(conv)",
                                "description" : "A subset of experiments for conv experiments"
                            }
                   
        }
}
```


```bash
create-compute-appendix ./data/ --site_spec leaderboard_generation_format.json --output_dir ./site/
```

To see this in action, take a look at our RL Energy Leaderboard: https://github.com/Breakend/RL-Energy-Leaderboard

### Asserting certain hardware

It may be the case that you're trying to run two sets of experiments and compare emissions/energy/etc. In this case, you generally want to ensure that there's parity between the two sets of experiments. If you're running on a cluster you might not want to accidentally use a different GPU/CPU pair. To get around this we provided an assertion check that you can add to your code that will kill a job if it's running on a wrong hardware combo. For example:

```python
from experiment_impact_tracker.gpu.nvidia import assert_gpus_by_attributes
from experiment_impact_tracker.cpu.common import assert_cpus_by_attributes

assert_gpus_by_attributes({ "name" : "GeForce GTX TITAN X"})
assert_cpus_by_attributes({ "brand": "Intel(R) Xeon(R) CPU E5-2640 v3 @ 2.60GHz" })
```

## Building docs

```bash
sphinx-build -b html docsrc docs
```

## Compatible Systems

Right now, we're only compatible with Linux systems running NVIDIA GPU's and Intel processors (which support RAPL). 

If you'd like support for your use-case or encounter missing/broken functionality on your system specs, please open an issue or better yet submit a pull request! It's almost impossible to cover every combination on our own!

### Tested Successfully On

GPUs:
+ NVIDIA Titan X
+ NVIDIA Titan V

CPUs:
+ Intel(R) Xeon(R) CPU E5-2640 v3 @ 2.60GHz
+ Intel(R) Xeon(R) CPU E5-2620 v3 @ 2.40GHz

OS:
+ Ubuntu 16.04.5 LTS

## Citation

If you use this work, please cite our paper:

```
TODO
```

Also, we rely on a number of downstream packages and work to make this work possible. For carbon accounting, we relied on open source code from https://www.electricitymap.org/ as an initial base. psutil provides many of the compute metrics we use. nvidia-smi and Intel RAPL provide energy metrics. 

