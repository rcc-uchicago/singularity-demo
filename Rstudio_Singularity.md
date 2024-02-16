
# Using Rstudio with Singulrity

Two Docker images we utilize are created by the Rocker Project. You can find more information about them at the following links: https://rocker-project.org/ and https://journal.r-project.org/archive/2020/RJ-2020-007/RJ-2020-007.pdf.

## Part1: Create a Rstudio server container
For the rocker/geospatial, which is based on rocker/rstudio and includes most of the packages, you can run it directly from the Docker image like this (use either of these images):

```shell
singularity pull rstudio-geo_latest.sif docker://rocker/geospatial:latest
```
And for the base R version available on https://hub.docker.com/r/rocker/rstudio:
```shell
singularity pull rstudio_latest.sif docker://rocker/rstudio:latest
```
Similarly, for the Bioconductor Docker image available on 
```shell
https://hub.docker.com/r/bioconductor/bioconductor_docker:
singularity pull bioconductor_latest.sif docker://bioconductor/bioconductor_docker:latest
```

RStudio images are also available on Singularity Hub. Obtaining them from Singularity Hub would save disk space occupied by intermediate files during the conversion from Docker.

```shell
singularity pull --name singularity-rstudio.sif shub://nickjer/singularity-rstudio
```

Save the RStudio image to your project or scratch directory.

## Part2: This exercise assumes you have a Rstudio container image

This example bind mounts the /project/gstein directory on the host into the Singularity container.
By default the only host file systems mounted within the container are $HOME, /tmp, /proc, /sys, and /dev.
type in the password you want to set, make it more complicated than this dummy one

```bash
#!/bin/sh
#SBATCH --time=08:00:00
#SBATCH --signal=USR2
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=2
#SBATCH --mem=8192
#SBATCH --output=/home/%u/rstudio-server.job.%j
# customize --output path as appropriate (to a directory readable only by the user!)
projectdir=/project/gstein

# Create temporary directory to be populated with directories to bind-mount in the container
# where writable file systems are necessary. Adjust path as appropriate for your computing environment.
workdir=$(python -c 'import tempfile; print(tempfile.mkdtemp())')

mkdir -p -m 700 ${workdir}/run ${workdir}/tmp ${workdir}/var/lib/rstudio-server
cat > ${workdir}/database.conf <<END
provider=sqlite
directory=/var/lib/rstudio-server
END

# Set OMP_NUM_THREADS to prevent OpenBLAS (and any other OpenMP-enhanced
# libraries used by R) from spawning more threads than the number of processors
# allocated to the job.
#
# Set R_LIBS_USER to a path specific to rocker/rstudio to avoid conflicts with
# personal libraries from any R installation in the host environment

cat > ${workdir}/rsession.sh <<END
#!/bin/sh
export OMP_NUM_THREADS=${SLURM_JOB_CPUS_PER_NODE}
export R_LIBS_USER=${HOME}/R/container-rstudio/4.3
exec rsession "\${@}"
END

chmod +x ${workdir}/rsession.sh

export SINGULARITY_BIND="${workdir}/run:/run,${workdir}/tmp:/tmp,${workdir}/database.conf:/etc/rstudio/database.conf,${workdir}/rsession.sh:/etc/rstudio/rsession.sh,${workdir}/var/lib/rstudio-server:/var/lib/rstudio-server,$PWD:/run/user"

# Do not suspend idle sessions.
# Alternative to setting session-timeout-minutes=0 in /etc/rstudio/rsession.conf
# https://github.com/rstudio/rstudio/blob/v1.4.1106/src/cpp/server/ServerSessionManager.cpp#L126
export SINGULARITYENV_RSTUDIO_SESSION_TIMEOUT=0

export SINGULARITYENV_USER=$(id -un)
export SINGULARITYENV_PASSWORD=$(openssl rand -base64 15)
# get unused socket per https://unix.stackexchange.com/a/132524
# tiny race condition between the python & singularity commands
readonly PORT=$(python -c 'import socket; s=socket.socket(); s.bind(("", 0)); print(s.getsockname()[1]); s.close()')
cat 1>&2 <<END
1. SSH tunnel from your workstation using the following command:

   ssh -N -L 8787:${HOSTNAME}:${PORT} ${SINGULARITYENV_USER}@midway3.rcc.uchicago.edu

   and point your web browser to http://localhost:8787

2. log in to RStudio Server using the following credentials:

   user: ${SINGULARITYENV_USER}
   password: ${SINGULARITYENV_PASSWORD}

When done using RStudio Server, terminate the job by:

1. Exit the RStudio Session ("power" button in the top right corner of the RStudio window)
2. Issue the following command on the login node:

      scancel -f ${SLURM_JOB_ID}
END

singularity exec --cleanenv rstudio_latest.sif \
    rserver --server-user ${USER} \
            --www-port ${PORT} \
            --auth-none=0 \
            --auth-pam-helper-path=pam-helper \
            --auth-stay-signed-in-days=30 \
            --auth-timeout-minutes=0 \
            --rsession-path=/etc/rstudio/rsession.sh
printf 'rserver exited' 1>&2
```

**Output**

```plaintext
1. SSH tunnel from your workstation using the following command:

   ssh -N -L 8787:midway3-0149:44415 pnsinha@midway3.rcc.uchicago.edu

   and point your web browser to http://localhost:8787

2. log in to RStudio Server using the following credentials:

   user: pnsinha
   password: tF8jhmW/eiWtwoVC/SyR

When done using RStudio Server, terminate the job by:

1. Exit the RStudio Session ("power" button in the top right corner of the RStudio window)
2. Issue the following command on the login node:

      scancel -f 4093518
```
