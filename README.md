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

#### [TODO] Prerequisites for Nvidia toolkit
Blacklist Nouveau - in conflict with the nvidia driver

```
echo -e "blacklist nouveau\nblacklist lbm-nouveau\noptions nouveau modeset=0\nalias nouveau off\nalias lbm-nouveau off\n" | sudo tee /etc/modprobe.d/blacklist-nouveau.conf
echo options nouveau modeset=0 | sudo tee -a /etc/modprobe.d/nouveau-kms.conf
sudo update-initramfs -u
sudo reboot
```

Log in once back up

```
sudo apt-get install -y linux-image-extra-virtual
sudo reboot
```

Install more deps
```
sudo apt-get install -y linux-source linux-headers-`uname -r`
```

#### [TODO] Install Nvidia toolkit

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

#### Install CudNN

Download cudnn-8.0-linux-x64-v5.1.tgz from [Nvidia](https://developer.nvidia.com/rdp/cudnn-download)
You need to be registered for Nvidia dev programm.

Copy localy downloaded file to AWS instance

`scp ~/Downloads/cudnn-8.0-linux-x64-v5.1.tgz ubuntu@[AWS_INSTANCE_IP]:~/`

Then installation is as simple as extracting and moving a few files.

```
cd ~/
tar -xzf cudnn-8.0-linux-x64-v5.1.tgz
sudo mv cuda /usr/local/
sudo chmod a+r /usr/local/cuda/include/cudnn.h /usr/local/cuda/lib64/libcudnn*
```

Add these lines to `~/.bashrc`

```
export LD_LIBRARY_PATH="$LD_LIBRARY_PATH:/usr/local/cuda/lib64"
export CUDA_HOME=/usr/local/cuda
```

#### [TODO] Install tensorflow from source for anaconda with GPU support

```
cd ~
git clone https://github.com/tensorflow/tensorflow
cd tensorflow
./configure
```
Specify required config parameters

### Note
#### This is WIP, TODO sections are not tested and all install is not tested ATM

