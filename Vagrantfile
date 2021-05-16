# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "centos/7"

  config.vm.hostname = "node"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  config.vm.network "public_network", :type => "bridge",
  use_dhcp_assigned_default_route: true

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
  
    vb.name = "node"

    vb.cpus = 4
    vb.memory = 24576
    # vb.customize ["resources", :id, "--memory", 24576]
    # vb.customize ["resources", :id, "--cpus", 4]
  end
  
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.


  # ssh-key-copy
  config.vm.provision "shell" do |s|
    ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
    s.inline = <<-SHELL
      echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
    SHELL
  end

  # Install (pre)requisites
  config.vm.provision "shell", inline: <<-SHELL
    
    systemctl stop firewalld && systemctl disable firewalld
    setenforce 0
    sed -i --follow-symlink 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux
    modprobe br_netfilter
    echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
    swapoff -a
    sed -i -e "$(sudo grep -n -m 1 swap /etc/fstab | awk -F ':' '{print $1}')d" /etc/fstab

    yum update -y
    yum upgrade -y
    yum install -y vim wget git net-tools epel-release yum-utils
    yum install -y device-mapper-persistent-data lvm2
    
    yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
    yum install -y docker-ce-18.09-1-3

    groupadd docker
    usermod -aG docker vagrant

    cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]

name=Kubernetes

baseurl=https://packages.cloud.google.com/yum/repos/kubernetes-el7-x86_64

enabled=1

gpgcheck=1

repo_gpgcheck=1

gpgkey=https://packages.cloud.google.com/yum/doc/yum-key.gpg

       https://packages.cloud.google.com/yum/doc/rpm-package-key.gpg

EOF
  yum install -y kubernetes-cni-0.7.5-0
  
  yum install -y kubelet-1.14.1-0 kubeadm-1.14.1-0 kubectl-1.14.1-0
  systemctl start docker && systemctl enable docker
  systemctl start kubelet && systemctl enable kubelet
  systemctl daemon-reload
  echo '1' > /proc/sys/net/bridge/bridge-nf-call-iptables
  echo '1' > /proc/sys/net/bridge/bridge-nf-call-ip6tables
  
  echo 'KUBELET_EXTRA_ARGS=--cgroup-driver=cgroupfs' > /etc/sysconfig/kubelet

  echo '1'> /proc/sys/net/bridge/bridge-nf-call-iptables
  echo '1'> /proc/sys/net/ipv4/ip_forward

  SHELL


end
