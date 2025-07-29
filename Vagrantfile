require 'yaml'
VAGRANTFILE_API_VERSION = "2"

if ENV['USER_PWD'].nil? || ENV['USER_PWD'].empty?
    raise 'Set your password in var USER_PWD, eg: USER_PWD=your_pwd vagrant up xxx'
end

node = []

vagrant.configure(2) do |config|
    if Vagrant.has_plugin?("vagrant-proxyconf")
        config.proxy.http           = "http://#{ENV['USER']}:#{ENV['USER_PWD']}@#{ENV['PROXY_IP']}:#{ENV['PROXY_PORT']}"
        config.proxy.https          = "https://#{ENV['USER']}:#{ENV['USER_PWD']}@#{ENV['PROXY_IP']}:#{ENV['PROXY_PORT']}"
        config.yum_proxy.http       = "http://#{ENV['USER']}:#{ENV['USER_PWD']}@#{ENV['PROXY_IP']}:#{ENV['PROXY_PORT']}"
        config.proxy.no_proxy       = "#{ENV['no_proxy']}"
    end
    ENV["VAGRANT_DETECTED_OS"] = ENV["VAGRANT_DETECTED_OS"].to_s + " cygwin"
    config.vm.network "private_network", type: "dhcp"

    node.each do |n|
        config.vm.define n[:hostname] do |nc|
            nc.vm.box = n[:box]
            nc.vm.hostname = n[:hostname]
            memory = n[:ram] ? n[:ram]: 256;
            cpu = n[:cpu] ? n[:cpu]: 1;
            if n[:provider].include? "virtualbox"
                nc.vm.provider "virtualbox" do |ncvb|
                    ncvb.memory = memory
                    ncvb.cpus = cpu
                end
            elif n[:provider].include? "vmware_fusion"
                nc.vm.provider "vmware_desktop" do |ncvw|
                    ncvw.vmx["memsize"] = memory.to_s
                    ncvw.vmx["numvcpus"] = cpu.to_s
                end
            elif n[:provider].include? "docker"
                nc.vm.provider "docker" do |ncd|
                    ncd.image = n[:image]
                end
            elif n[:provider].include? "hyperv"
                nc.vm.provider "hyperv" do |nchv|
                    nchv.vm_integration_services = {
                      guest_service_interface: true,
                      CustomVMSRV: true,
                      memory: memory,
                      cpus: cpu,
                      vm_name: n[:hostname]
                    }
                end
            end
            if n[:provision].include? "shell"
                nc.vm.provision "shell", path: n[:path]
            elsif n[:provision].include? "docker"
                if n[:provision].include? "build"
                    nc.vm.provision "docker" do |ncdocker|
                        ncdocker.build_image n[:dockerfile_path]
                    end
                elif n[:provision].include? "run"
                    nc.vm.provision "docker" do |ncdocker|
                        ncdocker.run n[:image], cmd: n[:cmd], args: n[:args]
                    end
                end
            elif n[:provison].include? "salt"
                if n[:type].include? "master"
                    nc.vm.provision "shell", :inline => "sudo yum install -y salt-master"
                elif n[:type].include? "minion"
                    nc.vm.provision "shell", :inline => "sudo yum install -y salt-minion"
                elif n[:type].include? "masterless"
                    nc.vm.provision "salt" do |ncsalt|
                        ncsalt.install_type = "stable"
                        ncsalt.masterless = true
                        ncsalt.log_level = "all"
                        ncsalt.minion_config = n[:salt_minion_config]
                    end
                end
            elif n[:provision].include? "ansible"
                nc.vm.provision "ansible" do |ncansible|
                    ncansible.playbook = n[:playbook_path]
                end
            end
        end
    end
end