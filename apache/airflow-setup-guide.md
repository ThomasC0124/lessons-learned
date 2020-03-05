Apache Airflow
==============

This markdown outlines the steps to install `apache-airflow` on AWS-EC2 Ubuntu instances and set up the environments for the GUI to be accessed remotely.

## Installation
1. Create a virtual environment and install `apache-airflow` inside the virtual environment
```sh
$ mkdir ~/airflow 	# create a directory for airflow and create the virtual environment inside
$ cd ~/airflow
$ virtualenv venv --python=python3.7	# please use at least Python3.6 or newer
$ source venv/bin/activate
$ (venv) pip install apache-airflow[crypto,postgres,s3,sendgrid]
$ (venv) pip install flask-bcrypt
```

2. (root) Install and set up PostgreSQL to support task executors other than the sequential executor
```sh
$ sudo apt update	# assume Ubuntu 18.04; for Ubuntu 14.04 or 16.04, please use `apt-get` instead
$ sudo apt install postgresql
$ sudo -i -u postgres 	# switch over to the postgres account
postgres $ psql
postgres=# CREATE DATABASE airflowdb;
postgres=# CREATE USER airflow WITH ENCRYPTED PASSWORD 'some-random-password';
postgres=# GRANT ALL PRIVILEGES ON DATABASE airflowdb TO airflow;
postgres=# \du 	# check for success
postgres=# \l 	# check for success
postgres=# \q	# exit the prompt
```

## Environment setup
1. Specify the home directory for `apache-airflow`
```sh
$ export AIRFLOW_HOME=~/airflow
$ echo $AIRFLOW_HOME	# check for success
```
Note: It is recommended that you save `export AIRFLOW_HOME=~/airflow` in the shell startup script, e.g., `.bashrc`, `.bash_profile` or `.profile`, so that `apache-airflow` knows where to locate the settings we are going to cover in the following sections.

2. Replace the default SQLite with PostgreSQL
```sh
$ cd $AIRFLOW_HOME
$ vi airflow.cfg
# Change the executor to Local Executor
executor = LocalExecutor
# Change the meta db configuration
sql_alchemy_conn = postgresql+psycopg2://airflow:some-random-password@localhost/airflowdb
```

3. Set up authentication and SSL connection
```sh
$ cd $AIRFLOW_HOME
$ vi airflow.cfg
# Turn on the web authentication
[webserver]
authenticate = True
auth_backend = airflow.contrib.auth.backends.password_auth
```

4. Create a user account
```sh
$ cd $AIRFLOW_HOME
$ source venv/bin/activate
$ (venv) python
# Within the Python interpreter
$ from airflow import models, settings
$ from airflow.contrib.auth.backends.password_auth import PasswordUser
$ user = PasswordUser(models.User())
$ user.username = 'your-username'
$ user.email = 'your-account@evolenthealth.com'
$ user.password = 'your-password'
$ session = settings.Session()
$ session.add(user)
$ session.commit()
$ session.close()
$ exit()
```

5. Update email backend
```sh
$ cd $AIRFLOW_HOME
$ vi airflow.cfg
# Change the following lines
[smtp]
smtp_host = smtp.sendgrid.net
smtp_starttls = True
smtp_ssl = False
smtp_user = apikey
smtp_password = <sendgrid-api-key-goes-here>
smtp_port = 587
smtp_mail_from = airflow@airflow.evolent.io
```

6. Start `apache-airflow`
```sh
$ cd $AIRFLOW_HOME
$ source venv/bin/activate
$ (venv) airflow initdb
$ (venv) airflow version # check for success
$ (venv) airflow webserver
$ (venv) airflow scheduler
```
