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
            nc.vm.provider :virtualbox do |ncvb|
                ncvb.customize [
                     "modifyvm", :id,
                     "--memory", memory.to_s
               ]
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