# How to use Singularity on Midway3


Docker is not supported by RCC on the Midway cluster because of security concerns that Singularity addresses. Users who wish to use Docker containers should convert them to Singularity containers. Instructions on how to do this are included below.

Usage `singularity [OPTIONS] run CONTAINERNAME`

### **Basic Instructions**

1. Log into your home directory
2. Create a folder in your home or scratch directory to store the container. For bigger image scratch directory would be advised.
    1. **`mkdir containername`** # Substitute containername with the name of the image
3. Enter that folder (`cd containername`)
4. Then run **singularity pull containername.sif docker://repo/containername:tags**
5. Example: **`singularity pull ubuntu.sif docker://ubuntu:latest`**
6. It pulls a docker container called "ubuntu" from the "default" repository and looks for the "latest" version. The docker container is then converted to a Singularity container.
7. From here, you should be able to run the container by running **`singularity run containername.sif`** where containername.sif is the name of the container created from the "singularity pull ..." command above**.**

### **Instructions for getting docker from private repository:**

Create a Singularity image from a Docker image that is in the NGC container registry requires an authentication or API key. Normally these can be provided by --docker-login command. To create the API key, use this user guide: [NGC Container User Guide for NGC Catalog: NVIDIA GPU Cloud Documentation](https://docs.nvidia.com/ngc/ngc-catalog-user-guide/index.html)

From the login node

```bash
module load singularity
singularity pull --docker-login dli-nlp-nemo.sif docker://nvcr.io/nvidia/dli/dli-nlp-nemo:v3-nemo1.0.1
```

prompted to enter your API Key

The **dli-nlp-nemo** file is located at **/project/rcc/pnsinha/dli-nlp-nemo.sif**

```bash
sinteractive  --account=rcc-staff --partition=gpu  --gres=gpu:1 --mem=16gb
module load singularity
singularity run dli-nlp-nemo.sif
```
As the singularity image is read only, it is desirable to make directories available inside the container

```bash
singularity exec --bind /path/outside/image/:/path/inside/image/ --bind $PWD:/run/user dli-nlp-nemo.sif
```
where 
--bind $PWD:/run/user is setting up the working directory to be accessible inside container.

![](images/run.png)

**Opening the Jupyter Lab**

**Option 1**: You can use Thinlinc and open Firefox and navigate to url  `http://<compute-node>:8888/lab/lab`

where <compute-node> is the name of node you have your interactive session is connected. 
![](images/jlab.png)
**Option 2**: Or you can create an ssh tunnel and forward all the traffic of jupyter notebook port to your local machine

`ssh -N -f -L 8888:<compute-node>:8888 cnetID@midway3.rcc.uchicago.edu`

open any browser and navigate to http://127.0.0.1:8888 or http://localhost:8888


### Creating sbatch file for singularity jupyter lab

```bash
#!/bin/bash
#SBATCH --job-name=jupyter_notebook
#SBATCH --time=05:00:00
#SBATCH --output=jupyter_notebook_%j.txt
#SBATCH --error=jupyter_notebook_%j.err
#SBATCH --account=pi-<group>
#SBATCH --partition=gpu  
#SBATCH --gres=gpu:1
#SBATCH --mem=16gb

#assign random port between 8000 and 9000
PORT_NUM=$(shuf -i8000-9000 -n1)
node=$(hostname -s)
user=$(whoami)
cluster="midway3"

#As the singularity image is read only, it is desirable to make directories available inside the container
#/path/outside/image/ is path in your midway3 dir
#/path/inside/image/ is path inside the container

# print tunneling instructions jupyter-log
echo -e "
# Note:# Check jupyter_notebook_%j.err to find the port.

# Command to create SSH tunnel:
ssh -N -f -L $PORT_NUM:${node}:$PORT_NUM ${user}@${cluster}.rcc.uchicago.edu

# Use a browser on your local machine to go to:
http://localhost:$PORT_NUM/
"
module load singularity
singularity exec --bind /path/outside/image/:/path/inside/image/ --bind $PWD:/run/user dli-nlp-nemo.sif jupyter lab --no-browser --ip=${node} --port=$PORT_NUM

# keep it alive
sleep 36000
```

### Troubleshooting:

As a default, Singularity uses the **/tmp** directory on the local machine (whichever node or login node you happen to be working on) to dump temporary and cache files generated during the build process. After a container is built, these are usually deleted. For containers with a large total size (more than a few GB), you may encounter an error. write /tmp/... : no space left on device

If you see this, then it most likely means that **/tmp** got filled up on the machine you're using. to get around create **faketmp** and **fakecache** directory and redirect Singularity's default temporary output to **SINGULARITY_TMPDIR** and **SINGULARITY_CACHEDIR**

```bash
export $SINGULARITY_CACHEDIR=$SCRATCH/$user/container/fakecache
export $SINGULARITY_TMPDIR=$SCRATCH/$user/container/faketmp
```

You can delete everything in the "faketmp" and "fakecache" directories after creation of your container.


**References:** 
* https://catalog.ngc.nvidia.com/orgs/nvidia/teams/dli/containers/dli-nlp-nemo
* https://www.hpcwire.com/2017/05/04/singularity-hpc-container-technology-moves-lab/
* https://github.com/apptainer/singularity
* https://apptainer.org/

For any additional question please email help@rcc.uchicago.edu
