Installation
============

Install and Configure git on the local machine
----------------------------------------------
Install Git
```bash
sudo apt-get install git
```

Your email address should be the same one as for your github account.
```bash
git config --global user.email "my@address.com"
git config --global user.name "name in quotes"
```

Download buildout
-----------------

Change akorn-site here if you want to set up in a different folder than <akorn-site>

```bash
mkdir -p ~/sites/akorn-site
cd ~/sites/akorn-site
git clone https://github.com/oakling/akorn.buildout.git ./
```

Setup server
------------

```bash
sudo apt-get install $(cat ubuntu_requirements)
```

Install newer version of java
-----------------------------

These commands need to be run separately, as console response is required.

```bash
sudo add-apt-repository ppa:webupd8team/java
```

```bash
sudo apt-get update
```

```bash
sudo apt-get install oracle-java7-installer
```

It will ask you to accept the license

Install Python
--------------

Change akorn here, if you are installing as a different user

```bash
cd ~/Downloads
wget http://www.python.org/ftp/python/2.7.3/Python-2.7.3.tgz
tar zxfv Python-2.7.3.tgz
cd Python-2.7.3
./configure --prefix=/home/akorn/python/2.7.3
make
make install
cd ..
wget http://python-distribute.org/distribute_setup.py
~/python/2.7.3/bin/python distribute_setup.py
~/python/2.7.3/bin/easy_install pip
~/python/2.7.3/bin/pip install virtualenv
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
eggs-directory = /home/akorn/.buildout/eggs
download-cache = /home/akorn/.buildout/downloads
extends-cache = /home/akorn/.buildout/extends" >> ~/.buildout/default.cfg
```

Create A VirtualEnv
-------------------

Change to the right directory first

```bash
cd ~/sites/akorn-site
```

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

Setup the Sqlite DB
-------------------

You will be asked to create a super user when you first run syncdb. This is the user you can then log into the django admin with.

```bash
cd ~/sites/akorn-site/src/akorn_search/akorn_search
mkdir db
django syncdb
```

Build the static pages
----------------------

```bash
cd ~/sites/akorn-site/
django collectstatic
```

Start the services
------------------

```bash
supervisord
```

Setup the Couch DB
------------------

```bash
curl -X PUT http://localhost:5984/journals
curl -X PUT http://localhost:5984/store
cd ~/sites/akorn-site/src/akorn_search/akorn_search/dumps
couchdb-load --input=journals.couchdump http://localhost:5984/journals
couchdb-load --input=store.couchdump http://localhost:5984/store
```

Setup RabbitMQ
--------------

```bash
rabbitmqctl add_user akorn akorn
rabbitmqctl add_vhost myvhost
rabbitmqctl set_permissions -p myvhost akorn ".*" ".*" ".*"
```

Setup couchdb-lucene
--------------------

Add a design document to couchdb that hooks up to couchdb-lucene and indexes text. The easiest way
is using Futon (http://127.0.0.1:5984/_utils). Select the database (store), select
"design documents" in the drop down. Create a document and name it "_design/lucene". Add a field
"fulltext" to the document, an example to index all document titles:

```js
{
    "by_title": {
        "index": "function(doc) { var ret=new Document(); ret.add(doc.title,
            {'store':'yes'});
            ret.add(doc.journal_id, {'field':'journalID'}); return ret; }"
    },
    "by_journal_id": {
        "index": "function(doc) { var ret=new Document();
            ret.add(doc.journal_id); return ret; }"
    }
}
```

Make sure the design document is saved and restart the database:
```bash
supervisorctl stop all
supervisorctl start all
```


Supervisor
----------

To start supervisor, this will also start the applications.

```bash
supervisord
```

To check the status of the applications

```bash
supervisorctl status
```

You can also stop and start the applications with supervisorctl.

To stop supervisor completely

```bash
supervisorctl shutdown
```

Running the test suite
----------------------

```bash
nose akorn.celery
nose akorn.scrapers

django test search
```

Maintaining the servers
=======================

Akorn is currently deployed on 3 servers:
* Web server (web.private | akorn.org)
* Couchdb server (couchdb.private)
* Celery server (celery1.private)

If HTTP requests to akorn.org are slow or failing, then restart the Web server.

If web pages are displaying, but searches produce no results or take a long time, then restart the Couchdb server.

If new articles are not being scraped, then restart the Celery server.

If in doubt, go nuts! Just restart everything.

Accessing the servers
---------------------

First you need to request an SSH key, once you have received
the key you need to add it to your key chain.

Move the key into a sensible place, add the key to your SSH
config, and ensure your SSH config has the correct permissions.
```bash
mv /path/to/key.pem ~/.ssh/akorn.pem
echo IdentityFile ~/.ssh/akorn.pem >> ~/.ssh/config
chmod 0600 ~/.ssh/akorn.pem
```

Test your setup by loggig into the web server as ubuntu.
```bash
ssh ubuntu@akorn.org
```

Restarting web server
---------------------
```bash
ssh ubuntu@akorn.org
sudo service django-akorn restart
```

Restarting couchdb server
-------------------------
```bash
ssh ubuntu@akorn.org
ssh ubuntu@couchdb.private
sudo service couchdb restart
sudo service couchdb-lucene restart
```
_Note: The chained SSH commands are to avoid having to setup 3 shared SSH keys._

Restarting celery server
------------------------
```bash
ssh ubuntu@akorn.org
ssh ubuntu@celery1.private
sudo service celeryd restart
```
_Note: The chained SSH commands are to avoid having to setup 3 shared SSH keys._
