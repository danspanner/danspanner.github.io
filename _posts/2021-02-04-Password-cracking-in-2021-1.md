---
layout: post
title: Password cracking in 2021 part one
---

Recently at work there was a Windows Server that noone appeared to have the password for. What better chance to try out some good old fashioned hacking?

Well, old fashioned was definitely the word for. A lot of the documentation out there for cracking Windows passwords is very outdated. After a frustrating couple of hours trying to 
1. Get the password hash
2. Install NVIDIA drivers and CUDA SDK
3. Compile John the Ripper to enable OpenCL cracking
I finally got the right combination of tools, technology and stars to align and setup my very own password cracking rig.

I'm pretty lucky at work- someone else has already gone through the hard yards and got PCI passthrough enabled. So there wasn't a lot of work I had to do in order to get an instance running on Openstack with PCI passthrough with an image of Ubuntu 18.04. However, none of the glance images we use have the drivers installed, that's all left to the user.

On to step one- getting an instance up and running with a working GPU.

### NVIDIA drivers

First, I found what sort of card I was dealing with-
```
lspci -Qknn | grep -i nvidia
```
which told me I was dealing with a Tesla V100 SXM2 32GB. Next step was getting the driver source, which was available from [NVIDIAs website](https://www.nvidia.com/Download/index.asp). In Linux the driver install is provided as a `.run` file. It does have a few pre-requisites before you can install though.
```
sudo apt install gcc make autoconf
```
Then it's just a matter of getting correct driver- after clicking through the selection for your driver you can wget it or download + scp it up to your server. 
```
wget https://us.download.nvidia.com/tesla/460.32.03/NVIDIA-Linux-x86_64-460.32.03.run
```
then install the driver
```
chmod +x NVIDIA-Linux-x86_64-460.32.03.run
sudo sh NVIDIA-Linux-x86_64-460.32.03.run
```

The NVIDIA documentation actually had a pretty good guide to installing both the driver and the [CUDA Toolkit](https://developer.nvidia.com/cuda-downloads), the CUDA Toolkit install in particular is super easy, follow the guide and you can't go wrong. For me it was a case of 
```
wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1804/x86_64/cuda-ubuntu1804.pin
sudo mv cuda-1804.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/11.2.0/local_installers/cuda-repo-ubuntu1804-11-2-local_11.2.0-460.27.04.1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1804-11-2-local_11.2.0-460.27.04.1_amd64.deb
```
at this point Ubuntu complained about a missing repo key, that's fine, it told me how to add it-
```
sudo apt-key add /var/cuda-repo-ubuntu1804-11-2-local/7fa2af80.pub
```
and then
```
sudo apt update
sudo apt install cuda
```

The main issue I ran into with the driver install was by following outdated documentation- if you follow the steps in the NVIDIA documentation for the _latest_ version of your driver, odds are you will succeed. 

### JtR

The next phase was [John the Ripper[(https://www.openwall.com/john)].

Again, I stumbled across some old info and ended up getting stuck on the install- once I had the most up-to-date code, it worked just fine.

Firstly, grab the bleeding edge code for jumbo-
```
wget https://github.com/openwall/john/archive/bleeding-jumbo.tar.gz
tar -xvzf bleeding-jumbo.tar.gz
```
All the documentation you need is included in the `doc` folder, after a quick read of `README-OPENCL` I had all the info I needed. First, I jumped in the `src` directory
```
cd src
```
Then ran `configure` with the flags provided with the documentation
```
./configure LDFLAGS=-L/usr/local/cuda-11.0/targets/x86_64-linux/lib CPPFLAGS=-I/usr/local/cuda-11.0/targets/x86_64-linux/include
```
Which of course failed, because I had a newer version of CUDA installed. Once again, with feeling
```
./configure LDFLAGS=-L/usr/local/cuda-11.2/targets/x86_64-linux/lib CPPFLAGS=-I/usr/local/cuda-11.2/targets/x86_64-linux/include
```
This worked fine, and I was able to build a binary with
```
make -s clean && make -sj4
```
The binary itself ends up in the `run` directory. If we cd to that directory we can confirm it's working with
```
./john --list=formats
```
And you should see a few different formats with the `-opencl` at the end. 

At this point, my cracking rig was done and dusted. Good to go. All I needed now was the password hash, which is a different tale entirely, which I will reserve for another post.