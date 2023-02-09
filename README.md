# Generative Adversarial Neuroevolution for Control Behaviour Imitation

### Information

This repository contains the instructions on how to reproduce the experiments from the paper "Generative Adversarial Neuroevolution for Control Behaviour Imitation".

The code itself is located in a larger library called [Neuroevo](https://github.com/authorsgecco2023/neuroevo).

### Installation (tested on Ubuntu 20.04)

```
# Debian packages                       ----------mpi4py---------- ~~~~~~~~~~~~~~~~~Gym~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
sudo apt install git python3-virtualenv python3-dev libopenmpi-dev g++ swig libosmesa6-dev patchelf ffmpeg libglew-dev

# MuJoCo
wget https://mujoco.org/download/mujoco210-linux-x86_64.tar.gz -P ~/Downloads/
mkdir -p ~/.mujoco/ && tar -zxf ~/Downloads/mujoco210-linux-x86_64.tar.gz -C ~/.mujoco/
echo -e "\n# MuJoCo\nMUJOCO_PATH=~/.mujoco/mujoco210/bin
export LD_LIBRARY_PATH=\$MUJOCO_PATH:\$LD_LIBRARY_PATH" >> ~/.bashrc
source ~/.bashrc

# Library & Dependencies
git clone https://github.com/authorsgecco2023/neuroevo && cd neuroevo/
virtualenv venv && source venv/bin/activate && pip3 install -r requirements.txt

# Pre-trained agents for imitation
git clone --recursive https://github.com/DLR-RM/rl-baselines3-zoo ~/rl-baselines3-zoo
```

### Execution

`<task>` = `acrobot`, `cart_pole`, `mountain_car`, `mountain_car_continuous`, `pendulum`, `lunar_lander`, `lunar_lander_continuous` or `swimmer`

```
mpiexec -n <n> python3 main.py --env_path envs/multistep/imitate/control.py \
                               --bots_path bots/network/static/rnn/control.py \
                               --nb_generations <nb_generations> \
                               --population_size <population_size> \
                               --additional_arguments '{"task" : "<task>"}' \
                               --data_path ~/rl-baselines3-zoo/
```

Example from the paper: (you can increase the number of MPI processes if your machine allows it)
```
mpiexec -n 2 python3 main.py --env_path envs/multistep/imitate/control.py \
                             --bots_path bots/network/static/rnn/control.py \
                             --nb_generations 200 \
                             --population_size 64 \
                             --additional_arguments '{"task" : "mountain_car_continuous"}' \
                             --data_path ~/rl-baselines3-zoo/
```

### Downloading the Paper's Results & Final Dynamic States

```
wget https://www.dropbox.com/s/xw1gn834vbc40jr/envs.multistep.imitate.control.zip -P ~/Downloads/
unzip -o ~/Downloads/envs.multistep.imitate.control.zip -d data/states/
```

You can now run additional generations ...
```
mpiexec -n 2 python3 main.py --env_path envs/multistep/imitate/control.py \
                             --bots_path bots/network/static/rnn/control.py \
                             --nb_elapsed_generations 300 \
                             --nb_generations 10 \
                             --population_size 64 \
                             --additional_arguments '{"task" : "cart_pole"}' \
                             --data_path ~/rl-baselines3-zoo/
```

... Evaluate the new agents ...
```
mpiexec -n 4 python3 utils/evaluate.py --states_path data/states/envs.multistep.imitate.control/merge.no~steps.0~task.cart_pole~transfer.no/bots.network.static.rnn.control/64/
```

... And record the elite's behaviour.
```
python3 utils/record.py --state_path data/states/envs.multistep.imitate.control/merge.no~steps.0~task.cart_pole~transfer.no/bots.network.static.rnn.control/64/310/
```

### Reproducing the Paper's Figures

```
# Verify Stable Baselines 3 Results
jupyter notebook utils/notebooks/generative-adversarial-neuroevolution/sb3_baselines.ipynb

# Reproduce the paper's figures  
jupyter notebook utils/notebooks/generative-adversarial-neuroevolution/figures.ipynb
```
