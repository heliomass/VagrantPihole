Vagrant.configure("2") do |config|

  config.vm.box = "alpine/alpine64"
  config.vm.define "pyhole"
  config.vm.hostname = "pyhole.local"

  #
  # Run Ansible from the Vagrant Host
  #
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "provisioning/playbook.yml"
    ansible.groups = {
      "pyhole" => ["pyhole"]
    }
  end

end
