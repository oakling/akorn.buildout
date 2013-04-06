Installation
============

Setup server
------------

```bash
sudo apt-get install $(cat ubuntu_requirements)
```

Install Python
--------------

```bash
cd ~/Downloads
wget http://www.python.org/ftp/python/2.7.3/Python-2.7.3.tgz
tar zxfv Python-2.7.3.tgz
cd Python-2.7.3
./configure --prefix=/home/<username>/python/2.7.3
make
make install
cd ..
wget http://python-distribute.org/distribute_setup.py
~/python/2.7.3/bin/python distribute_setup.py
~/python/2.7.3/bin/easy_install virtualenv
```

Setup the buildout cache
------------------------

```bash
mkdir ~/.buildout
cd ~/.buildout
mkdir eggs
mkdir downloads
mkdir extends
```

Write the buildout default file
-------------------------------
```bash
echo "[buildout]
eggs-directory = /home/<username>/.buildout/eggs
download-cache = /home/<username>/.buildout/downloads
extends-cache = /home/<username>/.buildout/extends" >> ~/.buildout/default.cfg
```

Download buildout
-----------------

```bash
mkdir -p ~/sites/<my_test_instance>
cd ~/sites/<my_test_instance>
git clone https://github.com/oakling/akorn.buildout.git ./
```

Configure git on the local machine
----------------------------------
Your email address should be the same one as for your github account.
```bash
git config --global user.email "my@address.com"
git config --global user.name "name in quotes"
```

Create A VirtualEnv
-------------------

```bash
~/python/2.7.3/bin/virtualenv .
source bin/activate
pip install zc.buildout
pip install distribute
buildout init
```

Run the buildout
----------------

```bash
buildout -c development.cfg
```

Setup the Couch DB
------------------

```bash
sudo update-rc.d couchdb enable
sudo service couchdb start
curl -X PUT http://localhost:5984/journals
curl -X PUT http://localhost:5984/store
cd src/akorn_search/akorn_search/dumps
couchdb-load --input=<journals-DD-MM-YY> http://localhost:5984/journals
couchdb-load --input=<store-DD-MM-YY> http://localhost:5984/store
```

Setup the Sqlite DB
-------------------

You will be asked to create a super user when you first run syncdb. This is the user you can then log into the django admin with.

**TODO: Clariy path required**

```bash
cd ..
mkdir db
django syncdb
```

Start up the server
-------------------

```bash
django runserver
```

Running the test suite
----------------------

```bash
nose akorn.celery
nose akorn.scrapers

django test search
```
