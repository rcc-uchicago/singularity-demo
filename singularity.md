---
marp: true
---



---


# Using Singularity/Apptainer on Midway Cluster


* **Parmanand Sinha, Computational Scientist**
* May 30, 2023

---


# Where to get this presentation

---
### Where to get this presentation

1.  Web browser:

        https://github.com/rcc-uchicago/singularity-demo

2.  GIT:

        git clone https://github.com/rcc-uchicago/singularity-demo.git
---

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

Imagine you have a bunch of different things you want to keep separate, like toys or food items. To do that, you might use different containers, like boxes or jars, to hold each item. Containers in the world of technology are quite similar. They are like virtual boxes that hold all the necessary things for an application or program to run, such as the code, libraries, and settings it needs.

Containers are lightweight and portable, meaning they can be easily moved from one place to another. They provide a consistent environment for applications, regardless of the underlying computer system. It's like having a portable "package" that includes everything an application needs to run correctly. Containers make it easier for developers to build, deploy, and manage applications, as they don't have to worry about differences between computer systems or software dependencies.

1. Containers are like virtual boxes that hold all the necessary things for an application to run, such as code, libraries, and settings.
2. Containers are lightweight and portable, making it easy to move them from one place to another.
3. Containers provide a consistent environment for applications, regardless of the underlying computer system.

---

---

## Singularity
4. Singularity is a specialized type of container technology for scientific and high-performance computing.
5. Singularity allows researchers to package their scientific workflows, experiments, or simulations into containers that can be easily shared and run on different computing systems.
6. Singularity ensures consistency of scientific software and dependencies across different computing environments.
7. Containers and Singularity make it easier for developers and researchers to build, deploy, manage, collaborate, and reproduce their work effectively.
---



---
### workflow

Let's say you're a researcher working on a complex simulation that requires specific software and libraries to run. You've developed the simulation on your personal computer, but now you want to run it on a high-performance computing cluster at your university. However, the cluster has different software versions and configurations than your computer.

To ensure your simulation runs smoothly on the cluster, you can create a Singularity container.

-   You define a Singularity recipe file that specifies the necessary software dependencies, libraries, and configurations required by your simulation.

-   Using the recipe file, you build the Singularity container on your personal computer. This process compiles the necessary components and packages them into a self-contained container.

-   Once the container is built, you transfer it to the high-performance computing cluster.

-  On the cluster, you load the Singularity software and execute your simulation within the container. Singularity provides an isolated environment where your simulation can run with all the required dependencies, regardless of the specific software versions on the cluster.

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

---
### What is Singularity

-   Singularity vs Docker

    -   Singularity was designed for HPC by the same guy, Gregory
        Kurtzer from LBNL, who also started CentOS Linux distribution

    -   Docker was designed for enterprise, to run trusted applications
        by trusted users, not suitable for HPC where 1000s users are
        doing crazy things on the same system

    -   Docker requires running a daemon, Singularity does not

    -   Docker allows a user to become root and therefore is not secure
        to run on the system with multiple untrusted users

    -   Singularity does not escalate priviliges: inside container you
        are still the same user as outside, with the same privileges.

-   Singularity can use Docker containers either directly or by
    converting them into Singularity images


---

# When to use Singularity

---
### When to use Singularity


---

---
### When to use Singularity

-   To create a Singularity recipe for installing Python, you can follow these steps:
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

%post is a section where you can execute commands inside the container during the build process. Here, we update the package repository and install python3-dev and python3-pip using apt-get.

%environment sets environment variables inside the container. Here, we add /usr/local/bin to the PATH environment variable to ensure Python is accessible.

%runscript specifies the command to execute when running the Singularity container. In this case, it runs Python with the arguments provided.

Save the file.
Now you have a basic Singularity recipe that installs Python inside the container. You can build the Singularity container using this recipe by running the following command:


```bash
sudo singularity build python_container.sif python_recipe.def
```
This will create a Singularity container file named python_container.sif based on the recipe specified in python_recipe.def. You can then use this container to run Python applications or scripts on different systems without worrying about installing Python or its dependencies.

---
### singularity image from NVIDIA GPU Cloud

Go to the NVIDIA GPU Cloud (NGC) website at https://ngc.nvidia.com/.

Create an account or log in to your existing account.

Once you're logged in, navigate to the "Catalog" section.

In the catalog, you'll find various containers and frameworks optimized for NVIDIA GPUs. Select the Singularity image you wish to download based on your requirements and click on it to access its details.

On the container details page, you will find information about the container, including the commands needed to pull it. Look for the section labeled "Pull Command" or "Singularity Pull Command."

Copy the entire pull command. It will resemble something like:

```bash
singularity pull --arch <architecture> <container-name:tag>
```


The <architecture> placeholder refers to the target architecture for which you want to download the Singularity image, such as amd64 for x86-64 processors.

Open a terminal on your local machine or the system where you want to download the Singularity image.

Paste the copied pull command into the terminal and execute it. This command will initiate the download of the Singularity image from the NGC registry to your local machine or the specified system.

Depending on your internet connection and the size of the Singularity image, the download may take some time. Once the download is complete, you will have the Singularity image file on your local machine or the specified system.

-   
---

# Example codes

## 
https://github.com/rcc-uchicago/singularity-demo/blob/main/How%20to%20use%20Singularity%20on%20Midway3.md

https://github.com/rcc-uchicago/singularity-demo/blob/main/Rstudio_Singularity.md

---

---



# References

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
---
