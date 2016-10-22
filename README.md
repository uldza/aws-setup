# AWS setup

**To start instance:**

`terraform apply -var 'public_key_path=~/.ssh/id_rsa.pub' -var 'aws_access_key=YOUR_ACCESS_KEY' -var 'aws_secret_key=YOUR_SECRET_KEY' -var 'aws_instance_type=t2.micro'`

---

**To destroy resources:**

`terraform destroy -var 'public_key_path=~/.ssh/id_rsa.pub' -var 'aws_access_key=YOUR_ACCESS_KEY' -var 'aws_secret_key=YOUR_SECRET_KEY' -var 'aws_instance_type=t2.micro'`

---

**Access this new ubuntu instace:**

`ssh ubuntu@0.0.0.0`
Replace 0.0.0.0 with you aws instance's public IP.


## Installing system (at the moment manually)

#### Update systems
`sudo add-apt-repository ppa:openjdk-r/ppa`
`sudo apt-get update`
`sudo apt-get upgrade -y`
`sudo apt-get install -y build-essential git swig default-jdk zip zlib1g-dev openjdk-8-jdk`

#### Get and run anaconda installer
`wget https://repo.continuum.io/archive/Anaconda3-4.2.0-Linux-x86_64.sh`
`bash Anaconda3-4.2.0-Linux-x86_64.sh`
(agree to terms, set up install dir, change path - this will be done automatically by anaconda install)

#### Configure Jupyther notebook (be sure to relogin before, since anaconda path variables will not be available without)
`jupyter notebook --generate-config`
Generate password: 
`key=$(python -c "from notebook.auth import passwd; print(passwd())")`

Create certificate:

```
cd ~
mkdir certs
cd certs
certdir=$(pwd)
openssl req -x509 -nodes -days 365 -newkey rsa:1024 -keyout mycert.key -out mycert.pem
# You'll be prompted to enter values for the cert,
# but it doesn't much matter, you can just leave them all blank.
```

Complete jupyther config:

```
cd ~
sed -i "1 a\
c = get_config()\\
c.NotebookApp.certfile = u'$certdir/mycert.pem'\\
c.NotebookApp.ip = '*'\\
c.NotebookApp.open_browser = False\\
c.NotebookApp.password = u'$key'\\
c.NotebookApp.port = 8888" .jupyter/jupyter_notebook_config.py
```

Create jupyther notebook dir
`mkdir ~/notebooks`
Launch jupyther (you can do it in seperate screen or tmux session)
`cd ~/notebooks`
`jupyter notebook --certfile=~/certs/mycert.pem --keyfile ~/certs/mycert.key`
Detach from screen on tmux session
You can acccess jupyther from browser now: https://[INSTANCES_IP]:8888

####
