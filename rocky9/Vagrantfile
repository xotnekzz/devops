# Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "generic/rocky9"
  
  # 서버 5대 정의 (IP: 192.168.56.11 ~ 15)
  servers = [
    { name: "server-1", ip: "192.168.56.11" },
    { name: "server-2", ip: "192.168.56.12" },
    { name: "server-6", ip: "192.168.56.16" },
    { name: "server-4", ip: "192.168.56.14" },
    { name: "server-5", ip: "192.168.56.15" }
  ]

  servers.each do |server|
    config.vm.define server[:name] do |node|
      node.vm.hostname = server[:name]
      node.vm.network "private_network", ip: server[:ip]
      
      node.vm.provider "virtualbox" do |vb|
        vb.memory = "2048"   # 2GB RAM
        vb.cpus = 2          # 2 vCPU
        vb.linked_clone = true
      end
    end
  end
end
