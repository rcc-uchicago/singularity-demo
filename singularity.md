---
marp: true
---






# Using Singularity/Apptainer on Midway Cluster

* **Parmanand Sinha, Computational Scientist**
* May 30, 2023

---

## Where to get this presentation

1. Web browser:

        https://github.com/rcc-uchicago/singularity-demo

2. GIT:

        git clone https://github.com/rcc-uchicago/singularity-demo.git

---

### Learning Objectives

* Singularity syntax, the use of container repositories
* Techniques for working with container images
* Some common containerization tools
* How to find and run containers built by other people
* How to build their own container
* How to distribute their container online

URL: <https://github.com/rcc-uchicago/singularity-demo>
---

---

## Containers and Singularity

Containers, like boxes for toys, hold everything an application needs to run: code, libraries, and settings. They offer a consistent environment and are easily portable, simplifying application development and management.

1. Containers are like virtual boxes that hold all the necessary things for an application to run, such as code, libraries, and settings.
2. Containers are lightweight and portable, making it easy to move them from one place to another.
3. Containers provide a consistent environment for applications, regardless of the underlying computer system.

---



## Singularity

4. Singularity is a specialized type of container technology for scientific and high-performance computing.
5. Singularity allows researchers to package their scientific workflows, experiments, or simulations into containers that can be easily shared and run on different computing systems.
6. Singularity ensures consistency of scientific software and dependencies across different computing environments.
7. Containers and Singularity make it easier for developers and researchers to build, deploy, manage, collaborate, and reproduce their work effectively.

---



### Workflow

- You are a researcher working on a complex simulation requiring specific software and libraries.
- You developed the simulation on your personal computer.
- You want to run the simulation on a high-performance computing cluster at your university.
- The cluster has different software versions and configurations than your computer.
- On the cluster, you load the Singularity software.
- You execute your simulation within the Singularity container.
- Singularity provides an isolated environment for your simulation.
- This environment allows your simulation to run with all required dependencies, regardless of the specific software versions on the cluster.

---

## Containers vs VM

    -   Container shares kernel with the host, VM does not

    -   As a result, container has direct access to the same hardware
        devices as the host: therefore no problem using GPU, infiniband,
        GPFS, etc.

    -   VM abstracts the hardware, introduces overhead, makes some
        hardware unavailable inside VM

    -   Container is much more lightweight than VM, almost native
        performance

    -   Container can only run Linux on a Linux host but the
        distributions/versions, software environment can be different.

    -   VM can run different OS than host, the host and VM do not have
        to be Linux.

    -   VM typically gives more flexibility at the expense of
        performance.
---


### Singularity vs Docker

* Singularity was designed for HPC by the same guy, Gregory Kurtzer from LBNL, who also started CentOS Linux distribution

* Docker was designed for enterprise, to run trusted applications by trusted users, not suitable for HPC where 1000s users are doing crazy things on the same system

* Docker requires running a daemon, Singularity does not

* Docker allows a user to become root and therefore is not secure to run on the system with multiple untrusted users

* Singularity does not escalate priviliges: inside container you are still the same user as outside, with the same privileges.

* Singularity can use Docker containers either directly or by converting them into Singularity images

---

# When to use Singularity

---

### When to use Singularity

To be done on User's system

- To create a Singularity recipe for installing Python, follow these steps:
Create a new text file and give it a suitable name, such as python_recipe.def.

Open the file in a text editor and add the following lines:

```bash
Bootstrap: docker
From: python:latest

%post
    apt-get update
    apt-get install -y python3-dev python3-pip

%environment
    export PATH="/usr/local/bin:$PATH"

%runscript
    python3 "$@"
```

---
Let's go through what each section does:

Bootstrap: docker indicates that we'll be using a Docker image as the base for our Singularity container. In this case, we're using the latest Python Docker image.

From: python:latest specifies the base image to use, which includes Python.

%post is a section where commands are executed inside the container during the build process. Here, we update the package repository and install python3-dev and python3-pip using apt-get.

%environment sets environment variables inside the container. Here, we add /usr/local/bin to the PATH environment variable to ensure Python is accessible.

%runscript specifies the command to execute when running the Singularity container. In this case, it runs Python with the provided arguments.

Save the file.

---
Now we have a basic Singularity recipe that installs Python inside the container. We can build the Singularity container using this recipe by running the following command:

```bash
sudo singularity build python_container.sif python_recipe.def
```

Executing the command initiates the creation of a Singularity container, named `python_container.sif`. This action crafts the container in accordance with the specifications outlined in `python_recipe.def`. 

---
## Singularity image from NVIDIA GPU Cloud

Go to the NVIDIA GPU Cloud (NGC) website at <https://ngc.nvidia.com/>.

Create an account or log in to your existing account.

Once you're logged in, navigate to the "Catalog" section.

In the catalog, we'll find various containers and frameworks optimized for NVIDIA GPUs. Select the Singularity image we wish to download based on our requirements, and click on it to access its details.

On the container details page, we will find information about the container, including the commands needed to pull it. Look for the section labeled "Pull Command" or "Singularity Pull Command."

---
Copy the entire pull command. It will resemble something like:

```bash
singularity pull --arch <architecture> <container-name:tag>
```

The placeholder <architecture> denotes the architecture required to download the Singularity image, like amd64 for x86-64 processors.

To initiate the download, launch a terminal on the target device or system.

Paste and execute the provided pull command in the terminal. This action will commence the retrieval of the Singularity image from the NGC registry to the designated device or system.

The download duration is contingent on internet speed and image size. Upon completion, the Singularity image file will be available on the device or specified system.

---

## Exercises 

1 Download a pre-built image for ubuntu and save into a file

```bash
mkdir /scratch/midway3/$USER/container

cd /scratch/midway3/$USER/container

#Pull the Ubuntu image from Singularity repository
singularity pull ubuntu.sif library://ubuntu:20.04

#Pull the same image from the Docker hub and convert to a Singularity image
singularity pull ubuntu_d.sif docker://ubuntu:20.04

#check the sizes of the two image files you just downloaded
ls -l *.sif
```

---
2 Explore the container
```bash
#Check the operating system of the host, you should see CentOS
head /etc/os-release

#Start the container, and enter the shell
singularity shell ./ubuntu.sif

#Now check the operating system of the container, you would see Ubuntu
head /etc/os-release

#List the root directory of the container, can you tell which are from the host? which are from the container?
ls -l /

#List the home directory in the Container, you should see a single user which is you.
ls -l /home

#your user id inside container
whoami

#exit the container
exit

```

---


3 Common options

The "--no-home" option instructs Singularity not to mount the home directory. This is useful for isolating the environment completely and avoiding the utilization of Python and R libraries installed in the home directory. It is recommended to compare the outcomes when using and not using the "--no-home" option.
```bash
singularity exec ./ubuntu.sif ls /home/$USER

singularity exec --no-home ./ubuntu.sif ls /home/$USER
```

---
The --bind/-B option allows you to mount a host directory inside the container. For instance, using this example, the host directory /workdir/$USER will be mounted as the /data directory within the container. If no ":" is used, for example, "--bind /workdir/$USER", the directory will be mounted with the same path inside the container.
```bash
singularity exec --bind /scratch ubuntu.sif ls /scratch

#to mount both scratch and project
singularity exec -B /scratch -B /project ubuntu.sif ls /project/
```

---
Modify an existing singularity image file
Fakeroot privilege is necessary for creating a Singularity image.

```bash
singularity build --fakeroot --sandbox myubuntu ./ubuntu.sif

#start the shell and install python
singularity shell --fakeroot --writable myubuntu
apt update
apt upgrade
apt install python3.9
ln -s /usr/bin/python3.9 /usr/bin/python
which python

python -V

exit
```
---

Convert the sandbox into a Singularity image file, and test the image file
```bash

singularity build --fakeroot myubuntu.sif myubuntu/

singularity exec myubuntu.sif python -V
```

Use the example definition file we covered previously to create a container using --fakeroot.

```bash
ingularity build --fakeroot python_container.sif python_recipe.def
```

---

## Example codes to run on Midway


<https://github.com/rcc-uchicago/singularity-demo/blob/main/How%20to%20use%20Singularity%20on%20Midway3.md>

<https://github.com/rcc-uchicago/singularity-demo/blob/main/Rstudio_Singularity.md>

---


### References

    http://singularity.lbl.gov   - Singularity home page

    https://singularity-hub.org/ - Singularity Hub

    https://hub.docker.com/      - Docker Hub

    https://github.com/singularityware/\
       singularity/tree/master/examples  - Examples of recipes

    https://www.virtualbox.org   - VirtualBox home page

    https://github.com/singularityhub/\
          singularityhub.github.io/wiki
                                 - Build containers in the Hub
