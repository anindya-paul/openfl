# PyTorch TinyImageNet

## **Habana Tutorials**
#### The name of the file/example that contain HPU adaptations start with "HPU". 
For example: HPU_pytorch_tinyimagenet.ipynb placed under workspace folder contains the required HPU adaptations.

 All the execution steps mention in last section (**V. How to run this tutorial**) remain same for HPU examples but as pre-requisite it needs some additional environment setup and Habana supported package installations which is explained below from **section I to IV**.

 <br/>

 ## **I. AWS DL1 Instance Setup**

 This example was tested on AWS EC2 instance created by following the instructions mentioned [here](https://docs.habana.ai/en/latest/AWS_EC2_DL1_and_PyTorch_Quick_Start/AWS_EC2_DL1_and_PyTorch_Quick_Start.html) . 
 
 Test setup - Habana 1.7 and Ubuntu 20.04

 <br/>

 ## **II. Set Up SynapseAI SW Stack**

 - To perform an installation of the full driver and SynapseAI software stack independently on the EC2 instance, run the following command:
 
 ```
 wget -nv https://vault.habana.ai/artifactory/gaudi-installer/latest/habanalabs-installer.sh
chmod +x habanalabs-installer.sh
./habanalabs-installer.sh install --type base
```
You can refer the [Habana docs](https://docs.habana.ai/en/latest/Installation_Guide/Bare_Metal_Fresh_OS.html#set-up-synapseai-sw-stack) mentioned [GitHub repository](https://github.com/HabanaAI/Setup_and_Install) for detailed instructions.

<br/>

 ## **III. HPU Pytorch Installation**

 For this example make sure to install the PyTorch package provided by Habana. These packages are optimized for Habana Gaudi HPU. Installing public PyTorch packages is not supported.
 Habana PyTorch packages consist of:
 - **torch** - PyTorch framework package with Habana support

- **habana-torch-plugin** - Libraries and modules needed to execute PyTorch on single card, single node and multi node setup.

- **habana-torch-dataloader** - Habana multi-threaded dataloader package.

- **torchvision** - Torchvision package compiled in torch environment. No Habana specific changes in this package. 
 
 Run the following command to install the above Habana PyTorch environment

 ```
 ./habanalabs-installer.sh install --type pytorch --venv
 ``` 

 The -- venv flag installs the relevant framework inside the virtual environment. To activate a virtual environment please perform the following:
 
 
 ```
  cd $HOME/habanalabs-venv
  source ./bin/activate
 ```

The default virtual environment folder is $HOME/habanalabs-venv. To override the default, run the following command:

 ```
 export HABANALABS_VIRTUAL_DIR=/path/to/dir
 ```

 You can refer the [Habana docs](https://docs.habana.ai/en/latest/Installation_Guide/Bare_Metal_Fresh_OS.html#set-up-synapseai-sw-stack) mentioned [GitHub repository](https://github.com/HabanaAI/Setup_and_Install) for detailed instructions.

 </br>

 ## **IV. HPU Adaptations For PyTorch Examples**

The following set of code additions are required in the workspace notebook to run a model on Habana. The following steps cover Eager and Lazy modes of execution.

### 1. Load the Habana PyTorch Plugin Library, libhabana_pytorch_plugin.so.

```
import torch
from habana_frameworks.torch.utils.library_loader import load_habana_module
load_habana_module()
```

### 2. Target the Gaudi HPU device:

```
device = torch.device("hpu")
```
### 3. Move the model to the device:

**There is a dependency in the order of execution (moving model to HPU and intializing optimizer). The workaround is to execute this step before initializing any optimizers.**

```
model.to(device)
```
### 4. Import the Habana Torch Library:

```
import habana_frameworks.torch.core as htcore
```
### 5. Enable Lazy execution mode by setting the environment variable shown below. 
Do not set this environment variable if you want to execute your code in Eager mode:

```
os.environ["PT_HPU_LAZY_MODE"] = "1"
```
### 6. In Lazy mode, execution is triggered wherever data is read back to the host from the Habana device. 
For example, execution is triggered if you are running a topology and getting loss value into the host from the device with loss.item(). Adding a mark_step() in the code is another way to trigger execution:

```
htcore.mark_step()
```

The placement of mark_step() is required at the following points in a training script:

* Right after optimizer.step() to cleanly demarcate training iterations,
* Between loss.backward and optimizer.step() if the optimizer being used is a Habana custom optimizer.

Refer [getting started with PyTorch](https://www.intel.com/content/www/us/en/developer/articles/technical/get-started-habana-gaudi-deep-learning-training.html#articleparagraph_cop_711677074) for detailed explaination and PyTorch Habana architecture. Sample example can be found [here](https://docs.habana.ai/en/latest/PyTorch/Getting_Started_with_PyTorch_and_Gaudi/Getting_Started_with_PyTorch.html)

<br/>


## **V. How to run this tutorial (without TLS and locally as a simulation):**
<br/>

### 0. If you haven't done so already, install OpenFL in the virtual environment created during Habana setup, and upgrade pip:
  - For help with this step, visit the "Install the Package" section of the [OpenFL installation instructions](https://openfl.readthedocs.io/en/latest/install.html#install-the-package).

<br/>
 
### 1. Split terminal into 3 (1 terminal for the director, 1 for the envoy, and 1 for the experiment)

<br/> 

### 2. Do the following in each terminal:
   - Activate the virtual environment same as in section **III**:

   - If you are in a network environment with a proxy, ensure proxy environment variables are set in each of your terminals.
   - Navigate to the tutorial:
    
   ```sh
   cd openfl/openfl-tutorials/interactive_api/HPU/PyTorch_TinyImageNet
   ```

<br/>

### 3. In the first terminal, run the director:

```sh
cd director
./start_director.sh
```

<br/>

### 4. In the second terminal, install requirements and run the envoy:

```sh
cd envoy
pip install -r requirements.txt
./start_envoy.sh env_one envoy_config_1.yaml
```

Optional: Run a second envoy in an additional terminal:
  - Ensure step 2 is complete for this terminal as well.
  - Run the second envoy:
```sh
cd envoy
./start_envoy.sh env_two envoy_config_2.yaml
```

<br/>

### 5. Now that your director and envoy terminals are set up, run the Jupyter Notebook in your experiment terminal:

```sh
cd workspace
jupyter lab pytorch_tinyimagenet.ipynb
```
- A Jupyter Server URL will appear in your terminal. In your browser, proceed to that link. Once the webpage loads, click on the pytorch_tinyimagenet.ipynb file. 
- To run the experiment, select the icon that looks like two triangles to "Restart Kernel and Run All Cells". 
- You will notice activity in your terminals as the experiment runs, and when the experiment is finished the director terminal will display a message that the experiment has finished successfully.  

<br/>

