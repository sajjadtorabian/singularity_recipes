# Singularity Recipes

## Nipype Plus Jupyter
We first need to download the Docker layers. Normally this happens automatically with `singularity build` or `singularity pull`, but if you aren't using the development version 3.0 branch of Singularity that has a fixed bug with respect to whiteout files, you will have issue when you do these commands with nipype (note that it has whiteout files). What we did (because didn't feel like installing another version of Singularity) was to do a pull of nipype with the [Singularity Global Client](https://singularityhub.github.io/sregistry-cli) that will download the fixed layers and then put them in the same cache that Singularity uses. Then we will have what we need :)  Here is how you can install and use the client:

```bash
$ pip install sregistry
$ sregistry pull nipype/nipype:latest
```

Now we want to debug the build and find the missing path! To do this, you can build a completely empty image to look around in. The recipe looks like this:

```
# Singularity.nipype-plus-jupyter-empty
Bootstrap: docker
From: nipype/nipype:latest
```
The above you can save to whatever file you want, we're calling ours `Singularity.nipype-plus-jupyter-empty` we can then build like this:

```
sudo singularity build npjup.simg Singularity.nipype-plus-jupyter-empty
```

Then we can shell inside and find locations of things.

```
singularity shell npjup.simg
```

We are able to see what will be exported in the environment at runtime, and then source it to add these locations to the path (so we can find executables there!)

```
cat /environment
$ cat /.singularity.d/env/10-docker.sh 
export PATH="/opt/conda/bin:/usr/lib/ants:/opt/afni:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin"
export LANG="en_US.UTF-8"
export LC_ALL="C.UTF-8"
export ND_ENTRYPOINT="/neurodocker/startup.sh"
export MATLABCMD="/opt/mcr/v92/toolbox/matlab"
export FORCE_SPMMCR="1"
export LD_LIBRARY_PATH="/usr/lib/x86_64-linux-gnu:/opt/mcr/v92/runtime/glnxa64:/opt/mcr/v92/bin/glnxa64:/opt/mcr/v92/sys/os/glnxa64:"
export FREESURFER_HOME="/opt/freesurfer"
export ANTSPATH="/usr/lib/ants"
export MKL_NUM_THREADS="1"
export OMP_NUM_THREADS="1"
export CONDA_DIR="/opt/conda"
```

There we see the location of conda! Strange that it wasn't where we expected. Let's add to the path and then we have pip

```
export PATH=/opt/conda/bin:$PATH
which pip
/opt/conda/bin/pip
```

Now we can update the recipe [Singularity.nipype.plus-jupyter](Singularity.nipype.plus-jupyter) with our found pip.

```
Bootstrap: docker
From: nipype/nipype:latest

%labels
  Maintainer Sajjad
  Version v1.0

%post
    export PATH=/opt/conda/bin:$PATH
    pip install --upgrade pip
    pip install jupyter
```

And build the image

```bash
sudo singularity build npjup.simg Singularity.nipype-plus-jupyter
```

:)
