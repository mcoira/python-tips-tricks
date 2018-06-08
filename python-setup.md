# Setting up Python environments, Jupyter notebooks and Pyspark

## Table of contents

* [Installing Homebrew](#installing-homebrew)
* [Installing python for common purposes (or not)](#installing-python-for-common-purposes-or-not)
* [Installing pandoc](#installing-pandoc)
* [Step 1: Install pyenv, virtualenv, and pyenv-virtualenv](#step-1-install-pyenv-virtualenv-and-pyenv-virtualenv)
* [Step 2: Create an isolated python environment](#step-2-create-an-isolated-python-environment)
  * [See all available python versions for pyenv](#see-all-available-python-versions-for-pyenv)
  * [Install the python versions you need](#install-the-python-versions-you-need)
  * [Potential errors installing python version](#potential-errors-installing-python-version)
* [Step 3: Creating virtual environments](#step-3-creating-virtual-environments)
* [Step 4: Configuring virtual evironments](#step-4-configuring-virtual-evironments)
* [Installing and configuring Spark](#installing-and-configuring-spark)
  * [Downloading Spark](#downloading-spark)
  * [Configuring PySpark with IPython Shell](#configuring-pyspark-with-ipython-shell)
  * [Creating pyspark kernel](#creating-pyspark-kernel)
  * [Configuring PySpark with Jupyter Notebook](#configuring-pyspark-with-jupyter-notebook)
  * [Confirming installation on Jupyter kernel list](#confirming-installation-on-jupyter-kernel-list)
  * [The final check of Pyspark-Jupyter combo](#the-final-check-of-pyspark-jupyter-combo)
* [Some useful commands](#some-useful-commands)
  * [Check where Jupyter is reading its configuration files](#check-where-jupyter-is-reading-its-configuration-files)
  * [pyenv which](#pyenv-which)
  * [pyenv local](#pyenv-local)
  * [Check libraries availability](#check-libraries-availability)

## Some info beforehand
* [pyEnv Command Reference](https://github.com/pyenv/pyenv/blob/master/COMMANDS.md)
* http://apple.stackexchange.com/questions/209572/how-to-use-pip-after-the-os-x-el-capitan-upgrade
* http://akbaribrahim.com/managing-multiple-python-versions-with-pyenv/
* http://www.alfredo.motta.name/create-isolated-jupyter-ipython-kernels-with-pyenv-and-virtualenv/
* https://medium.com/@henriquebastos/the-definitive-guide-to-setup-my-python-workspace-628d68552e14#.tw1jcq9jt

## Installing Homebrew
```bash
/usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
brew update
```

## Installing python for common purposes (or not)
brew install python

## Installing pandoc
brew install pandoc

## Step 1: Install pyenv, virtualenv, and pyenv-virtualenv

```bash
# Install a Python version management.
# https://github.com/pyenv/pyenv
# https://github.com/yyuu/pyenv
brew install pyenv

# pyenv-virtualenv is a pyenv plugin that provides features to manage virtualenvs and conda environments for Python on UNIX-like systems.
# https://github.com/pyenv/pyenv-virtualenv
brew install pyenv-virtualenv

# add to your .bashrc the following:
echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
TODO:
REMOVE 
# if we use virtualenv we should initialize it (https://github.com/yyuu/pyenv-virtualenv#installation)
# unless we use pyenv-virtualenvwrapper because activate both extensions causes conflicts (https://github.com/yyuu/pyenv-virtualenvwrapper/issues/28#issuecomment-177051559)
# echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
#echo 'pyenv virtualenvwrapper_lazy' >> ~/.bash_profile
```

## Step 2: Create an isolated python environment

### See all available python versions for pyenv
```bash
# You can see all the python versions available to pyenv:
pyenv install --list
```

### Install the python versions you need

```bash
pyenv install 3.6.4
```

### Potential errors installing python version

ERROR: The Python ssl extension was not compiled. Missing the OpenSSL lib?
If you have homebrew openssl and pyenv installed, you may need to tell the compiler where the openssl package is located:

```bash
CFLAGS="-I$(brew --prefix openssl)/include" \
LDFLAGS="-L$(brew --prefix openssl)/lib" \
pyenv install -v 2.7.13
```

In MacOS it can fail with the following error:

```bash
/bin/sh: line 1: 72774 Trace/BPT trap: 5       CC='clang' LDSHARED='clang -bundle -undefined dynamic_lookup -L/usr/local/opt/readline/lib -L/usr/local/opt/readline/lib -L/usr/local/opt/openssl/lib -L/Users/mikel/.pyenv/versions/2.7.13/lib -L/usr/local/opt/openssl/lib' OPT='-DNDEBUG -g -fwrapv -O3 -Wall -Wstrict-prototypes' _TCLTK_INCLUDES='' _TCLTK_LIBS='' ./python.exe -E ./setup.py $quiet build
make: *** [sharedmods] Error 133
BUILD FAILED (OS X 10.11.6 using python-build 20160602)
```

It can be fixed with the command below:

```bash
xcode-select --install
```

More problems can be discover [here](https://github.com/yyuu/pyenv/wiki/Common-build-problems).

## Step 3: Creating virtual environments

Jupyter supports many kernels. This allows a single Jupyter install to create notebooks for Python2, Python3, R, Bash and many other languages

```bash
pyenv virtualenv 3.6.4 ipython3
pyenv virtualenv 3.6.4 jupyter
```

## Step 4: Configuring virtual evironments

```bash

pyenv activate ipython3
# enable kernel for python 3
pip install ipykernel
python -m ipykernel install --user
# install libraries
pip install numpy scipy pandas nltk scikit-learn matplotlib seaborn 
pip install pyproj LatLon haversine shapely fiona geojson geopandas geopy
pip install rtree # Spatial indexing for Python - http://toblerity.org/rtree/
pip install sphinx sphinx_rtd_theme nbsphinx Jinja2
pip install requests
pip install tesseract-ocr pytesseract
pip install Pillow # The friendly PIL fork (Python Imaging Library) - http://pillow.readthedocs.io/en/latest/
pip install xlrd # Working with Excel Files in Python - http://www.python-excel.org/
pip install bs4 # Beautiful Soup is a Python library for pulling data out of HTML and XML files - ttps://www.crummy.com/software/BeautifulSoup/
pip install dbfread # Read DBF Files with Python - https://dbfread.readthedocs.io/en/latest/
#pip install pyldavis
pip install pyyaml
pyenv deactivate

# Now let’s install tools which run on Python3
pyenv activate jupyter
pip install jupyter
pyenv deactivate

## toree is not supporting spark >= 2.0 so we can't use it yet
##pip install toree
##jupyter toree install --spark_home=/Users/mikel/Applications/spark --interpreters=Scala,PySpark
```

Finally, it’s time to make all Python versions and special virtualenvs work with each other.

```bash
pyenv global 3.6.4 ipython3 jupyter
```

Then you have to install libspatialindex with Brew using the following formula `brew install spatialindex` or you can compile it:

```bash
wget http://download.osgeo.org/libspatialindex/spatialindex-src-1.8.5.tar.gz
tar -zxvf spatialindex-src-1.8.5.tar.gz
cd spatialindex-src-1.8.5/
su
./configure; make; make install
sudo ldconfig
```

## Installing CLI tools with [pipsi](https://github.com/mitsuhiko/pipsi)

```bash
# Download the pipsi installer script from github
$ curl -O https://raw.githubusercontent.com/mitsuhiko/pipsi/master/get-pipsi.py

# Read about the --src option in the help menu
$ python get-pipsi.py --help

# Run the installer script to get the master branch from github
$ python get-pipsi.py --src=git+https://github.com/mitsuhiko/pipsi.git#egg=pipsi
```

```bash
echo 'export PATH=/Users/mcoira/.local/bin:$PATH' >> ~/.bash_profile
```

```bash
# Now let’s install tools which run on cli
pipsi install hovercraft
pipsi install SimpleHTTPServer 
pipsi install scrapy
pipsi install Pygments
```

## Installing and configuring Spark

### Downloading Spark

Download and unzip Spark:

```bash
# Add SPARK_HOME to .bash_profile
export SPARK_HOME=~/Applications/spark-2.2.1-bin-hadoop2.7
export PATH=$SPARK_HOME/bin:$PATH
```

### Configuring PySpark with IPython Shell

```bash
pyenv activate ipython3
ipython profile create pyspark
pyenv deactivate
```

### Creating pyspark kernel

First create the setup file:

```bash 
nano ~/.ipython/profile_pyspark/startup/00-pyspark-setup.py
```

with the content below:

```python
import os
import sys
spark_home = os.environ.get('SPARK_HOME', None)
if not spark_home:
    raise ValueError('SPARK_HOME environment variable is not set')
sys.path.insert(0, os.path.join(spark_home, 'python'))
sys.path.insert(0, os.path.join(spark_home, 'python/lib/py4j-0.10.4-src.zip'))
exec(open(os.path.join(spark_home, 'python/pyspark/shell.py')).read())
```

Try pyspark with ipython with `ipython --profile=pyspark`:

```
Welcome to
      ____              __
     / __/__  ___ _____/ /__
    _\ \/ _ \/ _ `/ __/  '_/
   /__ / .__/\_,_/_/ /_/\_\   version 2.2.1
      /_/

Using Python version 3.6.0 (default, Feb 20 2018 09:22:25)
SparkSession available as 'spark'.
```

Note: To use Python 2 instead of Python 3 we should do `ipython2 --profile=pyspark`.

### Configuring PySpark with Jupyter Notebook

Create the kernel configuration file `nano ~/Library/Jupyter/kernels/pyspark/kernel.json` with the content below:

```json
{
    "display_name": "PySpark (Spark 2.2.1)",
    "language": "python",
    "argv": [
        "/Users/mcoira/.pyenv/versions/3.6.0/envs/ipython3/bin/python",
        "-m",
        "ipykernel",
        "--profile=pyspark",
        "-f",
        "{connection_file}"
    ]
}
```

### Confirming installation on Jupyter kernel list

```jupyter kernelspec list```

### The final check of Pyspark-Jupyter combo

Open a new notebook with `jupyter notebook` and then create a new kernel for pyspark and try the code below:

```python
x=sc.range(5)
print (x)
y = sc.parallelize(range(5), 5)
print (y)

%matplotlib inline
import matplotlib.pyplot as plt
plt.bar(x.collect(), y.map(lambda x: x * 2).collect(), color='#D94426')
plt.show()
```

## Some useful commands

### Check where Jupyter is reading its configuration files

```bash
jupyter --paths

config:
    /Users/mcoira/.jupyter
    /Users/mcoira/.pyenv/versions/3.6.0/envs/ipython3/etc/jupyter
    /usr/local/etc/jupyter
    /etc/jupyter
data:
    /Users/mcoira/Library/Jupyter
    /Users/mcoira/.pyenv/versions/3.6.0/envs/ipython3/share/jupyter
    /usr/local/share/jupyter
    /usr/share/jupyter
runtime:
    /Users/mcoira/Library/Jupyter/runtime
```

### pyenv which

Displays the full path to the executable that pyenv will invoke when you run the given command.

```bash
~$ pyenv which python
/Users/mikel/.pyenv/versions/3.6.0/bin/python

~$ pyenv which python2
/Users/mikel/.pyenv/versions/2.7.13/bin/python2

~$ pyenv which ipython3
/Users/mcoira/.pyenv/versions/ipython3/bin/ipython3

~$ pyenv which ipython2
/Users/mikel/.pyenv/versions/ipython2/bin/ipython2
```

### pyenv local

Sets the Python version to be used by a specific application. This command writes the specified version number to the `.python-version` file in the in the current directory. This version overrides the global version and is used for the current directory and all its sub-directories.

```
$ pyenv local system
```

You can also unset the local version with the following command. This deletes the `.python-version` file from the current directory.

```
$ pyenv local --unset
```

When run without a version number, it displays the currently configured local version.

```
$ pyenv local
```

### Check libraries availability

```
ipython
In [1]: import numpy
In [2]: print numpy.version.full_version
In [3]: import scipy
In [4]: print scipy.version.full_version
In [5]: import pandas
In [6]: print pandas.version.version
In [7]: import nltk
In [8]: print nltk.version_info
In [9]: import matplotlib
In [10]: print matplotlib.__version__
```
