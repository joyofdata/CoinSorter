1.) I(Paul Krush) started with this, by Raffael Vogler:
GPU Powered DeepLearning with NVIDIA DIGITS on EC2:
http://www.joyofdata.de/blog/gpu-powered-deeplearning-with-nvidia-digits/

2.) I rebuilt for Digits 2.

3.) I then rebuilt for cudnn3 and Nidvia Caffe v0.13.0 
	This one took me about 2 hours 

I think Nivida is saying they have an web-installer that does better. I don't see how?

I use ModaXterm(Free) instead of Putty because there was some sort of issue with 
	DIGITS needing an X-Server. Login: ubuntu

Starting with AMI:
Unbuntu 14.04 (Near the top of the list) 
g2.2xlarge or g2.8xlarge
*************************************************************************
#This is written as a script, but I don't see how you could run it as one...
#There is just too many things that change and can go wrong. 
#I just copy and paste one or more lines at a time. 
#But I am just a beginner unix user...

#To further speed up deep learning relevant calculations 
#it is a good idea to set up the cuDNN library.
#For that purpose you will have to get an NVIDIA developer account
#and join the CUDA registered developer program. 
#The last step requires NVIDIA to unlock your account 
#and that might take hours or 1-2 days.
#But you can get started also without cuDNN library. 
#As soon as you have the okay from them � download cuDNN and upload it to your instance.

#Paul's Comment:  If you going through this much trouble, 
#I would just register as a developer and download CuDnn 3 first. 




#FTP Install Notes
#FTP is not required, but I think it's handy. 
#There are all sorts of examples of using vsftpd on AWS, but they all a bit different. 
#This is what works for me, just to get things started. 
#This is just for temporary instance that only I would be using. 

#AWS Security group changes: 
#Make sure 20-21 and 1024-1048 is open to your IP
cd ~
sudo apt-get update
sudo apt-get install vsftpd

sudo vi /etc/vsftpd.conf
#Add the following lines to the bottom of the vsftpd.conf file:
#Change pasv_address to the Public IP of the AWS Instance:
pasv_address=54.157.147.8
pasv_enable=YES
pasv_min_port=1024
pasv_max_port=1048
port_enable=YES
pasv_addr_resolve=YES
write_enable=YES

#Change it to ubuntu password to something: 
sudo passwd ubuntu
sudo service vsftpd restart
cd ~


# don't forget to get your system up to date
sudo apt-get update
sudo apt-get dist-upgrade

#Installing CUDA 7
#Installation of required tools
#A menu pops up, and I just tell it to keep the local version:
sudo apt-get install -y gcc g++ gfortran build-essential \
  git wget linux-image-generic libopenblas-dev python-dev \
  python-pip python-nose python-numpy python-scipy
 
# downloading CUDA 7
sudo wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1404/x86_64/cuda-repo-ubuntu1404_7.0-28_amd64.deb

# installing CUDA
sudo dpkg -i cuda-repo-ubuntu1404_7.0-28_amd64.deb
 
sudo apt-get update
sudo apt-get install cuda
 
# setting the environment variables so CUDA will be found
echo -e "\nexport PATH=/usr/local/cuda/bin:$PATH" >> .bashrc
echo -e "\nexport LD_LIBRARY_PATH=/usr/local/cuda/lib64" >> .bashrc
 
sudo reboot
 
# installing the samples and checking the GPU
cuda-install-samples-7.0.sh ~/
cd NVIDIA\_CUDA-7.0\_Samples/1\_Utilities/deviceQuery  
make  
./deviceQuery



# Install CuDNN v 3
#You can't just get the file: sudo wget https://developer.nvidia.com/rdp/assets/cudnn-7.0-linux-x64-v3-rc
#You have to register as a developer and manually download. 
#This is why FTP was the step before this. 
#So first FTP the cudnn-7.0-linux-x64-v3.0-rc.tgz file to home

gzip -d cudnn-7.0-linux-x64-v3.0-rc.tgz
tar xf cudnn-7.0-linux-x64-v3.0-rc.tar

# copy the library files into CUDA's include and lib folders
sudo cp cuda/include/cudnn.h /usr/local/cuda-7.0/include
sudo cp cuda/lib64/libcudnn* /usr/local/cuda-7.0/lib64


#Installing caffe
#Main source for this and the following step is the readme of the DIGITS project.
sudo apt-get install libprotobuf-dev libleveldb-dev \
  libsnappy-dev libopencv-dev libboost-all-dev libhdf5-serial-dev \
  libgflags-dev libgoogle-glog-dev liblmdb-dev protobuf-compiler \
  libatlas-base-dev
 
sudo apt-get install python-dev python-pip python-numpy gfortran

#Install Git:
sudo apt-get install git

# the version number of the required branch might change
# consult https://github.com/NVIDIA/DIGITS/blob/master/README.md
git clone --branch v0.13.0 https://github.com/NVIDIA/caffe.git   
 
#This takes a long time,(20-30+ minutes) 
cd ~/caffe/python
for req in $(cat requirements.txt); do sudo pip install $req; done
 
cd ~/caffe
cp Makefile.config.example Makefile.config
 
# In Makefile.config check that USE_CUDNN is set to 1 in case you would
# like to use it and to 0 if not
# It defaults to 1 

make all
make py
make pycaffe
make test
make runtest

echo -e "\nexport CAFFE_HOME=/home/ubuntu/caffe" >> ~/.bashrc
# load the new environmental variables
bash

Installing DIGITS
cd ~
git clone https://github.com/NVIDIA/DIGITS.git digits   
cd digits
sudo apt-get install graphviz gunicorn
for req in $(cat requirements.txt); do sudo pip install $req; done

###I added this line because the pycaffe module was not being found in Digits:
###OK, I took this out and added:  make pycaffe
###echo 'export PYTHONPATH=/home/ubuntu/caffe/python' >> ~/.bashrc 
###bash 


cd ~

# You might want to consider locating your job-directory ( jobs_dir) on an EBS 
# See https://github.com/NVIDIA/DIGITS/blob/master/docs/GettingStarted.md for details. 
# I set the the jobs_dir at /data/digits/jobs by using the --config option:
#./digits-devserver --config


# start the server
./digits/digits-devserver

#AWS Security group changes: 
#Make sure port 5000 is open to your IP

#To view DIGITS go to port 5000 the public IP of your instance:
#Example:  I pasted 50.16.43.125:5000 in my browser. 


#Mounting a new EBS Volume
#WARNING! Only format a new volume:
#sudo mkfs -t ext4 xvdf
#Create a mount point:
#sudo mkdir /data
#Mount the new volume
#sudo mount /dev/xvdf /data
#Add permissions:
#sudo chmod 777 /data









