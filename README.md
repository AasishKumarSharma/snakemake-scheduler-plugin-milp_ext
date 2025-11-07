# Snakemake MILP Optimizer Scheduler Plugin

A **pip-installable** scheduler plugin for [Snakemake](https://snakemake.readthedocs.io/) that uses Mixed-Integer Linear Programming (MILP) to optimally schedule jobs across heterogeneous compute resources.

**Plugin Name**: `milp_ext` (Note: `milp` is reserved for Snakemake's internal scheduler)

## Features

* **Resource-Aware Scheduling**: Considers CPU cores, memory, GPU, custom resources, and I/O constraints
* **Multi-Objective Optimization**: Balances makespan vs. energy consumption with configurable weights
* **Feature Compatibility**: Ensures jobs run only on nodes with required hardware/software features
* **Historical Estimation**: Learns from past executions to refine runtime predictions
* **Graceful Fallbacks**: Falls back to greedy scheduling if MILP fails or times out
* **Plugin Auto-Discovery**: Integrates seamlessly via Snakemake's new plugin interface

## Installation

### From PyPI (when published)

```bash
pip install snakemake-scheduler-plugin-milp_ext
```

### From Source

```bash
git clone https://github.com/AasishKumarSharma/snakemake-scheduler-plugin-milp_ext.git
cd snakemake-scheduler-plugin-milp_ext
pip install .
```

For development:

```bash
pip install -e .[dev]
```

## Requirements

- Python ≥ 3.11
- snakemake-interface-scheduler-plugins ≥ 2.0.0
- pulp ≥ 2.0 (for MILP optimization)
- networkx ≥ 2.5 (for dependency analysis)
- pyyaml ≥ 5.0 (for configuration files)

## Quick Start

1. **Create a System Profile**: `system_profile.json` describes your compute clusters/nodes.

```json
{
  "clusters": {
    "local": {
      "nodes": {
        "cpu_node": {
          "resources": {
            "cores": 8,
            "memory_mb": 16384,
            "gpu_count": 0
          },
          "features": ["cpu", "x86_64", "avx2"],
          "properties": {
            "cpu_flops": 1e11,
            "memory_bandwidth_mbps": 25600
          }
        },
        "gpu_node": {
          "resources": {
            "cores": 16,
            "memory_mb": 32768,
            "gpu_count": 1,
            "gpu_memory_mb": 11264
          },
          "features": ["cpu", "gpu", "cuda", "x86_64"],
          "properties": {
            "cpu_flops": 2e11,
            "gpu_flops": 14e12,
            "memory_bandwidth_mbps": 51200
          }
        }
      }
    }
  }
}
```

2. **Run Snakemake with MILP Optimizer Scheduler**:

```bash
snakemake --scheduler milp_ext \
          --scheduler-milp_ext-system-profile system_profile.json \
          --cores 8
```

Or with plugin settings:

```bash
snakemake --scheduler milp_ext \
          --scheduler-milp_ext-time-limit 60 \
          --scheduler-milp_ext-fallback greedy \
          --cores 8
```

## Job Specification

Enhanced job specifications in your Snakefile:

```python
rule gpu_job:
    output: "results/gpu_output.txt"
    threads: 4
    resources:
        mem_mb=8192,
        runtime=30
    params:
        job_specification={
            "features": ["gpu", "cuda"],
            "resources": {
                "gpu_memory_mb": 4000,
                "input_size_mb": 1000,
                "output_size_mb": 500
            },
            "properties": {
                "gpu_flops": 1e12
            }
        }
    shell:
        "python gpu_processing.py {input} {output}"
```

## Configuration Options

### Plugin Settings (CLI)

- `--scheduler-milp_ext-system-profile`: Path to system profile JSON
- `--scheduler-milp_ext-scheduler-config`: Path to scheduler config YAML
- `--scheduler-milp_ext-time-limit`: MILP solver time limit in seconds (default: 30)
- `--scheduler-milp_ext-fallback`: Fallback scheduler (default: "greedy")

## Testing

Run the test suite:

```bash
pytest tests/
```

---

## Contributing

1. Fork and clone the repo
2. Create a branch: `git checkout -b feature/new-flag`
3. Implement changes and add tests
4. Ensure all tests pass: `pytest`
5. Submit a pull request

Please follow [Conventional Commits](https://www.conventionalcommits.org/) for commit messages.

---

## Reference

If you use this repository or its code, please cite the following paper:

> **Aasish Kumar Sharma, Julian Kunkel et. al.**  
> *Workflow-Driven Modeling for the Compute Continuum: An Optimization Approach to Automated System and Workload Scheduling*  
> In: *Proceedings of the 2025 IEEE 49th Annual Computers, Software, and Applications Conference (COMPSAC)*, IEEE, 2025.  
> DOI: [10.1109/COMPSAC65507.2025.00343](https://doi.org/10.1109/COMPSAC65507.2025.00343)  
> arXiv: [2506.00260](https://arxiv.org/abs/2506.00260)

📘 Official IEEE Xplore entry:  
[https://ieeexplore.ieee.org/document/11126665](https://ieeexplore.ieee.org/document/11126665)

---

## Citation

```bibtex
@inproceedings{sharma2025workflow,
  author = {Aasish Kumar Sharma; Christian Boehme; Patrick Gelß; Ramin Yahyapour; Julian Kunkel},
  title = {Workflow-Driven Modeling for the Compute Continuum: An Optimization Approach to Automated System and Workload Scheduling},
  booktitle = {2025 IEEE 49th Annual Computers, Software, and Applications Conference (COMPSAC)},
  year = {2025},
  publisher = {IEEE},
  doi = {10.1109/COMPSAC65507.2025.00343},
  url = {https://doi.org/10.1109/COMPSAC65507.2025.00343}
}
```




## License

This project is licensed under the MIT License.
