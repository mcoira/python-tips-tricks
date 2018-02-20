# Setting up Python environments, Jupyter notebooks and Pyspark

Some info beforehand
* http://apple.stackexchange.com/questions/209572/how-to-use-pip-after-the-os-x-el-capitan-upgrade
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

# pyenv-virtualenvwrapper an alternative approach to manage virtualenvs from pyenv.
# https://github.com/pyenv/pyenv-virtualenvwrapper
brew install pyenv-virtualenvwrapper

# add to your .bashrc the following:
echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
# if we use virtualenv we should initialize it (https://github.com/yyuu/pyenv-virtualenv#installation)
# unless we use pyenv-virtualenvwrapper because activate both extensions causes conflicts (https://github.com/yyuu/pyenv-virtualenvwrapper/issues/28#issuecomment-177051559)
# echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
echo 'pyenv virtualenvwrapper_lazy' >> ~/.bash_profile
```

## Step 2: Create an isolated python environment

### See all available python versions for pyenv
```bash
# You can see all the python versions available to pyenv:
pyenv install --list
```

### Install the python versions you need

```bash
# pyenv install anaconda3-4.3.0
pyenv install 2.7.13
pyenv install 3.6.0
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

## Creating virtual environments

Jupyter supports many kernels. This allows a single Jupyter install to create notebooks for Python2, Python3, R, Bash and many other languages

```bash
# spark 2.1 runs over python 3.4+
#pyenv virtualenv anaconda3-4.3.0 anaconda3
pyenv virtualenv 2.7.13 ipython2
pyenv virtualenv 3.6.0 ipython3
pyenv virtualenv 3.6.0 tools3
```

### Configuring virtual evironments

```bash

pyenv activate ipython2
# enable kernel for python 2
pip install ipykernel
python -m ipykernel install --user
# install libraries
pip install numpy scipy matplotlib pandas nltk
pyenv deactivate

#pyenv activate anaconda3
#pip install ipython
#pip install ipykernel
#pip install numpy scipy matplotlib pandas nltk
#conda install -c dato-internals graphlab-create
#pip install pyldavis
#pip install sphinx sphinx_rtd_theme nbsphinx
#pip install Jinja2
#pyenv deactivate

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
pyenv deactivate

# Now let’s install tools which run on Python3
pyenv activate tools3
pip install jupyter
pip install hovercraft
pip install SimpleHTTPServer 
pip install scrapy
pip install Pygments
pyenv deactivate

## toree is not supporting spark >= 2.0 so we can't use it yet
##pip install toree
##jupyter toree install --spark_home=/Users/mikel/Applications/spark --interpreters=Scala,PySpark
```

Finally, it’s time to make all Python versions and special virtualenvs work with each other.

```bash
pyenv global 3.6.0 2.7.13 ipython3 ipython2 tools3
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
#pyenv activate anaconda3
#ipython profile create pyspark
#pyenv deactivate
#pyenv activate ipython2
#ipython2 profile create pyspark
#pyenv deactivate
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

Verify that this works.

todo: python 3 should be supported by spark 2.2 so we have to try 
# we have to use ipython2 instead of ipython because python3 is not supported yet by pyspark

```
ipython --profile=pyspark
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

```
# "/Users/mikel/.pyenv/versions/3.4.5/envs/jupyter3/bin/python"
nano ~/Library/Jupyter/kernels/pyspark/kernel.json
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
