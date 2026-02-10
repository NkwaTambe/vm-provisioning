# Virtualization and VM Provisioning

This repository contains examples and configurations for VM provisioning and management using different tools and technologies. The examples are based on an assignment covering VM export/import, automated provisioning with Vagrant, and configuration with Cloud-Init.

For detailed steps and screenshots, please refer to the [assignment PDF](VM_Provisioning.pdf).

## 1. Exporting and Importing a VM

This section demonstrates the process of exporting an existing Virtual Machine and importing it to create a new one.

### Objective
Create a VM, export its configuration and disk, then import it into a new VM and verify it boots successfully.

### Steps Performed
1.  **Create a Virtual Machine**: A VM was created with 2 CPUs, 2 GB RAM, and a 20 GB disk. Updates and basic packages were installed.
2.  **Export the VM**: The VM was exported in the Open Virtualization Format (OVF), including all disk files and configuration.
3.  **Import into a New VM**: The exported OVF package was imported to create a new VM, verifying that the resources matched the original.
4.  **Verification**: The new VM was booted successfully. For screenshots of the process, please refer to the [assignment PDF](VM_Provisioning.pdf).

## 2. VM Provisioning using Vagrant

This section covers the automatic provisioning of a Virtual Machine using Vagrant.

### Objective
Automatically provision a VM using Vagrant.

### Vagrantfile
The following `Vagrantfile` is used to define and provision the VM. It sets up a virtual machine with Ubuntu 22.04, configures its resources, forwards a port, syncs a local folder, and installs Nginx.

```ruby
Vagrant.configure("2") do |config|
  # Base box
  config.vm.box = "generic/ubuntu2204"

  # VMware provider settings
  config.vm.provider "vmware_desktop" do |v|
    v.memory = 2048   # 2 GB RAM
    v.cpus = 2        # 2 CPU cores
    v.linked_clone = false
  end

  # Forward HTTP port from guest to host
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Sync current folder to VM
  config.vm.synced_folder ".", "/vagrant"

  # Provision VM with Nginx and copy index.html
  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    apt-get install -y nginx
    sudo cp /vagrant/index.html /var/www/html/index.html
    sudo systemctl enable nginx
    sudo systemctl start nginx
  SHELL
end
```

### Steps Performed
1.  **Initialize Vagrant environment**: `vagrant init`
2.  **Start and provision VM**: `vagrant up`
3.  **Verification**: The Nginx service was verified to be running on the VM. For a screenshot of the service status, please refer to the [assignment PDF](VM_Provisioning.pdf).

## 3. VM Configuration using Cloud-Init

This section explains how to configure a VM on its first boot using `cloud-init`.

### Objective
Configure a VM on first boot using cloud-init, automating user creation, package installation, and service setup.

### Cloud-Init Configuration (`cloud-init.yaml`)
The configuration below automates the setup of a user, installation of various packages, and execution of initial commands.

```yaml
#cloud-config
# Create a user with sudo privileges
users:
  - name: student
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    shell: /bin/bash

# Install packages automatically on first boot
packages:
  - nginx                 # Web server
  - git                   # Version control
  - vim                   # Text editor
  - htop                  # System monitor
  - curl                  # Command-line HTTP client
  - wget                  # Download tool
  - build-essential       # Compiler and development tools
  - python3               # Python runtime
  - python3-pip           # Python package manager
  - unzip                 # Extract zip files
  - net-tools             # Network tools (ifconfig, netstat)
  - tree                  # Directory structure visualization

# Run commands at first boot
runcmd:
  # Set up a welcome page for Nginx
  - echo "<h1>Welcome to Multipass Nginx Lab!</h1>" > /home/student/index.html
  - sudo mv /home/student/index.html /var/www/html/index.html
  # Enable and start Nginx
  - sudo systemctl enable nginx
  - sudo systemctl start nginx
  # Create directories for practice
  - mkdir -p /home/student/projects
  - mkdir -p /home/student/scripts
  # Update pip and install a Python package
  - python3 -m pip install --upgrade pip
  - python3 -m pip install requests
  # Log system info in a welcome file
  - echo "Multipass VM initialized successfully on $(date)" >> /home/student/welcome.log
  - uname -a >> /home/student/welcome.log
  - lsb_release -a >> /home/student/welcome.log
```

### Steps Performed
1.  **Create `cloud-init.yaml`**: The configuration file was created in the local directory.
2.  **Launch Multipass VM**: The VM was launched with `multipass launch --name cloudinit --cloud-init cloud-init.yaml`.
3.  **Connect to VM**: Connected to the VM using `multipass shell cloudinit`.
4.  **Verification**:
    *   User `student` exists with sudo privileges.
    *   Nginx service is running.
    *   Packages like `git`, `vim`, `htop`, `python3`, `pip` are installed.
    *   Welcome webpage is accessible.
    *   `projects` and `scripts` directories are created.
    *   `welcome.log` file contains system info.

For screenshots of the verification steps, please refer to the [assignment PDF](VM_Provisioning.pdf).
