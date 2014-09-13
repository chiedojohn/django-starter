VAGRANTFILE_API_VERSION = "2"

# These are the scripts that will be run by the terminal upon creation of a new machine.
# The answer 'yes' is piped into the commands that require 'Y' as user input
$script = <<SCRIPT
sudo debconf-set-selections <<< 'mysql-server-5.5 mysql-server/root_password password root'
sudo debconf-set-selections <<< 'mysql-server-5.5 mysql-server/root_password_again password root'
sudo apt-get update
sudo apt-get -y install mysql-server-5.5
yes | sudo apt-get install libpq-dev
yes | sudo apt-get install nodejs nodejs-legacy npm
yes | sudo apt-get install python2.6 python2.6-dev
yes | sudo apt-get install python-pip build-essential 
yes | sudo pip install --upgrade pip
yes | sudo pip install django==1.6
yes | sudo apt-get install git
yes | sudo apt-get install vim-nox
yes | sudo apt-get install sqlite3 libsqlite3-dev
sudo update-rc.d mysql defaults

# sets up mysql server
if [ ! -f /var/log/databasesetup ];
then
    echo "CREATE DATABASE django_app_default" | mysql -uroot -proot

    touch /var/log/databasesetup
    echo "CREATE USER 'root'@'%' IDENTIFIED BY 'root';" | mysql -uroot -proot
    #make mysql listen to connections from the outside
    echo "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;" | mysql -uroot -proot
    echo "FLUSH PRIVILEGES;" | mysql -uroot -proot
    sudo sed -i 's/127.0.0.1/0.0.0.0/g' /etc/mysql/my.cnf
    sudo service mysql restart

    if [ -f /vagrant/data/initial.sql ];
    then
        mysql -uroot -proot rails_app_development < /vagrant/data/initial.sql
    fi
fi

# This sets up elastic beanstalk
if [ ! -f /var/log/ebsetup ];
then
  cd
  wget https://s3.amazonaws.com/elasticbeanstalk/cli/AWS-ElasticBeanstalk-CLI-2.6.3.zip
  unzip AWS-ElasticBeanstalk-CLI-2.6.3.zip
  rm AWS-ElasticBeanstalk-CLI-2.6.3.zip
  echo 'export PATH="$PATH:/home/vagrant/AWS-ElasticBeanstalk-CLI-2.6.3/eb/linux/python2.7"' | sudo tee -a /home/vagrant/.bashrc
  touch /var/log/ebsetup
fi

# This sets up dev environVment variables
if [ ! -f /var/log/devenv ];
then
  cd
  echo 'export FLASK_DEV_ENV="true"' | sudo tee -a /home/vagrant/.bashrc
  touch /var/log/devenv
fi


SCRIPT


Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "ubuntu/trusty64"

  # forward the python runserver port
  config.vm.network "forwarded_port", guest: 8000, host: 3001
  # forward postgresql
  config.vm.network "forwarded_port", guest: 3306, host: 5433

  # run the script from above
  config.vm.provision "shell", inline: $script
  config.vm.synced_folder ".", "/vagrant", :mount_options => ['dmode=777,fmode=666']
end