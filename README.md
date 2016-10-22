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
```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get upgrade -y
sudo apt-get install -y build-essential git swig zip zlib1g-dev oracle-java8-installer
```

#### Get and run anaconda installer
```
wget https://repo.continuum.io/archive/Anaconda3-4.2.0-Linux-x86_64.sh
bash Anaconda3-4.2.0-Linux-x86_64.sh
```
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

```
cd ~/notebooks
jupyter notebook --certfile=~/certs/mycert.pem --keyfile ~/certs/mycert.key
```

Detach from screen on tmux session, you can acccess jupyther from browser now: https://[INSTANCES_IP]:8888

#### Install [Bazel](https://www.bazel.io/versions/master/docs/install.html) required for Tensorflow

```
echo "deb [arch=amd64] http://storage.googleapis.com/bazel-apt stable jdk1.8" | sudo tee /etc/apt/sources.list.d/bazel.list
curl https://storage.googleapis.com/bazel-apt/doc/apt-key.pub.gpg | sudo apt-key add -
sudo apt-get update
sudo apt-get install bazel
sudo apt-get upgrade bazel
```
---

#### Install Nvidia toolkit

`wget https://developer.nvidia.com/compute/cuda/8.0/prod/local_installers/cuda_8.0.44_linux-run`

Check md5 checksum:

```
cs=$(md5sum cuda_8.0.44_linux-run | cut -d' ' -f1)
if [ "$cs" != "6dca912f9b7e2b7569b0074a41713640" ]; then echo "WARNING: Unverified MD5 hash"; fi
```

Next run Nvidia install scripts
```
chmod +x cuda_8.0.44_linux-run
./cuda_8.0.44_linux-run -extract=`pwd`/nvidia_installers
pushd nvidia_installers
sudo ./NVIDIA-Linux-x86_64-346.46.run
# When prompted you'll need to:
# * Agree to the license terms
# * Accept the X Library path and X module path
# * Accept that 32-bit compatibility files will not be installed
# * Review and accept the libvdpau and libvdpau_trace libraries notice
# * Choose `Yes` when asked about automatically updating your X configuration file
# * Verify successful installation by choosing `OK`
sudo modprobe nvidia
```

Now we can run the CUDA installation script

```
sudo ./cuda-linux64-rel-8.0.44-19326674.run
# When prompted, you'll need to:
# * Accept the license terms (long scroll, page down with `f`)
# * Use the default installation path
# * Answer `n` to desktop shortcuts
# * Answer `y` to create a symbolic link
```

### TODO
* Install Nvidia toolkit
* Install CudNN
* Install tensorflow for anaconda with GPU support
