
## Setup a Kubernetes cluster using Vagrant 

### Introduction

To use kubeadm, we first need to have a working VM network. There are several choices to do so. We could either set it up by hand, use vagrant and ansible or use NikitaLXD.

For this tutorial, we look at the Vagrant way. 

### Summary 

 1. Install Vagrant
 2. Clone repository
 3. Provision machines with Shell
 4. Provision machines with Ansible
 5. Launch cluster

 ### 1. Install Vagrant

For this setup, you'll first need to install Vagrant on your machine. Read the [documentation here](https://www.vagrantup.com/docs/installation). In our case, I'm using the a NixOS distribution. 

I'll add `vagrant` in the  in my `/etc/nixos/confguration.nix` file : 

```
{...}
environment.systemPackages = with pkgs; [
    vim
    curl
    thunderbird
    nodejs
    chromium
    keybase-gui
    keybase
    docker
    docker-compose
    kubernetes
    kubernetes-helm
    kubectl
    minikube
    yarn
    vagrant 
  ];
```

Finally, run `sudo nixos-rebuild switch`.

### 2. Clone repository 

get in your terminal and run `git clone https://github.com/EliottElek/k8s-setup-with-vagrant.git` to clone the repository.

You'll get the following structure:

![image](https://user-images.githubusercontent.com/64375473/164414643-fbc1807e-35d3-4500-b6b2-a46db0501f21.png)
 
 ### 3. Provision machines with Shell
 
 Take a look at the `Vagrantfile`.
 
 First, we define the IP base for our machines, as well as the number of worker machines (nodes).
 
```
NUM_WORKER_NODES=5
IP_NW="192.168.56."
IP_START=10
```
Then we start the configuration of the master machine : 

```
agrant.configure("2") do |config|
  config.vm.provision "shell", inline: <<-SHELL
      apt-get update -y
      echo "$IP_NW$((IP_START))  master-node" >> /etc/hosts
      echo "$IP_NW$((IP_START+1))  worker-node01" >> /etc/hosts
      echo "$IP_NW$((IP_START+2))  worker-node02" >> /etc/hosts
  SHELL

  config.vm.box = "bento/ubuntu-21.10"
  config.vm.box_check_update = true

  config.vm.define "master" do |master|
    # master.vm.box = "bento/ubuntu-18.04"
    master.vm.hostname = "master-node"
    master.vm.network "private_network", ip: IP_NW + "#{IP_START}"
    master.vm.provider "virtualbox" do |vb|
        vb.memory = 4048
        vb.cpus = 2
    end
    master.vm.provision "shell", path: "scripts/common.sh"
    master.vm.provision "shell", path: "scripts/master.sh"
  end

```

Finally, for each worker node, we define the configuration, just like for the master node : 

```
(1..NUM_WORKER_NODES).each do |i|

  config.vm.define "node0#{i}" do |node|
    node.vm.hostname = "worker-node0#{i}"
    node.vm.network "private_network", ip: IP_NW + "#{IP_START + i}"
    node.vm.provider "virtualbox" do |vb|
        vb.memory = 2048
        vb.cpus = 1
    end
    node.vm.provision "shell", path: "scripts/common.sh"
    node.vm.provision "shell", path: "scripts/node.sh"
  end

  end

```
The `Vagrantfile` file reffers to the shell scripts located in the `/scripts` folder. The master node uses `master.sh` and the worker nodes use `node.sh`.

These scripts are used to configure your machines. They'll be run on your machines to install the necessary dependencies, in our case, kubeadm, kubectl...

### 4. Provision machines with Ansible
  
This part is still on process.
  
### 5. Launch cluster
   
Navigate to your main folder, where your `Vagrantfile` is located.

run `vagrant up`. This might take a while. 

Once the setup is finished, run `vagrant status` to make sure that your machines are correctly setup. This is what you should get : 

![image](https://user-images.githubusercontent.com/64375473/164425661-64fabc34-8cba-4fa2-83b6-850135b5e8a6.png)

We'll make some further tests to make sure our cluster is correctly setup and that our machines can communicate with each other.

On you terminal, run `vagrant ssh master` to enter the master machine. 

![image](https://user-images.githubusercontent.com/64375473/164426095-b32ce9d4-a033-4b0d-8f76-0e0c3656ae68.png)

You are now on the master machine's terminal. 

run `kubectl get nodes` to list the nodes : 

![image](https://user-images.githubusercontent.com/64375473/164426382-39986b5b-5bff-4d1e-9cc0-e15f0c553e5b.png)

We can see that all our machines are running correctly. Let's now make sure that our machines can communicate with each other. 

For that, we'll try to ping one of our worker machines from master.

In our case, one of the worker machines, worker2 for example, has the following IP address : **192.168.56.12**.

To ping it, let's run, from the master's terminal : `ping 192.168.56.12`. If everything went right, we should get :

![image](https://user-images.githubusercontent.com/64375473/164427179-430810ae-1c69-4616-be75-7f774eaeec36.png)

