# nlmixr2_on_AWS
In this README file we will describe how to install R, RStudio server and the rxode2/nlmixr2 packages on a ubuntu server hosted on AWS.

## Create an AWS account and set up the ubuntu server instance

The first step is to create an AWS account. This can easily be done following [these](https://aws.amazon.com/premiumsupport/knowledge-center/create-and-activate-aws-account/?nc1=h_ls) instructions.

In this guideline, RStudio server will be installed on an Ubuntu server hosted on an AWS EC2 instance. Instruction for setting up a Linux AWS EC2 instance can be found [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html).

>**Note**  
> EC2 instances are associated to the server's regions, that you can find on the top right of EC2 dashboard page. You can decide the region you prefer.

Briefly, 
1. Once you have created your AWS account, go to the [AWS EC2 dashboard page](https://console.aws.amazon.com/ec2/).
2. Press `Launch an instance`.
3. In `Name and Tags` tab write the instance name.
4. In `Application and OS Images (Amazon Machine Image, AMI)`, `Quick Start` tab, select the default Ubuntu AMI (on 16/01/2023 we selected *Ubuntu Server 22.04 LTS (HVM), SSD Volume Type*). Leave the other options as default.
5. In the `Instance type` tab, we selected the `t2.large instance`. We found that the free tier eligible t2.micro instance was "too slow" for our purposes.
6. In the `Key pair` tab, you can select a key that you have already created in the past. If you don't have a key pair, you can create a new one by pressing `Create new key pair`. A new tab will then open. We selected RSA as Key pair type and `.ppk` as private key file format (this because for connecting through SSH we used PuTTY). Press `Create key pair` and then the key will be downloaded. Store it safely! We will need it later.
7. I network settings, tick `Allow SSH traffic from` `Anywhere 0.0.0.0/0`, `Allow HTTPS traffic from the internet` and `Allow HTTP traffic from the internet`.
  >**Note**  
  By default, RStudio server listens on port `8787`. Later on in this guideline, we will allow RStudio server to listen also on port `80`, which is the HTTP default port on AWS (which is already selected by ticking `Allow HTTP traffic from the internet` in point 7). If we want to access the RStudio server on port `8787` as well, we should define an additional security rule. In network settings, click edit and then `add security group rule`. Select `Custom TCP`, in `Port range` write `8787` and in source select `0.0.0.0/0`.
8. In `Configure storage` we selected 30 GiB of `gp2 General purpose SSD root volume`.
9. Press `Launch instance` and then `View all instances`.

Now, in the list of all your instances you should find the newly created one in the "Running" state.


## Install R and RStudio

Guidelines for installing R and RStudio server on a Ubuntu 22 server can be found [here](https://posit.co/download/rstudio-server/).
For installing R and RStudio we need to access the instance through SSH. For this, we will using PuTTY. You can freely download PuTTY from [here](https://www.putty.org/).
To access the instance throgh SSH:

1. launch PuTTY.
2. In the `Session` tab, under `Host Name (or IP address)` write `ubuntu@X`, where `X` is the public IPv4 DNS address that you can find ticking the instance you want to connect to on the `Instances` page of the EC2 dashboard. Leave port to 22.
3. Open `SSH/Auth` tab and click `Browse` button next to the `Private key file for authentication` field. Search for the key you associated to the previously generated EC2 instance and open it.
4. Click open and then Accept.

Now, you are in your AWS ubuntu server instance!
First of all, let's run

```
sudo apt update
sudo apt upgrade
sudo apt-get install build-essential
```


### Install R 

First, to install the latest version of R on Ubuntu server we should run.

```
# install two helper packages we need
sudo apt install --no-install-recommends software-properties-common dirmngr
```
```
# add the signing key (by Michael Rutter) for these repos
# To verify key, run gpg --show-keys /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc 
# Fingerprint: E298A3A825C0D65DFD57CBB651716619E084DAB9
wget -qO- https://cloud.r-project.org/bin/linux/ubuntu/marutter_pubkey.asc | sudo tee -a /etc/apt/trusted.gpg.d/cran_ubuntu_key.asc
```
```
# add the R 4.0 repo from CRAN -- adjust 'focal' to 'groovy' or 'bionic' as needed
sudo add-apt-repository "deb https://cloud.r-project.org/bin/linux/ubuntu $(lsb_release -cs)-cran40/"
```

Then, let's install R
```
sudo apt install --no-install-recommends r-base
```

To check the R version, run

```
R
R.Version()
quit()
```

On 16/01/2023 R version should be R 4.2.2.

### Install RStudio Server

For installing RStudio please run the following commands.

```
sudo apt-get install gdebi-core
```
```
wget https://download2.rstudio.org/server/jammy/amd64/rstudio-server-2022.12.0-353-amd64.deb
```
```
sudo gdebi rstudio-server-2022.12.0-353-amd64.deb
```

Now, RStudio server should be up and running on port `8787`! If you have set the security group for port `8787`, to access RStudio server you just need to copy and paste the Public IPv4 address (you can find by selecting the instance you want to connect to in the Instances page of EC2 dashboard) followed by `:8787`, `http://X.X.X.X:8787`.  
If you want to open RStudio by directly copy paste the IPv4 address in a new browser tab, we should tell the RStudio server to listen on port `80`.

First we need to add the writing access to the `rserver.conf` file.
```
sudo chmod a+rw /etc/rstudio/rserver.conf
```
Then, we need to add the access to port `80`.
```
echo 'www-port=80' >> /etc/rstudio/rserver.conf
```
Finally, we need to restart the RStudio server.
```
sudo rstudio-server restart
```
Now, RStudio can be accessed from port `80`.

As it is possible to see, the RStudio requires an username and password. By default, all the ubuntu's user are allowed to access the RStudio server.
If you want to setup a new user, you can run the following command through SSH.

```
sudo adduser new_username
```

## Install tidyverse and nlmixr2

Before installing tidyverse and nlmixr2 we need to install a few libraries.
Let's access the instance through SSH and run the following code.

First of all, let's install cmake.
```
sudo apt install make
sudo snap install cmake --classic
```
Then, let's install the following libraries.
```
sudo apt-get install libcurl4-openssl-dev
sudo apt-get install libxml2-dev
sudo apt-get install libmpfr-dev
sudo apt-get install libgmp-dev
sudo apt-get install libboost-all-dev
```

We found the previous libraries necessary for installing nlmixr2.  

Now, let's install `tidyverse`. You can do it both by opening R through SSH or RStudio server.
```
install.packages("tidyverse")
```
Let's install `rxode2`.
```
install.packages("rxode2")
```
Finally we shall install `nlmixr2`.
```
install.packages("nlmixr2", dependencies=T)
```
If you have installed R versions prior to 4.2, please refer to [this page](https://github.com/nlmixr2/nlmixr2/) for nlmixr2 installation. 

Now you should have RStudio running on AWS with both `tidyverse` and `nlmixr2` installed, try this out with the examples at [this page](https://github.com/nlmixr2/nlmixr2/).









