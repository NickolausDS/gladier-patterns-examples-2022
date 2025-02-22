# Gladier Client Examples

Our paper presents 5 clients

* [XPCS]()
* [SSX]()
* [HEDM]()
* [BRAGGNN]()
* [PTYCHOGRAPHY]()

Although each has its own particular set of tools, they share common patterns and leverage the same services. Initial setup can be installed follow the PAPER-HOW-TO.md manual.

## Test Data

By default, test data is pulled from [globus endpoint](https://app.globus.org/file-manager?origin_id=a17d7fac-ce06-4ede-8318-ad8dc98edd69&origin_path=%2F~%2F). Data is available for
XPCS, SSX, and ptychography.

## FuncX Endpoint Setup

For these examples, A "compute" machine is needed to stand in as the HPC environment, where data will be transfered and processed. It does not need to be powerful, but we do recommend linux (preferably Ubuntu)
and Conda (for python). A Globus collection is also required (Globus Connect Personal is recommended)

**Note**: There are current issues using macs as FuncX Endpoints. We highly recommend using Linux instead.

```bash

# Install the necessary components
conda create -n gladier_demo_remote python=3.9
conda activate gladier_demo_remote
pip install funcx-endpoint

# Setup your FuncX "login" endpoint - this is used for organization tasks
# Use the generated UUID for "funcx_endpoint_non_compute" states
funcx-endpoint configure login
funcx-endpoint start login

# Setup your FuncX "compute" endpoint - this is typically used for computationally expensive tasks
# Use the generated UUID for "funcx_endpoint_compute" states
funcx-endpoint configure compute
funcx-endpoint start compute
```

The two FuncX Endpoints `funcx_endpoint_compute` and `funcx_endpoint_non_compute` simulate an HPC environment. "Compute" endpoints are not required to have internet access, but must be able to
handle large CPU loads. `funcx_endpoint_non_compute` (login/head nodes) are not required to handle
large CPU loads, but they do require internet access. For these examples, you are welcome to use
one FuncX endpoint for both if it satisfies both constraints.

Ensure you have both a Globus Collection UUID, and your FuncX Endpoint(s) setup:

```
funcx-endpoint list
+----------------+--------------+--------------------------------------+
| Endpoint Name  |    Status    |             Endpoint ID              |
+================+==============+======================================+
+----------------+--------------+--------------------------------------+
| login          | Running      | 12345678-0480-473c-beef-be762ba979a9 |
+----------------+--------------+--------------------------------------+
| compute        | Running      | abcdefgh-0454-4af1-97ec-012771c869f9 |
+----------------+--------------+--------------------------------------+
```

### FuncX on HPC systems

In an HPC environment, you may also want to modify your funcX endpoint configuration to provision compute nodes. Several configuration examples can be found at the [funcx endpoints documentation](https://funcx.readthedocs.io/en/latest/endpoints.html).

## Gladier Setup

Gladier is used for registering FuncX functions and deploying flows. Below are steps to
setup funcx-endpoints and a Globus Collection. After this, running the test client below
will be possible. The test client, unlike the scientific tools, requires no additional external dependencies.


You will need to edit the test_client.py script to include your
FuncX endpoints, along with a Globus Collection. Note: Your
FuncX endpoints _must_ have access to the Globus Collection you use.

You may install these in a separate python environment or a separate machine
if you wish:

```
pip install glaider gladier-tools
```

Test your basic setup by running the test_client.py:

```bash

python test_client.py
```

Running the test client will prompt you to login and will direct you to the running flow in the Globus Web app. Ensure you see output similar to the following:

```
python test_client.py
Run started, you can also track the progress at:
https://app.globus.org/runs/1742ee41-3e14-4e5e-a191-149857f2ccea
[ACTIVE]: The Flow is starting execution
[ACTIVE]: State TransferFromStorage of type Action started
...
[ACTIVE]: State ShellCmd of type Action started
...
The flow completed with the status: SUCCEEDED
Output: [0, 'Success! You environment has been setup correctly!\n', '']
```

Now you're ready to run the other science flows.

## Ptychography

The ptychography flow uses a shell command tool to execute the `ptychodus` tool on the example data.

Refer to the python environment used for your FuncX Endpoint above:

```
| compute        | Running      | abcdefgh-0454-4af1-97ec-012771c869f9 |
```

Your compute endpoint will require the following dependencies:

```
#Ptycho tools
git clone https://github.com/AdvancedPhotonSource/ptychodus
cd ptychodus
conda install -c conda-forge --file requirements-dev.txt
conda install -c conda-forge tike
pip install -e . 
```

Before you run `ptychodus_client.py`, remember to restart your FuncX compute endpoint and set
the values in the script below.

```bash

python ptychodus_client.py --datadir <data path>
```

## XPCS flow

The XPCS flow uses the boost_corr python sdk for execution, and requires the following dependencies
for its compute endpoint:

```bash
#XPCS flow
conda install -c nvidia cudatoolkit
conda install -c pytorch pytorch
pip install -e git+https://github.com/AZjk/boost_corr#egg=boost_corr
```


Remember to restart your compute endpoint and add the values to the `xpcs_client.py` script.

```bash

python xpcs_client.py --datadir <data path>
```