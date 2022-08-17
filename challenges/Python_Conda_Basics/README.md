# Python: Conda Basics

In high-performance computing, [Python](https://www.python.org/) is heavily used to analyze scientific data on the system.
Some users require specific versions of Python or niche scientific packages to analyze their data, which may further depend on numerous other Python packages.
Because of all the dependencies that some Python packages require, and all the types of data that exist, it can be quite troublesome to get different Python installations to "play nicely" with each-other, especially on an HPC system where the system environment is complicated.
[Conda](https://conda.io/projects/conda/en/latest/index.html), a package and virtual environment manager from the [Anaconda](https://www.anaconda.com/) distribution, helps alleviate these issues. 

Conda allows users to easily install different versions of binary software packages and any required libraries appropriate for their computing platform.
The versatility of conda allows a user to essentially build their own isolated Python environment, without having to worry about clashing dependencies and other system installations of Python.
Conda is available on OLCF systems, and loading the default python module loads an Anaconda Python distribution.
Loading this distribution automatically puts you in a "base" conda environment, which already includes packages that one can use for simulation, analysis, and machine learning.

This hands-on challenge will introduce a user to the basic workflow of using conda environments, as well as providing an example of how to create a conda environment that uses a different version of Python than the base environment uses on Summit.

## Inspecting and setting up the environment

First, we will unload all the current modules that you may have previously loaded on Summit and then immediately load the default modules.
Assuming you cloned the repository in your home directory:

```
$ cd ~/hands-on-with-summit/challenges/Python_Conda_Basics
$ source deactivate_envs.sh
$ module purge
$ module load DefApps
```

The `source deactivate_envs.sh` command is only necessary if you already have the Python module loaded.
The script unloads all of your previously activated conda environments, and no harm will come from executing the script if that does not apply to you.

Next, we need to load the python module and the gnu compiler module on Summit (most Python packages assume use of GCC)

```
$ module load gcc
$ module load python
```

This puts us in the "base" conda environment, which is the default Python environment after loading the module.
To see a list of environments, use the command `conda env list`:

```
$ conda env list

# conda environments:
#
base                  *  /sw/summit/python/3.8/anaconda3/2020.07-rhel8
```

This also is a great way to keep track of the locations and names of all other environments that have been created.
The current environment is indicated by `*`.

To see what packages are installed in the active environment, use `conda list`:

```
$ conda list

# packages in environment at /sw/summit/python/3.8/anaconda-base:
#
# Name                    Version                   Build  Channel
_ipyw_jlab_nb_ext_conf    0.1.0                    py36_0
_libgcc_mutex             0.1                        main
alabaster                 0.7.12                   py36_0
anaconda-client           1.7.2                    py36_0
anaconda-project          0.8.2                    py36_0
appdirs                   1.4.3            py36h28b3542_0
asn1crypto                0.24.0                   py36_0
astroid                   2.1.0                    py36_0
astropy                   3.1              py36h7b6447c_0
.
.
.
```

We can find the version of Python that exists in this base environment by executing: 

```
$ python --version

Python 3.8.3
```

## Creating a new environment

For this challenge, we are going to install a newer version of Python.

To do so, we will create a new environment using the `conda create` command:

```
$ conda create -p /ccs/proj/<YOUR_PROJECT_ID>/<YOUR_USER_ID>/conda_envs/summit/py3711-summit python=3.7.11
```

The "-p" flag specifies the desired path and name of your new virtual environment.
The directory structure is case sensitive, so be sure to insert "<YOUR_PROJECT_ID>" as lowercase.
Directories will be created if they do not exist already (provided you have write-access in that location).
Instead, one can solely use the `--name <your_env_name>` flag which will automatically use your `$HOME` directory.

> NOTE: It is highly recommended to create new environments in the "Project Home" directory (on Summit, this is `/ccs/proj/<YOUR_PROJECT_ID>/<YOUR_USER_ID>`).
> This space avoids purges, allows for potential collaboration within your project, and works better with the compute nodes.
> It is also recommended, for convenience, that you use environment names that indicate the hostname, as virtual environments created on one system will not necessarily work on others.

After executing the `conda create` command, you will be prompted to install "the following NEW packages" -- type "y" then hit Enter/Return.
Downloads of the fresh packages will start and eventually you should see something similar to:

```
Preparing transaction: done
Verifying transaction: done
Executing transaction: done
#
# To activate this environment, use
#
#     $ conda activate /ccs/proj/<YOUR_PROJECT_ID>/<YOUR_USER_ID>/conda_envs/summit/py3711-summit
#
# To deactivate an active environment, use
#
#     $ conda deactivate
```

Due to the specific nature of conda on Summit, we will be using `source activate` and `source deactivate` instead of `conda activate` and `conda deactivate`.
Let's activate our new environment:

```
$ source activate /ccs/proj/<YOUR_PROJECT_ID>/<YOUR_USER_ID>/conda_envs/summit/py3711-summit
```

The path to the environment should now be displayed in "( )" at the beginning of your terminal lines, which indicate that you are currently using that specific conda environment.
And if you check with `conda env list` again, you should see that the `*` marker has moved to your newly activated environment:

```
$ conda env list

# conda environments:
#
                      *  /ccs/proj/<YOUR_PROJECT_ID>/<YOUR_USER_ID>/conda_envs/summit/py3711-summit
base                     /sw/summit/python/3.8/anaconda3/2020.07-rhel8
```

## Installing packages

Next, let's install a package ([NumPy](https://numpy.org/)). 
There are a few different approaches.

One way to install packages into your conda environment is to build packages from source using [pip](https://pip.pypa.io/en/stable/).
This approach is useful if a specific package or package version is not available in the conda repository, or if the pre-compiled binaries don't work on the HPC resources (which is common).
However, building from source means you need to take care of some of the dependencies yourself, especially for optimization.
Pip is available to use after installing Python into your conda environment, which we have already done.

> NOTE: Because issues can arise when using conda and pip together (see link in [Additional Resources Section](#refs)), it is recommended to do this only if absolutely necessary.

To build a package from source, use `pip install --no-binary=<package_name> <package_name>`:

```
$ CC=gcc pip install --no-binary=numpy numpy
```

The `CC=gcc` flag will ensure that we are using the proper compiler and wrapper.
Building from source results in a longer installation time for packages, so you may need to wait a few minutes for the install to finish.

Congratulations, you have built NumPy from source in your conda environment!
However, we did not link in any additional linear algebra packages, so this version of NumPy is not optimized.
Let's install a more optimized version using a different method instead, but first we must uninstall the pip-installed NumPy:

```
$ pip uninstall numpy
```

The traditional, and more basic, approach to installing/uninstalling packages into a conda environment is to use the commands `conda install` and `conda remove`.
Installing packages with this method checks the [Anaconda Distribution Repository](https://docs.anaconda.com/anaconda/packages/pkg-docs/) for pre-built binary packages to install.
Let's do this to install NumPy:

```
$ conda install numpy
```

Because NumPy depends on other packages for optimization, this will also install all of its dependencies.
Congratulations, you have just installed an optimized version of NumPy, now let's test it!

## Testing your new environment

Let's run a small script to test that things installed properly.
Since we are running a small test, we can do this without having to run on a compute node. 

> NOTE: Remember, at larger scales both your performance and your fellow users' performance will suffer if you do not run on the compute nodes.
It is always highly recommended to run on the compute nodes (through the use of a batch job or interactive batch job).

Make sure you're in the correct directory and execute the example Python script:

```
$ cd ~/hands-on-with-summit/challenges/Python_Conda_Basics/
$ python3 hello.py

Hello from Python 3.7.11!
You are using NumPy 1.20.3
```

Congratulations, you have just created your own Python environment and ran on one of the fastest computers in the world!

## Additional Tips

* Cloning an environment:

    It is not recommended to try to install new packages into the base environment.
    Instead, you can clone the base environment for yourself and install packages into the clone.
    To clone an environment, you must use the `--clone <env_to_clone>` flag when creating a new conda environment.
    An example for cloning the base environment into your `$HOME` directory on Summit is provided below:

    ```
    $ conda create -p /ccs/home/<YOUR_USER_ID>/.conda/envs/baseclone-summit --clone base
    $ source activate /ccs/home/<YOUR_USER_ID>/.conda/envs/baseclone-summit
    ```

* Deleting an environment:

    If for some reason you need to delete an environment, you can execute the following:

    ```
    $ conda env remove -p /path/to/your/env
    ```

* Exporting (sharing) an environment:

    You may want to share your environment with someone else.
    As mentioned previously, one way to do this is by creating your environment in a shared location where other users can access it.
    A different way (the method described below) is to export a list of all the packages and versions of your environment (an `environment.yml` file).
    If a different user provides conda the list you made, conda will install all the same package versions and recreate your environment for them -- essentially "sharing" your environment.
    To export your environment list:
    
    ```
    $ source activate my_env
    $ conda env export > environment.yml
    ```
    
    You can then email or otherwise provide the `environment.yml` file to the desired person.
    The person would then be able to create the environment like so:
    
    ```
    $ conda env create -f environment.yml
    ```

* Adding known environment locations:

    For a conda environment to be callable by a "name", it must be installed in one of the `envs_dirs` directories.
    The list of known directories can be seen by executing:
    ```
    $ conda config --show envs_dirs
    ```
    On Summit, the default location is your `$HOME` directory.
    If you plan to frequently create environments in a different location than the default (such as `/ccs/proj/...`), then there is an option to add directories to the `envs_dirs` list.
    To do so, you must execute:
    ```
    $ conda config --append envs_dirs /ccs/proj/<YOUR_PROJECT_ID>/<YOUR_USER_ID>/conda_envs/summit
    ```
    This will create a `.condarc` file in your `$HOME` directory if you do not have one already, which will now contain this new envs_dirs location.
    This will now enable you to use the `--name env_name` flag when using conda commands for environments stored in that specific directory, instead of having to use the `-p /ccs/proj/<YOUR_PROJECT_ID>/<YOUR_USER_ID>/conda_envs/summit/env_name` flag and specifying the full path to the environment.
    For example, you can do `source activate py3711-summit` instead of `source activate /ccs/proj/<YOUR_PROJECT_ID>/<YOUR_USER_ID>/conda_envs/summit/py3711-summit`.

## Quick-Reference Commands

* List environments:

    ```
    $ conda env list
    ```

* List installed packages in current environment:

    ```
    $ conda list
    ```

* Creating an environment with Python version X.Y:

    For a **specific path**:

    ```
    $ conda create -p /path/to/your/my_env python=X.Y
    ```

    For a **specific name**:

    ```
    $ conda create -n my_env python=X.Y
    ```
       
* Deleting an environment:

    For a **specific path**:

    ```
    $ conda env remove -p /path/to/your/my_env
    ```

    For a **specific name**:

    ```
    $ conda env remove -n my_env
    ```

* Copying an environment:

    For a **specific path**:

    ```
    $ conda create -p /path/to/new_env --clone old_env
    ```

    For a **specific name**:

    ```
    $ conda create -n new_env --clone old_env
    ```
       
* Activating/Deactivating an environment:

    ```
    $ source activate my_env
    $ source deactivate # deactivates the current environment
    ```

* Installing/Uninstalling packages:

    Using **conda**:

    ```
    $ conda install package_name
    $ conda remove package_name
    ```

    Using **pip**:

    ```
    $ pip install package_name
    $ pip uninstall package_name
    $ pip install --no-binary=package_name package_name # builds from source
    ```

## <a name="refs"></a>Additional Resources

* [Conda User Guide](https://conda.io/projects/conda/en/latest/user-guide/index.html)
* [Anaconda Package List](https://docs.anaconda.com/anaconda/packages/pkg-docs/)
* [Pip User Guide](https://pip.pypa.io/en/stable/user_guide/)
* [Using Pip In A Conda Environment](https://www.anaconda.com/blog/using-pip-in-a-conda-environment)
