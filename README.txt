You can follow the steps below to do a manual installation on linux. This procedure assumes that you start from
your home directory and are happy with having directories created there. The sed command below attempts to modify
~/.sigmakee/KBs/config.xml to conform to your local paths.  If your paths differ, then you may need to
edit your config.xml manually. If you are running tomcat on vagrant or another VM, you may need to change the port
value from 8080 .  E will only work if your $TMPDIR is set correctly.  No particular version of tomcat is required.
If you load a different version of tomcat, be sure to change $CATALINA_HOME and your paths to conform to the version.
If you use a different mirror or version you'll need to change the wget commend below. Change "theuser" below to your
user name.

-----------------
System preparation on Linux

# create user theuser
  useradd theuser

# add password for theuser
  passwd theuser

# add to sudoers file
  usermod -aG sudo theuser

# install unzip
  sudo apt-get install unzip

cd KBs
cp ~/workspace/sigmakee/config.xml .

sudo apt-get update

# may need to create a .bashrc
touch .bashrc

# handy to add stuff to .bashrc
echo "alias dir='ls --color=auto --format=vertical -la'" >> .bashrc
echo "export HISTSIZE=10000 HISTFILESIZE=100000" >> .bashrc
echo "export JAVA_HOME=/home/theuser/Programs/jdk1.8.0_112" >> .bashrc
echo "export PATH=$PATH:$JAVA_HOME/bin" >> .bashrc

# You may need to download Java and set your
# JAVA_HOME http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html .
# The following command line version may work
wget --no-check-certificate --no-cookies --header "Cookie: oraclelicense=accept-securebackup-cookie"
  http://download.oracle.com/otn-pub/java/jdk/8u112-b15/jdk-8u112-linux-x64.tar.gz

-----------------------------

mkdir workspace
mkdir Programs
cd Programs
wget 'http://nlp.stanford.edu/software/stanford-corenlp-full-2015-12-09.zip'
wget 'http://www-us.apache.org/dist/tomcat/tomcat-8/v8.5.15/bin/apache-tomcat-8.5.15.zip'
wget 'http://wordnetcode.princeton.edu/3.0/WordNet-3.0.tar.gz'
wget 'http://wwwlehre.dhbw-stuttgart.de/~sschulz/WORK/E_DOWNLOAD/V_1.9/E.tgz'
tar -xvzf E.tgz
unzip apache-tomcat-8.5.15.zip
rm apache-tomcat-8.5.15.zip
cd ~/Programs/apache-tomcat-8.5.15/bin
chmod 777 *
cd ~/Programs
unzip stanford-corenlp-full-2015-12-09.zip
rm stanford-corenlp-full-2015-12-09.zip
cd ~/Programs/stanford-corenlp-full-2015-12-09/
unzip stanford-corenlp-3.6.0-models.jar
cd ~/workspace/
sudo apt-get install git
git clone https://github.com/ontologyportal/sigmakee
git clone https://github.com/ontologyportal/sumo
cd ~
mkdir .sigmakee
cd .sigmakee
mkdir KBs
cp -R ~/workspace/sumo/* KBs
me="$(whoami)"
cp ~/workspace/sigmakee/config.xml ~/.sigmakee/KBs
sed -i "s/theuser/$me/g" config.xml
cd ~/Programs
gunzip WordNet-3.0.tar.gz
tar -xvf WordNet-3.0.tar
cp WordNet-3.0/dict/* ~/.sigmakee/KBs/WordNetMappings/
cd ~/Programs/E
sudo apt-get install make
sudo apt-get install gcc
./configure
make
make install
cd ~
echo "export SIGMA_HOME=~/.sigmakee" >> .bashrc
echo "export CATALINA_OPTS=\"$CATALINA_OPTS -Xms500M -Xmx2500M\"" >> .bashrc
echo "export CATALINA_HOME=~/Programs/apache-tomcat-8.5.15" >> .bashrc
source .bashrc
cd ~/workspace/sigmakee
sudo add-apt-repository universe
sudo apt-get update
sudo apt-get install ant
ant
cd ~/Programs/stanford-corenlp-full-2015-12-09/

# to test run

java  -Xmx2500m -classpath ~/workspace/sigmakee/build/classes:/home/theuser/workspace/sigmakee/build/lib/*
  com.articulate.sigma.KB


# Start Tomcat with
$CATALINA_HOME/bin/startup.sh

# point your browser at http://localhost:8080/sigma/login.html


Debugging
- If login.html redirects you to init.jsp that means the system is still initializing. Wait a minute or two and try
again.
- If you are repeatedly getting 404s, check the port value in ~/.sigmakee/KBs/config.xml. 8080 for local,
9090 for Vagrant
- If you are on mac and getting errors related to not finding jars when running com.articulate.sigma.KB, copy all jars
from /home/theuser/workspace/sigmakee/build/lib/ to /Library/Java/Extensions

---------------------------------------------------------------------
jUnit testing on the command line:

java  -Xmx2500m -classpath
  /home/theuser/workspace/sigmanlp/build/classes:/home/theuser/workspace/sigmanlp/build/lib/*
  org.junit.runner.JUnitCore com.articulate.sigma.UnitTestSuite

---------------------------------------------------------------------
The installation approaches below may be out of date

Install via script from source on Linux or Mac OS with
bash <(curl -L https://raw.githubusercontent.com/ontologyportal/sigmakee/master/install.sh)

Note that you need to enter the entire statement above, including calling "bash".

Users should also see

https://sourceforge.net/p/sigmakee/wiki/required_data_files/
Mac instructions - https://sourceforge.net/p/sigmakee/wiki/Sigma%20Setup%20on%20Mac/ 
Ubuntu - https://sourceforge.net/p/sigmakee/wiki/Setting%20up%20Sigma%20on%20Ubuntu/

You can also install Sigma on a Vagrant virtual machine.  You'll need VirtualBox too
https://www.virtualbox.org/

> mkdir sigma_vagrant
> cd sigma_vagrant
> wget https://raw.githubusercontent.com/ontologyportal/sigmakee/master/Vagrantfile
> vagrant up
> vagrant ssh
> bash <(curl -L https://raw.githubusercontent.com/ontologyportal/sigmakee/master/install-vagrant.sh)

follow the prompts and Sigma will be running.  Then on the browser of your host machine, go to
http://localhost:9090/sigma/login.html

To run natural language interpretation from the command line in the virtual machine,
run the following additional steps

> cd ~
> mkdir Programs
> cd Programs
> cp /vagrant/Downloads/stanford-corenlp-full-2015-12-09.zip .
> unzip stanford-corenlp-full-2015-12-09.zip
> cd stanford-corenlp-full-2015-12-09
> export SIGMA_HOME="/home/vagrant/.sigmakee"
> jar -xf stanford-corenlp-3.6.0-models.jar
> cd ~/Programs/stanford-corenlp-full-2015-12-09
> java  -Xmx2500m -classpath  /home/vagrant/workspace/sigma/sigma/build/classes:/home/vagrant/workspace/sigma/sigma/build/lib/*  com.articulate.sigma.semRewrite.Interpreter -i


