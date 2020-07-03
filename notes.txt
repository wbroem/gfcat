[see: https://medium.com/@kmshannon/up-and-running-with-aws-ec2-s3-linux-and-deep-learning-c3c17c7b579c]

[THE FOLLOWING IS WRONG: copy the correct URL from the EC2 console]
ssh -i "[pem]" ubuntu@ec2-3-15-12-120.us-east-2.compute.amazonaws.com

sudo apt update && sudo apt upgrade
[will have to click through a few things]

wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh
bash Miniconda3-latest-Linux-x86_64.sh
[then confirm license and location]
[close and reopen shell]

ssh -i "[pem]" ubuntu@ec2-3-15-12-120.us-east-2.compute.amazonaws.com

conda install ipython -y
conda install astropy -y
conda install pandas -y
conda install scipy -y
conda install numpy -y
conda install matplotlib -y
conda install photutils -c astropy -y
conda install -c conda-forge awscli -y
pip install tables

aws configure
[input AWS root username and key]
aws s3 ls

lsblk
[find the EBS volume... maybe xvdb]
sudo file -s /dev/xvdb
sudo mkfs -t ext4 /dev/xvdb
sudo mkdir datadir
sudo mount /dev/xvdb datadir
sudo chown -R ubuntu:ubuntu datadir

[from local computer]
scp -i "[pem]" -r code ubuntu@ec2-3-15-192-205.us-east-2.compute.amazonaws.com:~/.
[copy local code to EC2 instance]
ls

cd code
[run code in python]
ipython
import gfcat as gf
gf.recalibrate(25575,'FUV',data_directory='/home/ubuntu/datadir/')

[get the photometry data and qa images from s3]
aws s3 sync s3://gfcat-test/data/ datadir/. --exclude '*scst*' --exclude '*mov*' --exclude '*h5' --exclude '*raw6*'

[get just the variable qa plots]
aws s3 sync s3://gfcat-test/data/ datadir/. --exclude '*fits*' --exclude '*csv*' --exclude '*Low*' --exclude '*No*' --exclude '*cnt*' --exclude '*h5*' --exclude '*ERROR*'

[move the data locally... maybe change this to targz first?]
scp -i "[pem]" -r ssh -i "[pem]" ubuntu@ec2-3-15-12-120.us-east-2.compute.amazonaws.com:~/datadir/* /Volumes/BigDataDisk/gPhotonData/GTDS/.

####
import numpy as np
import gfcat as gf
def foo(): 
    for eclipse in np.arange(20500,21000): 
        for band in ['FUV','NUV']: 
            %time gf.recalibrate(eclipse,band,data_directory='/home/ubuntu/datadir/')
%time foo()


In [4]: for eclipse in np.arange(10000,15000): 
   ...:     for band in ['FUV','NUV']: 
      ...:         %time gf.recalibrate(eclipse,band,data_directory='/home/ubuntu/datadir/',rerun=True) 

