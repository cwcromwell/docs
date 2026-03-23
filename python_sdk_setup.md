# Get started with the eBay Python Feed SDK

Here's what I did:

* Cloned the repository 
* Installed and debugged dependencies
* Verified usability 

Up next: 

* Download a feed file
* Filter through the file

I am on macOS Tahoe 26.2


## Setting up the repo and dependencies

1.  Cloned the  repo 

~~~
git clone https://github.com/eBay/FeedSDK-Python.git

~~~

2. Installed requirements with pip. (I actually have pip3, rather than pip, because of the requirements of macOS and mac silicon.)

~~~
pip3 install -r requirements.txt

~~~

### Problems: Module not found and python version mismatch

Response was an error. Here's partial output: 

~~~
 ModuleNotFoundError: No module named 'pkg_resources'
  error: subprocess-exited-with-error

~~~

 I ran the same command with verbose output for more information and found that two things were happening: 
 
 * Most of the packages were calling for Python 3.11, and I am running a different version.
 * For some reason, the required module 'pkg_resources' was not found.


### Solution: Addressed version mismatch

 I resolved the Python version issue first, by using a virtual environment (`pyenv`) to install Python 3.11 and use it locally, which will avoid breaking other scripts.

First I installed the correct version of Python. 

~~~
$ pyenv install 3.11

~~~

Then I installed the virtual environment in my FeedSDK-Python directory. 

~~~
python -m venv venv   

~~~

Then started the virtual environment: 

~~~
source venv/bin/activate

~~~


### Solution: Panda install subprocess was using rogue Python version

This one step where I needed Claude's help.
The initial advice was to reinstall and then force-reinstall the setuptools module. The error message persisted. When none of that worked, I provided every bit of the error text from the console, and Claude identified the issue: the subprocess was running its own version of Python and not using the versions of Python or setuptools I had installed. Claude suggested unblocking the `pandas` installation as follows: 

~~~
pip install --no-build-isolation pandas

~~~

It worked. Here's the output:

~~~
Collecting pandas
  Downloading pandas-3.0.1-cp311-cp311-macosx_11_0_arm64.whl.metadata (79 kB)
Collecting numpy>=1.26.0 (from pandas)
  Downloading numpy-2.4.3-cp311-cp311-macosx_14_0_arm64.whl.metadata (6.6 kB)
Collecting python-dateutil>=2.8.2 (from pandas)
  Using cached python_dateutil-2.9.0.post0-py2.py3-none-any.whl.metadata (8.4 kB)
Collecting six>=1.5 (from python-dateutil>=2.8.2->pandas)
  Downloading six-1.17.0-py2.py3-none-any.whl.metadata (1.7 kB)
Downloading pandas-3.0.1-cp311-cp311-macosx_11_0_arm64.whl (9.9 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 9.9/9.9 MB 19.8 MB/s  0:00:00
Downloading numpy-2.4.3-cp311-cp311-macosx_14_0_arm64.whl (5.5 MB)
   ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 5.5/5.5 MB 35.4 MB/s  0:00:00
Using cached python_dateutil-2.9.0.post0-py2.py3-none-any.whl (229 kB)
Downloading six-1.17.0-py2.py3-none-any.whl (11 kB)
Installing collected packages: six, numpy, python-dateutil, pandas
Successfully installed numpy-2.4.3 pandas-3.0.1 python-dateutil-2.9.0.post0 six-1.17.0

~~~

According to Claude this command "tells pip to use your current environments packages (including your freshly installed setuptools) instead of spinning up a clean subprocess."

I finished the dependencies by removing pandas from the `requirements.txt` file and re-running the installation command: 

~~~
pip3 install -r requirements.txt

~~~

The installation of remaining dependencies had this output: 

~~~
Collecting urllib3==1.26.5 (from -r requirements.txt (line 1))
  Using cached urllib3-1.26.5-py2.py3-none-any.whl.metadata (43 kB)
Collecting certifi==2019.3.9 (from -r requirements.txt (line 2))
  Using cached certifi-2019.3.9-py2.py3-none-any.whl.metadata (2.5 kB)
Collecting aenum==2.1.2 (from -r requirements.txt (line 3))
  Using cached aenum-2.1.2-py3-none-any.whl.metadata (3.1 kB)
Collecting SQLAlchemy==1.3.3 (from -r requirements.txt (line 4))
  Downloading SQLAlchemy-1.3.3.tar.gz (5.9 MB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 5.9/5.9 MB 13.6 MB/s  0:00:00
  Installing build dependencies ... done
  Getting requirements to build wheel ... done
  Preparing metadata (pyproject.toml) ... done
Downloading urllib3-1.26.5-py2.py3-none-any.whl (138 kB)
Downloading certifi-2019.3.9-py2.py3-none-any.whl (158 kB)
Downloading aenum-2.1.2-py3-none-any.whl (39 kB)
Building wheels for collected packages: SQLAlchemy
  Building wheel for SQLAlchemy (pyproject.toml) ... done
  Created wheel for SQLAlchemy: filename=sqlalchemy-1.3.3-cp311-cp311-macosx_26_0_arm64.whl size=1136791 sha256=32f56f209f1aee37bf6ce6751d2bedc45db99caf42be2398c704d0060ab33698
  Stored in directory: /Users/clarencecromwell/Library/Caches/pip/wheels/36/11/5a/784378db04e0196f4acc204c24784e530e5b794faf142bab8b
Successfully built SQLAlchemy
Installing collected packages: certifi, aenum, urllib3, SQLAlchemy
Successfully installed SQLAlchemy-1.3.3 aenum-2.1.2 certifi-2019.3.9 urllib3-1.26.5

~~~

This installation was done — almost. 


3. I attempted to test the SDK by using the --help flag, but there were still problems. 

~~~
python feed_cli.py --help

~~~

First, I changed permissions of the file to make it executable: 

~~~
chmod 744 feed_cli.py

~~~

Then I tried the `--help` flag again and there was a new error message.


### Problem: Module called `getargspec` is nowhere to be found

The error message read in part:

~~~
    args, varargs, keywords, defaults = inspect.getargspec(method)
                                        ^^^^^^^^^^^^^^^^^^
AttributeError: module 'inspect' has no attribute 'getargspec'. Did you mean: 'getargs'?

~~~

### Solution: Change everything to Python 3.10

The issue was Python version incompatibility with the project. After a bit of back-and-forth with Claude, including an attempt to try different versions of `aenum`, I changed the local python version in my virtual environment to Python 3.10 and re-installed the dependencies. This meant running all the following: 

~~~
pyenv install 3.10
pyenv local 3.10
python -m venv venv --clear
source venv/bin/activate

pip3 install -r requirements.txt 
pip install --no-build-isolation pandas

~~~

The first four commands were to get the right version of Python running in the local environment, and the final two were needed to reinstall the dependencies. I installed `pandas` separately again, since that had been necessary in the first attempt. 


Finally, I ran the SDK with the help flag, successfully.

~~~
python3 feed_cli.py --help

~~~

Here's the output from that command. 

~~~     
usage: FeedSDK [-h] [-dt DT] -c1 C1 [-scope {ALL_ACTIVE,NEWLY_LISTED}] [-mkt MKT] [-token TOKEN]
               [-env {SANDBOX,PRODUCTION}] [-lf LF [LF ...]] [-sellerf SELLERF [SELLERF ...]] [-locf LOCF [LOCF ...]]
               [-pricelf PRICELF] [-priceuf PRICEUF] [-epidf EPIDF [EPIDF ...]] [-iepidf IEPIDF [IEPIDF ...]]
               [-gtinf GTINF [GTINF ...]] [-itemf ITEMF [ITEMF ...]] [-dl DOWNLOADLOCATION] [--filteronly]
               [-format FORMAT] [-qf QF]

Feed SDK CLI

options:
  -h, --help            show this help message and exit
  -dt DT                the date when feed file was generated
  -c1 C1                the l1 category id of the feed file
  -scope {ALL_ACTIVE,NEWLY_LISTED}
                        the feed scope. Available scopes are ALL_ACTIVE or NEWLY_LISTED
  -mkt MKT              the marketplace id for which feed is being requested. For example - EBAY_US
  -token TOKEN          the oauth token for the consumer. Omit the word 'Bearer'
  -env {SANDBOX,PRODUCTION}
                        environment type. Supported Environments are SANDBOX and PRODUCTION
  -lf LF [LF ...]       list of leaf categories which are used to filter the feed
  -sellerf SELLERF [SELLERF ...]
                        list of seller names which are used to filter the feed
  -locf LOCF [LOCF ...]
                        list of item locations which are used to filter the feed
  -pricelf PRICELF      lower limit of the price range for items in the feed
  -priceuf PRICEUF      upper limit of the price range for items in the feed
  -epidf EPIDF [EPIDF ...]
                        list of epids which are used to filter the feed
  -iepidf IEPIDF [IEPIDF ...]
                        list of inferred epids which are used to filter the feed
  -gtinf GTINF [GTINF ...]
                        list of gtins which are used to filter the feed
  -itemf ITEMF [ITEMF ...]
                        list of item IDs which are used to filter the feed
  -dl DOWNLOADLOCATION, --downloadlocation DOWNLOADLOCATION
                        override for changing the directory where files are downloaded
  --filteronly          filter the feed file that already exists in the default path or the path specified by -dl,
                        --downloadlocation option. If --filteronly option is not specified, the feed file will be
                        downloaded again
  -format FORMAT        feed and filter file format. Default is gzip
  -qf QF                any other query to filter the feed file. See Python dataframe query format

~~~


### Revised setup instructions

Here's the straightforward way to set up FeedSDK-Python on systems like mine. 

1. Clone the repository

~~~
git clone https://github.com/eBay/FeedSDK-Python.git

~~~

1. Use `pyenv` to install Python 3.10, if it isn't installed already, and make it the local version in the FeedSDK-Python repo

~~~
pyenv install 3.10
pyenv local 3.10

~~~


1. Create a virtual environment and start it up.

~~~
python -m venv venv   
source venv/bin/activate

~~~

1. Install the dependencies.

~~~
pip3 install -r requirements.txt 

~~~

If you have any issues with `pandas` install it using `--no-build-isolation`.

1. Test that everything works by outputting the help file. 

~~~
python3 feed_cli.py --help

~~~

(To be continued with downloading a file and filtering, after my developer account is approved. )
