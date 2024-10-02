# Aiida-Workflow

## What is AiidA?

[AiiDA](https://www.aiida.net/index.html) is an open-source Python infrastructure to help researchers automate, manage, persist, share, and reprogram the complex workflows associated with modern computational science and all associated data. It is very helpful framework for high-throughput calculations.

## Why AiidA? 
We increasingly have access to more computational resources that are capable of handling larger volumes of calculations, and high throughput calculation is more feasible. Imagine having to create, execute, and error-check for over 1000 calculations by "hand". That would be awful! With Aiida, you can automate many of these procedures and use your brain power for analysis, interpretation of results, and drawing conclusions. 

## AiiDA Quantum ESPRESSO
Aiida has many plugins for popular software including [VASP](https://aiidateam.github.io/aiida-registry/aiida-vasp), [Gaussian](https://aiidateam.github.io/aiida-registry/aiida-gaussian), and [Wannier90](https://aiidateam.github.io/aiida-registry/aiida-wannier90). There is currently a plugin for [Quantum Espresso](https://aiida-quantumespresso.readthedocs.io/en/latest/) that integrates the Quantum ESPRESSO software suite. Compute a variety of material properties with the popular open-source DFT code with automatic [data provenance](https://www.nnlm.gov/guides/data-glossary/data-provenance#:~:text=Definition,to%20where%20it%20is%20presently) provided by AiiDA. 

## Installing AiidA on a local computer (Mac OS) 

1. Create a conda environment
   
`Conda create -n aiida python==3.11`

2. Now activate the environment so we can install aiida within this active environment

`Conda activate aiida`

3. Change into what ever folder you would like your aiida installation to be in. Then make an "aiida" directory within that folder.

```
cd 'what/ever/directory/you/want/aiida/to/be/in' 

mkdir aiida
```

4. Git clone aiida in that directory. After, change into the newly created directory. 
```
git clone https://github.com/aiidateam/aiida-core.git 
cd aiida-core
```

5. Install aiida within the current directory.

```
pip install -e .
```

The "-e" means that we would like to make your installation editable. The "." means we would like it to be installed in this current directory. Look [here](https://pip.pypa.io/en/stable/cli/pip_install/) for more information on pip install commands or type "pip --h". 

### Installing a messaging broker, Rabbitmq (Mac OS)

 RabbitMQ is a messaging and streaming broker. It accepts messages from publishers/applications, routes them and, if there were queues to route to, stores them for consumption or immediately delivers to consumers, if any. It will also retain all sent messages in a queue the interim of a dropped connection. Learn more about Rabbitmq [here](https://www.rabbitmq.com/).
First, install rabbitmq using homebrew. 
```
brew install rabbitmq
```
Once installed, we have to complete configuring your profile. Open the rabbitmq-env.conf file and copy the path to your config file, mostlikely named "rabbitmq" and can be found under the card/variable "CONFIG_FILE=".

```
vi /opt/homebrew/etc/rabbitmq/rabbitmq-env.conf
```
![alt text](image-3.png)

Now, change into the directory where your config file is located. 
```
cd /opt/homebrew/etc/rabbitmq/

vi rabbitmq
```
Once in the rabbitmq file, add the following language to indicate the timeout length for a connection. You can further configure your profile. To learn more about that refer to this [page](https://www.rabbitmq.com/docs/configure).  

```
# in milliseconds, below is a little over 95 years
consumer_timeout = 3000000000000
```
Now, save and exit the file. Restart rabbitmq's services. 



```
brew service restart rabbitmq
```

Change into rabbitmq's executable directory. On Mac, that is usually in Homebrew's "Cellar" (the directory where all applications/libraries/software within Homebrew is stored).  Finally, execute diagnostics to ensure Rabbitmq's messaging services are all operational. 

```
# or what every version of rabbitmq
cd /opt/homebrew/Cellar/rabbitmq/3.13.7/sbin/ 

./rabbitmq-diagnostics status
```

7. Call verdi, an [AiidA subcommand](https://aiida.readthedocs.io/projects/aiida-core/en/latest/reference/command_line.html#commands), to check if aiida and rabbitmq are interfacing. Learn more about verdi presto and controlling your installation [here](https://aiida.readthedocs.io/projects/aiida-core/en/latest/reference/command_line.html#reference-command-line-verdi-presto). 



### Quick Set up
If you want a quick setup, call verdi presto and check if the status of the verdi and how it is interfacing with aiida and Rabbbitmq.

```
# calling the line below quickly creates and configures a new profile
verdi presto
verdi status
```
If you are receiving a warning message and find them annoying, then you can execute the optional code below. 

```
verdi config set warnings.rabbitmq_version false
verdi status
```
### Custom Profile Setup with Verdi 

If you need to configure your verdi config file, navigate to your AiiDA configuration file. 

```
cd #into your home directory 
cd .aiida
vi config.json
```
## Yaml files for straightforward configurations

Make a yaml file for your verdi profile. If you are curious about the database (db) engine and backend look [here](https://pypi.org/project/psycopg2/)

```
---
non_interactive: y
profile: profile_name
email: your_email
first_name: First
last_name: Last
institution:
set_default: true 
database_engine: core.psql_dos
database_hostname: host.name
database_port: port
database_name: your_db_name
database_username: username
database_password: password
broker_protocol: amqps #or amqp depending on system
broker_username: username
broker_password: password
broker_host: broker.host.name
broker_port: port
broker_virtual_host: aiida # could be empty on local computer but necessary on a supercomputer
repository: /path/to/where/you/want/the/repository # read more 
 ```

### Create a connection to a supercomputer

Create yaml file for the computer for aiida to reference configuration to interface with a supercomputer. 

Make sure to create aiida folder before executing config file. If you need to load any modules, do so into the 'prepend_text' line. 

Below we are creating yaml files for each compute node available at Penn State.

```
label:roar_basic
hostname: "submit.hpc.psu.edu"
description: "Roar Collab  basic nodes at Penn State"
transport: core.ssh
scheduler: "core.slurm"
work_dir: "/LOCATION/OF/SCRATCH/scratch/aiiDA" #make Aiida folder beforehand
mpirun_command: "srun -n {tot_num_mpiprocs}"
mpiprocs_per_machine: "24"
prepend_text: ""
append_text: " "
shebang: "#!/bin/bash"
```
```
label:roar_standard
hostname: "submit.hpc.psu.edu"
description: "Roar Collab  standard nodes at Penn State"
transport: core.ssh
scheduler: "core.slurm"
work_dir: "/LOCATION/OF/SCRATCH/scratch/aiiDA" #make Aiida folder beforehand
mpirun_command: "srun -n {tot_num_mpiprocs}"
mpiprocs_per_machine: "20"
prepend_text: ""
append_text: " "
shebang: "#!/bin/bash"
```
```
label:roar_high
hostname: "submit.hpc.psu.edu"
description: "Roar Collab high nodes at Penn State"
transport: core.ssh
scheduler: "core.slurm"
work_dir: "/LOCATION/OF/SCRATCH/scratch/aiiDA" #make Aiida folder beforehand
mpirun_command: "srun -n {tot_num_mpiprocs}"
mpiprocs_per_machine: "40"
prepend_text: ""
append_text: " "
shebang: "#!/bin/bash"
```
**!!** Only use below with GPU enabled codes!
```
label:roar_gpu
hostname: "submit.hpc.psu.edu"
description: "Roar Collab gpu nodes at Penn State"
transport: core.ssh
scheduler: "core.slurm"
work_dir: "/LOCATION/OF/SCRATCH/scratch/aiiDA" #make Aiida folder beforehand
mpirun_command: "srun -n {tot_num_mpiprocs}"
mpiprocs_per_machine: "2"
prepend_text: ""
append_text: " "
shebang: "#!/bin/bash"
```

To finish configuring your profile, call verdi computer setup.  

```
verdi computer setup --config path/to/roar_basic.yaml
```
**Note**: before the computer can be used, it has to be configured with the command:

`verdi -p presto computer configure core.ssh roar_basic`

Create this file to further configure your user information.  
```
---
username: "YOUR USER NAME"
port: 22
look_for_keys: true
key_filename: "/storage/home/LOCATION OF YOUR ID_RSA/.ssh/id_rsa"
timeout: 60 #minutes
allow_agent: true
proxy_command: ""
compress: true
gss_auth: false
gss_kex: false
gss_deleg_creds: false
gss_host: "roar_basic" #set to any computer you are using
load_system_host_keys: true
key_policy: "RejectPolicy"
use_login_shell: true
safe_interval: 10.0 #minutes
non_interactive: false
```




## Installation of QE of Mac OS 
### QE for AiidA installation 

It can be helpful to make a bash script to install quantum espresso. If you would prefer to make one proceed with the following steps. Note: all of the steps within the script can be called through the command line. 

1. Make build.sh an installation script within your quantum espresso software directory. 

`vi build.sh`

Inside the script, build.sh should look like this:
```
/!#bin/bash

#set the version of QE that you would like to build with version variable
VERSION=qe-7.3.1 

#It will clone it to folder named the version variable
git clone https://github.com/QEF/q-e.git $VERSION 

cd $VERSION

git checkout $VERSION

if [ -d build ]; then
        rm -rf build
fi

mkdir build
cd build

#Set your chosen complilers 
cmake -DCMAKE_C_COMPILER=mpicc -DCMAKE_Fortran_COMPILER=mpif90 ..

# Make all of the executables within the QE suite, this is with 1 processor (j)
make -j all

```

Exit the file and convert the build.sh into an excutable. Then execute build.sh
```
chmod +x build.sh
.build.sh
```
Quantum espresso should find all the necessary files and libraries needed for compiling the code. On a linux you may need to export your libraries BLAS, LAPACK, Scalapack, FTTW. If intel, ompi.

**!!** If you get a compilation error including regarding "foffload"for Mac OS, change into your cmake library and remove the f in front of "foffload". 
```
cd cmake
vi GNdcFortranCompiler.cmake 
/%s/foffload/offload/g
```








## Installing AiidA-Quantumespresso Plugin
Finally, we can install the AiidA plugin for Quantum Espresso. Look [here](https://github.com/aiidateam/aiida-quantumespresso/tree/b79189d7ce4756e846ab39c567ba4681474741ed) for their GitHub page. 
```
pip install aiida-quantumespresso 

# just to show available workflows for quantum espresso
verdi plugin list aiida.calculations
verdi plugin list aiida.workflows 
```

Set up pseudo library families with set level of precision and description. We will install the PseudoDojo pseudopotential family. To see a list of all available options for your pseudo family, type the below.  

```
aiida-pseudo install pseudo-dojo -h    
```

![alt text](image-1.png)

Below we install a PseudoDojo pseudopotential family that is from version 0.4, PBEsol functional, has a strict convergence threshold, and high default stringency level. 

`aiida-pseudo install pseudo-dojo -v 0.4 -x PBEsol -p stringent -f upf -s high`

The same procedure for installing the SSSP pseudopotential family from Materials Cloud. 
```
aiida-pseudo install sssp -h    
aiida-pseudo install sssp -v 1.3 -x PBEsol -p precision
```

```
#call the path to your selected pseudo-family
PseudoDojo/0.4/PBEsol/SR/stringent/upf
SSSP/1.3/PBEsol/precision
```




Now, install the blah blah and install the config file to the selected computer


```
 verdi -p presto computer configure core.ssh roar_basic --config conf_to_roar.yaml
```
Test that AiiDA access the selected computer. This works because there is an established connection open currently but you will need to login within 24 hours. 

```
 verdi computer test roar_basic 
```


```
vi code_qe_roar-basic.yaml
verdi code create core.code.installed --config code_qe_roar-basic.yaml
```
## Run example
 
Below is an example "workflow" that will run a quantum espresso calculation with of Si using SSSP pseudopotential family. It will reference your default AiiDA profile loaded in with verdi. 
```
from aiida import load_profile
from aiida.orm import Code, load_node, load_code, load_group
from aiida.orm.nodes.data.upf import get_pseudos_from_structure
from aiida.plugins import WorkflowFactory
from aiida.engine import submit
from aiida.plugins import DataFactory
from ase.build import bulk
from ase.io.espresso import kspacing_to_grid

 

# Initiate the default profile

load_profile()


# Import relax work chain from WorkflowFactory

PwRelaxWorkChain = WorkflowFactory('quantumespresso.pw.relax')

# Group label
group_name = 

try:
    group = load_group(group_name)
except: 
    group = Group(label=group_name)
    group.store()


# Overrides


overrides = {
        "base": {'pseudo_family': 'SSSP/1.3/PBEsol/precision'},
        'base_final_scf': {'pseudo_family': 'SSSP/1.3/PBEsol/precision'}
        }

# Get structure for calculation

StructureData = DataFactory('core.structure')

ase_structure = bulk('Si',crystalstructure='diamond',a=5.43)*(2,2,2)

structure = StructureData(ase=ase_structure)


#Load pseudo family 

# Create the builder

builder = PwRelaxWorkChain.get_builder_from_protocol(code=load_code('pw_environ@roar_basic'),
            structure=structure, overrides=overrides)


# Alter any input

builder.base['pw']['metadata']['options']['max_wallclock_seconds'] = 1800

builder.base_final_scf['pw']['metadata']['options']['max_wallclock_seconds'] = 1800

builder.base['pw']['metadata']['options']['queue_name'] = 'open'

builder.base_final_scf['pw']['metadata']['options']['queue_name'] = 'open'

builder.base['pw']['metadata']['options']['account'] = 'open'

builder.base_final_scf['pw']['metadata']['options']['account'] = 'open'

 

# Submit calculation

calc = submit(builder)

group.add_nodes(calc)
 
#individual numbers associated with a calculation
print(f'Created calculation with PK={calc.pk}')
````
To check the status of the process, use below. 
```
verdi process show 190 
verdi shell
```
This is just the beginning. There are tutorials for Quantum Espresso's AiiDA workflows [here](https://aiida-quantumespresso.readthedocs.io/en/latest/tutorials/index.html) and other available worklows on their github [here](https://github.com/aiidateam/aiida-quantumespresso/tree/b79189d7ce4756e846ab39c567ba4681474741ed/docs/source/topics/workflows). Good luck on your journey for promoting data provenance and better time management! 
