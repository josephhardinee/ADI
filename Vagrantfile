# _*_ mode: ruby _*_
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"


$scriptfirst = <<SCRIPT
cat >> /etc/yum.repos.d/ARM.repo << EOF
[ARMP6]
name=ARM Production RH6 Repository
baseurl=http://yum.arm.gov/prod6
gpgcheck=0
EOF

SCRIPT

$scriptsecond = <<SCRIPT
cat >> /etc/ld.so.conf.d/adi_libs.conf << EOF
/usr/lib64
/apps/ds/lib64
/apps/ds/lib
EOF

SCRIPT


$script = <<SCRIPT

cat >> ~/.bashrc << EOF

export ADI_HOME=/home/vagrant/adi_home
export SHARE_HOME=/vagrant

export PATH=/apps/base/python2.7/bin:\\$PATH:\\$HOME/bin:/apps/base/bin:/apps/ds/bin:/apps/bin:/apps/tool/bin:\\$HOME/dev/process/vap/bin:/usr/lib64:/apps/ds/lib64
export DSDB_DATA=/apps/ds
export PERLLIB=/apps/ds/lib
export DSDB_HOME=/apps/ds
export VAP_HOME=\\$ADI_HOME/dev/vap

export DATA_HOME=\\$SHARE_HOME/data
export DATASTREAM_DATA=\\$DATA_HOME/adi_example1/datastream
export CONF_DATA=\\$DATA_HOME/adi_example1/conf
export LOGS_DATA=\\$DATA_HOME/adi_example1/logs

export CPATH=\\$CPATH:/apps/ds/include
export LD_LIBRARY_PATH=$:/apps/ds/lib64
export PATH=\\$PATH:/apps/base/python2.7/bin:\\$ADI_HOME/bin:/apps/base/bin:/apps/ds/bin:/apps/bin:/apps/tool/bin:\\$ADI_HOME/dev/process/vap/bin:/usr/lib64
export PATH=/home/vagrant/anaconda2/bin:\\$PATH:\\$VAP_HOME/bin
EOF

source ~/.bashrc

wget -N https://engineering.arm.gov/~whao/adi_vm_home.tar.gz
tar -xzvf adi_vm_home.tar.gz
rm adi_vm_home.tar.gz
mv ~/adi_home/data /vagrant/data
mkdir -p /vagrant/data/db/sqlite
rm ~/adi_home/env_vars_bash
rm ~/adi_home/env_vars_bash_linux
cp /apps/ds/conf/sqlite/20151006.185806.dsdb.sqlite /vagrant/data/db/sqlite/dsdb.sqlite

cat > ~/.db_connect << EOF
dsdb_data sqlite  /vagrant/data/db/sqlite/dsdb.sqlite
dsdb_read sqlite /vagrant/data/db/sqlite/dsdb.sqlite
EOF

cd ~/adi_home/dev/vap/src/adi_example1/process_dod_defs
db_import_process -a dsdb_data -fmt json adi_example1.json
db_load_dod -a dsdb_data cpc.json
db_load_dod -a dsdb_data met.json
cd ../
cp linux_makefile Makefile
make clean
make
cd ~/

wget -N -r -O adi-python.tar https://engineering.arm.gov/~gaustad/adi-python-1.1.tar
tar -xvf adi-python.tar
rm adi-python.tar

wget -N -O anaconda.sh https://3230d63b5fc54e62148e-c95ac804525aac4b6dba79b00b39d1d3.ssl.cf1.rackcdn.com/Anaconda2-2.4.1-Linux-x86_64.sh
bash anaconda.sh -b
rm anaconda.sh

export PATH=/home/vagrant/anaconda2/bin:$PATH
cd ~/py_lib
python setup.py build_ext -L/apps/ds/lib64 
python setup.py install --user

SCRIPT




Vagrant.configure(2) do |config|

  config.vm.box = "bento/centos-6.7"
  config.vm.synced_folder ".", "/vagrant", type: "nfs"
  config.ssh.insert_key = false
  config.ssh.private_key_path = ["~/.ssh/id_rsa","~/.vagrant.d/insecure_private_key"]
  config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination:"~/.ssh/authorized_keys"
  config.ssh.forward_agent = true

  config.vm.network "private_network", ip: "192.168.120.120"
  config.vm.provision :shell, inline: $scriptfirst

  config.vm.provision :shell, inline: "sudo yum -y update"
  config.vm.provision :shell, inline: "sudo yum -y install epel-release vim"
  config.vm.provision :shell, inline: "sudo yum -y groupinstall adi6"
  config.vm.provision :shell, inline: "sudo yum -y install kernel-devel"
  config.vm.provision :shell, inline: "sudo yum -y install netcdf-devel dsdb-sqlite dsdb-python_lib unzip udunits2 man afl-libcds3"

  config.vm.provision :shell, inline: $scriptsecond 

  config.vm.provision :shell, inline: "sudo ldconfig"
  config.vm.provision :shell, inline: "sudo ln -s /opt/VBoxGuestAdditions-4.3.10/lib/VBoxGuestAdditions /usr/lib/VBoxGuestAdditions"
  config.vm.provision :shell, inline: $script, privileged: false
end
