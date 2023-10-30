# gcp-slurm-configs

The files in the following are:

- `slurm.yaml`: This defines the slurm cluster on GCP.
  - It was cobbled together from:
    - https://github.com/GoogleCloudPlatform/hpc-toolkit/tree/main/examples#hpc-slurm-ubuntu2004yaml-
    - https://github.com/GoogleCloudPlatform/hpc-toolkit/tree/main/examples#ml-slurmyaml-


## Deploying Google HPC
```
# Under ~/hpc-toolkit
./ghpc create examples/hpc-slurm-custom.yaml \
    -l ERROR --vars project_id=hacman -w
# First-time deployment
./ghpc deploy slurm-gcp-v5
# Update
terraform -chdir=slurm-gcp-v5/primary init
terraform -chdir=slurm-gcp-v5/primary plan      # check the changes
terraform -chdir=slurm-gcp-v5/primary apply
```
## Modifying the cuda
```
wget https://developer.download.nvidia.com/compute/cuda/11.8.0/local_installers/cuda_11.8.0_520.61.05_linux.run
# root
sudo sh cuda_11.8.0_520.61.05_linux.run
# cuda
export PATH=/usr/local/cuda-11.8/bin:$PATH
export LD_LIBRARY_PATH=/usr/local/cuda-11.8/lib64:$LD_LIBRARY_PATH
# non-root
sh cuda_11.8.0_520.61.05_linux.run --silent --override --toolkit --toolkitpath=~/cuda-toolkit-11.8
# cuda
export PATH=/home/bowenj_andrew_cmu_edu/cuda-toolkit-11.8/bin:$PATH
export LD_LIBRARY_PATH=/home/bowenj_andrew_cmu_edu/cuda-toolkit-11.8/lib64:$LD_LIBRARY_PATH
```
## Adding miniconda
```
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
~/miniconda3/bin/conda init bash
```
## Adding mujoco
```
mkdir -p ~/.mujoco
wget https://github.com/google-deepmind/mujoco/releases/download/2.1.0/mujoco210-linux-x86_64.tar.gz -O ~/mujoco.tar.gz
tar -xf ~/mujoco.tar.gz -C ~/.mujoco
rm ~/mujoco.tar.gz
# mujoco
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/home/bowenj/.mujoco/mujoco210/bin
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/lib/nvidia
```
## Mounting the NFS
```
# https://github.com/GoogleCloudPlatform/hpc-toolkit/blob/main/modules/file-system/filestore/scripts/install-nfs-client.sh
sudo apt -y install nfs-common
wget https://raw.githubusercontent.com/GoogleCloudPlatform/hpc-toolkit/main/modules/file-system/filestore/scripts/mount.sh -O ~/mount.sh
chmod +x ~/mount.sh
sudo ~/mount.sh 10.83.183.170 /nfsshare/bowenj_andrew_cmu_edu /home/bowenj_andrew_cmu_edu nfs
sudo ~/mount.sh 10.83.183.170 /nfsshare /home nfs
sudo ~/mount.sh 10.83.183.170 /nfsshare /mnt/filestore nfs
```
## Test the cluster
```
# ssh into the login node
srun -N 1 --gpus 1 nvidia-smi
srun -N 1 -p a2 --gpus 1 nvidia-smi
srun --gpus 1 --pty /bin/bash
```
## Destroying the cluster
```
terraform -chdir=slurm-gcp-v5/primary destroy -auto-approve
```
# Other operations
## Copying files to the cluster
```
gcloud compute scp --recurse ~/Projects/hacman_cleanup slurmgcpv5-login-jqutpgkq-001:/home/bowenj_andrew_cmu_edu/Projects
```