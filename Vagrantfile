# Welcome! Thanks for taking an interest in contributing to Alaveteli.
# This Vagrantfile creates a VM with an Alaveteli production install set up
# to receive deployments from Capistrano.
#
# See http://alaveteli.org/docs/installing/deploy/ for more info on deploying
# Alaveteli with Capistrano.
#
# Usage
# =====
#
# TODO
#
# Customizing the Vagrant instance
# ================================
#
# This Vagrantfile allows customisation of some aspects of the virtaual machine
# See the customization options below for details.
#
# The options can be set either by prefixing the vagrant command, or by
# exporting to the environment.
#
#   # Prefixing the command
#   $ ALAVETELI_CAP_VAGRANT_MEMORY=2048 vagrant up
#
#   # Exporting to the environment
#   $ export ALAVETELI_CAP_VAGRANT_MEMORY=2048
#   $ vagrant up
#
# Both have the same effect, but exporting will retain the variable for the
# duration of your shell session.
#
# Customization Options
# =====================
ALAVETELI_IP = ENV['ALAVETELI_CAP_VAGRANT_IP'] || "192.168.33.155"
ALAVETELI_MEMORY = ENV['ALAVETELI_CAP_VAGRANT_MEMORY'] || 1536
ALAVETELI_OS = ENV['ALAVETELI_VAGRANT_OS'] || 'wheezy64'

SUPPORTED_OPERATING_SYSTEMS = {
  'precise64' => 'https://cloud-images.ubuntu.com/vagrant/precise/current/precise-server-cloudimg-amd64-vagrant-disk1.box',
  'squeeze64' => 'http://puppet-vagrant-boxes.puppetlabs.com/debian-607-x64-vbox4210-nocm.box',
  'wheezy64' => 'http://puppet-vagrant-boxes.puppetlabs.com/debian-73-x64-virtualbox-nocm.box'
}

def box
  ALAVETELI_OS
end

def box_url
  SUPPORTED_OPERATING_SYSTEMS[ALAVETELI_OS]
end

$cap_setup = <<-EOF
mv /var/www/alaveteli/alaveteli /home/vagrant/
mkdir -p /var/www/alaveteli/alaveteli/shared
pushd /home/vagrant/alaveteli
  cp    config/{general,database,newrelic}.yml /var/www/alaveteli/alaveteli/shared/
  cp    config/rails_env.rb /var/www/alaveteli/alaveteli/shared/
  cp -r cache /var/www/alaveteli/alaveteli/shared/
  cp -r files /var/www/alaveteli/alaveteli/shared/
  cp -r log /var/www/alaveteli/alaveteli/shared/
  cp -r lib/acts_as_xapian/xapiandbs /var/www/alaveteli/alaveteli/shared/
  sudo -u alaveteli sed -i '/^SHARED_DIRECTORIES:/a\ \ - lib/themes' shared/general.yml
popd
chown -R alaveteli:alaveteli /var/www/alaveteli/alaveteli
chmod -R 755 /var/www/alaveteli/alaveteli

old_path='/var/www/alaveteli/alaveteli'
new_path='/var/www/alaveteli/alaveteli/current'
sed -i "s%$old_path%$new_path%g" /etc/nginx/sites-available/default
sed -i "s%$old_path%$new_path%g" /etc/cron.d/alaveteli
sed -i "s%$old_path%$new_path%g" /etc/postfix/master.cf
sed -i "s%$old_path%$new_path%g" /etc/init.d/alaveteli-alert-tracks
sed -i "s%$old_path%$new_path%g" /etc/init.d/alaveteli

postfix stop && postfix start
rm -rf /var/www/alaveteli/alaveteli/current
EOF

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = box
  config.vm.box_url = box_url
  config.vm.network :private_network, :ip => ALAVETELI_IP

  config.ssh.forward_agent = true

  # The bundle install fails unless you have quite a large amount of
  # memory; insist on 1.5GiB:
  config.vm.provider "virtualbox" do |vb|
    host = RbConfig::CONFIG['host_os']
    # Give VM access to all cpu cores on the host
    if host =~ /darwin/
      cpus = `sysctl -n hw.ncpu`.to_i
    elsif host =~ /linux/
      cpus = `nproc`.to_i
    else # sorry Windows folks, I can't help you
      cpus = 1
    end

    vb.customize ["modifyvm", :id, "--memory", ALAVETELI_MEMORY]
    vb.customize ["modifyvm", :id, "--cpus", cpus]
  end

  # Fetch and run the install script:
  config.vm.provision :shell, :inline => "apt-get -y install curl"
  config.vm.provision :shell, :inline => "curl -O https://raw.githubusercontent.com/mysociety/commonlib/master/bin/install-site.sh"
  config.vm.provision :shell, :inline => "chmod a+rx install-site.sh"
  config.vm.provision :shell, :inline => "./install-site.sh " \
                                             "--default " \
                                             "alaveteli " \
                                             "alaveteli " \
                                             "#{ ALAVETELI_IP }"
  config.vm.provision :shell, :inline => 'echo "alaveteli:secret" | chpasswd'
  config.vm.provision :shell, :inline => $cap_setup
end
