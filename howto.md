# The Easy Way - just install WSL
wsl --install

# The Hard Way - Install WSL (on a drive of your choice)
## https://medium.com/@sharansh.sinha/wsl-2-installation-on-different-drive-3d9f0cc88850

Enable-WindowsOptionalFeature -Online -FeatureName Microsoft-Windows-Subsystem-Linux

### Restart
dism.exe /online /enable-feature /featurename:VirtualMachinePlatform /all /norestart
dism.exe /online /enable-feature /featurename:Microsoft-Windows-Subsystem-Linux /all /norestart
wsl --version 
wsl --install
wsl --set-default-version 2

### Grab a distrubtion of Ubuntu
https://learn.microsoft.com/en-us/windows/wsl/install-manual#downloading-distributions
dowload the verison you want, change the extension to .zip
Unzip it
look in the contents for another file ending in 'x64.appx' copy that file to whatever drive youw want WSL to run from
change the extension from .appx to .zip
Unzip the file
locate the Ubuntu.exe file and execute it

C:\WINDOWS\system32\wsl.exe -d Ubuntu

# Setup GPT on WSL with CUDA support
# https://medium.com/@docteur_rs/installing-privategpt-on-wsl-with-gpu-support-5798d763aa31

## Update Ubuntu
sudo apt-get update
sudo apt-get upgrade
sudo apt-get install build-essential

## Clone privateGPT repo
git clone https://github.com/imartinez/privateGPT

## Intall Pyenv
sudo apt-get install git gcc make openssl libssl-dev libbz2-dev libreadline-dev libsqlite3-dev zlib1g-dev libncursesw5-dev libgdbm-dev libc6-dev zlib1g-dev libsqlite3-dev tk-dev libssl-dev openssl libffi-dev
curl https://pyenv.run | bash
export PATH="/home/$(whoami)/.pyenv/bin:$PATH"

## Modify your .bashrc file to contain this content (vi ~/.bashrc)

'# Added for privateGPT
export PYENV_ROOT="$HOME/.pyenv"
[[ -d $PYENV_ROOT/bin ]] && export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"

# Change YOUR_USERNAME
export PATH="/home/<YOUR_USERNAME>/.local/bin:$PATH"

# Added for Cuda
export PATH="/usr/local/cuda-12.6/bin:$PATH"
export LD_LIBRARY_PATH="/usr/local/cuda-12.6/lib64:$LD_LIBRARY_PATH"
'

## Reload Bash
source ~/.bashrc

## Install more Pyenv Stuff
sudo apt-get install lzma
sudo apt-get install liblzma-dev

## Install Python3 and set the default verion
pyenv install 3.11
pyenv global 3.11
pip install pip --upgrade
pyenv local 3.11

## Install Poetry to help manage dependancies
curl -sSL https://install.python-poetry.org | python3 -

### Check that it works (if not make sure you edited your .bashrc correctly)
poetry --version 

## Install the GPT
cd privateGPT
poetry install --extras "ui embeddings-huggingface llms-llama-cpp vector-stores-qdrant"

## Grab the Cuda intaller from the Nvidia Website

https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0&target_type=deb_network

## Follow the steps on the Nvida site 
wget https://developer.download.nvidia.com/compute/cuda/repos/wsl-ubuntu/x86_64/cuda-keyring_1.1-1_all.deb
sudo dpkg -i cuda-keyring_1.1-1_all.deb
sudo apt-get update
sudo apt-get -y install cuda-toolkit-12-6

## Run this to make sure your system sees your GPU properly
nvcc --version
nvidia-smi.exe

## Install the stuff
### The old way CMAKE_ARGS='-DLLAMA_CUBLAS=on' poetry run pip install --force-reinstall --no-cache-dir llama-cpp-python
CMAKE_ARGS='-DGGML_CUDA=on' poetry run pip install --force-reinstall --no-cache-dir llama-cpp-python

## Run it
poetry run python scripts/setup
make run


