
## Setup a Kubernetes cluster using Vagrant 


### Summary 

 1. Install Vagrant
 2. Clone repository
 3. Provision machines with Shell
 4. Provision machines with Ansible
 5. Launch cluster

 ### 1. Install Vagrant

For this setup, you'll first need to install Vagrant on your machine. Read the <a src = "https://www.vagrantup.com/docs/installation">documentation here</a>. In our case, I'm using the a NixOS distribution. 

I'll add `vagrant` in the  in my `/etc/nixos/confguration.nix` file : 

`
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
`

Finally, run `sudo nixos-rebuild switch`.

2. Clone repository 

get in your terminal and run `git clone https://github.com/EliottElek/k8s-setup-with-vagrant.git` to clone the repository.
