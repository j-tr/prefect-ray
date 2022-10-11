# prefect-ray

<p align="center">
    <a href="https://pypi.python.org/pypi/prefect-ray/" alt="PyPI version">
        <img alt="PyPI" src="https://img.shields.io/pypi/v/prefect-ray?color=0052FF&labelColor=090422"></a>
    <a href="https://github.com/PrefectHQ/prefect-ray/" alt="Stars">
        <img src="https://img.shields.io/github/stars/PrefectHQ/prefect-ray?color=0052FF&labelColor=090422" /></a>
    <a href="https://pepy.tech/badge/prefect-ray/" alt="Downloads">
        <img src="https://img.shields.io/pypi/dm/prefect-ray?color=0052FF&labelColor=090422" /></a>
    <a href="https://github.com/PrefectHQ/prefect-ray/pulse" alt="Activity">
        <img src="https://img.shields.io/github/commit-activity/m/PrefectHQ/prefect-ray?color=0052FF&labelColor=090422" /></a>
    <br>
    <a href="https://prefect-community.slack.com" alt="Slack">
        <img src="https://img.shields.io/badge/slack-join_community-red.svg?color=0052FF&labelColor=090422&logo=slack" /></a>
    <a href="https://discourse.prefect.io/" alt="Discourse">
        <img src="https://img.shields.io/badge/discourse-browse_forum-red.svg?color=0052FF&labelColor=090422&logo=discourse" /></a>
</p>

## Welcome!

Prefect integrations with the [Ray](https://www.ray.io/) execution framework, a flexible distributed computing framework for Python.

Provides a `RayTaskRunner` that enables flows to run tasks requiring parallel execution using Ray.

## Getting Started

### Python setup

Requires an installation of Python 3.7+.

We recommend using a Python virtual environment manager such as pipenv, conda, or virtualenv.

These tasks are designed to work with Prefect 2.0. For more information about how to use Prefect, please refer to the [Prefect documentation](https://orion-docs.prefect.io/).

### Installation

Install `prefect-ray` with `pip`:

```bash
pip install prefect-ray
```

Users running Apple Silicon (such as M1 macs) will need to additionally run:
```
pip uninstall grpcio
conda install grpcio
```
Click [here](https://docs.ray.io/en/master/ray-overview/installation.html#m1-mac-apple-silicon-support) for more details.

## Running tasks on Ray

The `RayTaskRunner` is a [Prefect task runner](https://orion-docs.prefect.io/concepts/task-runners/) that submits tasks to [Ray](https://www.ray.io/) for parallel execution. 

By default, a temporary Ray instance is created for the duration of the flow run.

For example, this flow counts to 3 in parallel.

```python
import time

from prefect import flow, task
from prefect_ray import RayTaskRunner

@task
def shout(number):
    time.sleep(0.5)
    print(f"#{number}")

@flow(task_runner=RayTaskRunner)
def count_to(highest_number):
    for number in range(highest_number):
        shout.submit(number)

if __name__ == "__main__":
    count_to(10)

# outputs
#3
#7
#2
#6
#4
#0
#1
#5
#8
#9
```

If you already have a Ray instance running, you can provide the connection URL via an `address` argument.

To configure your flow to use the `RayTaskRunner`:

1. Make sure the `prefect-ray` collection is installed as described earlier: `pip install prefect-ray`.
2. In your flow code, import `RayTaskRunner` from `prefect_ray.task_runners`.
3. Assign it as the task runner when the flow is defined using the `task_runner=RayTaskRunner` argument.

For example, this flow uses the `RayTaskRunner` with a local, temporary Ray instance created by Prefect at flow run time.

```python
from prefect import flow
from prefect_ray.task_runners import RayTaskRunner

@flow(task_runner=RayTaskRunner())
def my_flow():
    ... 
```

This flow uses the `RayTaskRunner` configured to access an existing Ray instance at `ray://192.0.2.255:8786`.

```python
from prefect import flow
from prefect_ray.task_runners import RayTaskRunner

@flow(task_runner=RayTaskRunner(address="ray://192.0.2.255:8786"))
def my_flow():
    ... 
```

`RayTaskRunner` accepts the following optional parameters:

| Parameter | Description |
| --- | --- |
| address | Address of a currently running Ray instance, starting with the [ray://](https://docs.ray.io/en/master/cluster/ray-client.html) URI. |
| init_kwargs | Additional kwargs to use when calling `ray.init`. |

Note that Ray Client uses the [ray://](https://docs.ray.io/en/master/cluster/ray-client.html) URI to indicate the address of a Ray instance. If you don't provide the `address` of a Ray instance, Prefect creates a temporary instance automatically.

!!! warning "Ray environment limitations"
    While we're excited about adding support for parallel task execution via Ray to Prefect, there are some inherent limitations with Ray you should be aware of:
    
    Ray currently does not support Python 3.10.

    Ray support for non-x86/64 architectures such as ARM/M1 processors with installation from `pip` alone and will be skipped during installation of Prefect. It is possible to manually install the blocking component with `conda`. See the [Ray documentation](https://docs.ray.io/en/latest/ray-overview/installation.html#m1-mac-apple-silicon-support) for instructions.

    See the [Ray installation documentation](https://docs.ray.io/en/latest/ray-overview/installation.html) for further compatibility information.

## Running tasks on a Ray remote cluster

When using the `RayTaskRunner` with a remote Ray cluster, you may run into issues that are not seen when using a local Ray instance. To resolve these issues, we recommend taking the following steps when working with a remote Ray cluster:

1. If your task fails with permission or file not found errors, set `PREFECT_LOCAL_STORAGE_PATH` in your Prefect settings to a path accessible on both the remote cluster and the machine executing the flow:
```bash
prefect config set PREFECT_LOCAL_STORAGE_PATH='/tmp/prefect/storage'
```

2. If you get an error stating that the module 'prefect' cannot be found, ensure `prefect` is installed on the remote cluster, with:
```bash
pip install prefect
```

3. If you are seeing timeout or other connection errors, double check the address provided to the `RayTaskRunner`. The address should look similar to: `address='ray://<head_node_ip_address>:10001'`:
```bash
RayTaskRunner(address="ray://1.23.199.255:10001")
```

## Resources

If you encounter and bugs while using `prefect-ray`, feel free to open an issue in the [prefect-ray](https://github.com/PrefectHQ/prefect-ray) repository.

If you have any questions or issues while using `prefect-ray`, you can find help in either the [Prefect Discourse forum](https://discourse.prefect.io/) or the [Prefect Slack community](https://prefect.io/slack).

Feel free to ⭐️ or watch [`prefect-ray`](https://github.com/PrefectHQ/prefect-ray) for updates too!

## Development

If you'd like to install a version of `prefect-ray` for development, clone the repository and perform an editable install with `pip`:

```bash
git clone https://github.com/PrefectHQ/prefect-ray.git

cd prefect-ray/

pip install -e ".[dev]"

# Install linting pre-commit hooks
pre-commit install
```
