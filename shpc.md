

#Using singularity as a package

```bash
mkdir /software/shpc-latest-el8-x86_64
cd /software/shpc-latest-el8-x86_64

# Variable definitions for installation directories
install_dir="$(pwd)/singularity-hpc"
lib_dir="$install_dir/lib/python${python_ver%.*}/site-packages"
bin_dir="$install_dir/bin"
mod_dir="$install_dir/modules"

# Load required system modules
module load python/anaconda-2020.11

# Add installation paths to relevant environment variables
export PYTHONPATH="$lib_dir:$PYTHONPATH" 
export PATH="$bin_dir:$PATH"

# Get SHPC repo and checkout desired version/commit

git clone https://github.com/singularityhub/singularity-hpc.git
cd singularity-hpc
git checkout $shpc_checkout


# Pip installation
pip install -e . --prefix="$(pwd)"
exitcode="$?"
cd ..
echo ""
if [ "$exitcode" == "0" ] ; then
# Perform CUSTOM EDITS TO SHPC CONFIGURATION
 module load singularity
 shpc config set module_sys:tcl
 shpc config set container_base:\$root_dir/containers
 shpc config set singularity_module:singularity
 echo ""
 echo " Installation of SHPC was successful!"
# Print information on how to use SHPC
echo ""
 echo " To use SHPC, run the following:"
 echo "module load singularity"
 echo "export PYTHONPATH="$lib_dir:\$PYTHONPATH""
 echo "export PATH="$bin_dir:\$PATH"  "
# Print information on how to use Container Modules installed by SHPC
 echo ""
 echo " To use Container Modules, run the following:"
 echo "module use $mod_dir"
else
 echo "Installation was not successful. Review the process and try again."
fi
echo ""

exit



mkdir /software/shpc/modules
mkdir /software/shpc/registry
mv singularity-hpc/registry/* /software/shpc/registry/
mkdir /software/shpc/containers
chown ubuntu /software/shpc/containers
shpc config set container_base:/software/shpc/containers
shpc config set module_base:/software/shpc/modules
shpc config add registry:/software/shpc/registry
```
