# -*- mode: ruby -*-
# vi: set ft=ruby :

# Environment variables:
#
# SKIP_BCC_BUILD: Set to skip the building bcc from source

$ubuntu_deps = <<EOF
wget https://apt.llvm.org/llvm.sh
bash ./llvm.sh 12
apt-get -qq update
apt-get -qq install linux-headers-$(uname -r) binutils-dev python
apt-get -qq install bison cmake flex g++ git libelf-dev zlib1g-dev libfl-dev systemtap-sdt-dev libclang-12-dev
EOF

$fedora_deps = <<EOF
dnf builddep -q -y bpftrace
dnf install -q -y git
EOF

$build_bcc = <<EOF
if [ -e /usr/local/lib/libbcc.so ]; then
   echo "libbcc already built, skipping"
   exit 0
fi
git clone https://github.com/iovisor/bcc.git
mkdir -p bcc/build
cd bcc/build
git checkout v0.19.0
cmake .. -DCMAKE_BUILD_TYPE=Debug -DCMAKE_INSTALL_PREFIX=/usr/local \
  -DENABLE_EXAMPLES=0 -DENABLE_TESTS=0 -DENABLE_MAN=0 \
  -DENABLE_LLVM_SHARED=1
make && sudo make install && sudo ldconfig
EOF

Vagrant.configure("2") do |config|
  boxes = {
    'ubuntu-16.04'     => {
      'image'          => 'ubuntu/xenial64',
      'scripts'        => [ $ubuntu_deps, ],
    },
    'ubuntu-18.04'     => {
      'image'          => 'ubuntu/bionic64',
      'scripts'        => [ $ubuntu_deps, ],
    },
    'ubuntu-20.04'     => {
      'image'          => 'ubuntu/focal64',
      'scripts'        => [ $ubuntu_deps, ],
      'fix_console'    => 0,
    },
    'fedora-31'        => {
      'image'          => 'fedora/31-cloud-base',
      'scripts'        => [ $fedora_deps, ],
      'skip_bcc_build' => 1
    },
    'fedora-32'        => {
      'image'          => 'fedora/32-cloud-base',
      'scripts'        => [ $fedora_deps, ],
      'skip_bcc_build' => 1
    },
    'fedora-33'        => {
      'image'          => 'fedora/33-cloud-base',
      'scripts'        => [ $fedora_deps, ],
      'skip_bcc_build' => 1
    }
}
  boxes.each do | name, params |
    config.vm.define name do |box|
      box.vm.box = params['image']
      box.vm.provider "virtualbox" do |v|
        v.memory = 2048
        v.cpus = 2
        if params['fix_console'] == 1
          v.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          v.customize ["modifyvm", :id, "--uartmode1", "file", "./#{name}_ttyS0.log"]
        end
      end
      box.vm.provider :libvirt do |v|
        v.memory = 2048
        v.cpus = 2
      end
      (params['scripts'] || []).each do |script|
        box.vm.provision :shell, inline: script
      end
      unless ENV['SKIP_BCC_BUILD'] || (params['skip_bcc_build'] == 1)
        box.vm.provision :shell, privileged: false, inline: $build_bcc
      end
      config.vm.post_up_message = <<-HEREDOC
#######
bpftrace source is available in /vagrant
Build command: cmake /vagrant -DCMAKE_INSTALL_PREFIX=/usr/local && make
#######
      HEREDOC
    end
  end
end
