# Open House - Mobile Device Testing as a Service

How difficult is to setup an infrastructure to test your mobile applications in Continous Infrastructure?

The presentation demoes what can be achieved with Ansible, Jenkins and few pipelines on standard hardware. It also talks about pros & cons of the solution.

## Android Testing

### Prerequisites

- machine where Jenkins master will be running
  - [git](https://git-scm.com/) installed
  - [VirtualBox](https://www.virtualbox.org/) installed
  - 10GB memory (16GB in case you want to run VM with Jenkins slave there)
- (virtual) machine with Fedora 27 installed (Jenkins slave)
  - 6GB memory
  - 100GB disk size
- Android device connected to Jenkins slave

### Installation

#### Virtual machine setup (optional)

- install [VirtualBox](https://www.virtualbox.org/)
- download [Fedora 27 Workstation Live image](https://getfedora.org/en_GB/workstation/download/)
- install Fedora in VM
- configure the VM
  - in Network tab change Network type to `Bridget networking`
  - in Ports/USB tab add your Android device connected via USB to your host machine
- start the VM

#### Jenkins slave ssh setup

Perform these steps on Jenkins slave machine:

- make sure there is an account with sudo enabled
- make sure sshd is running
  - `sudo systemctl enable sshd`
  - `sudo systemctl start sshd`
  - `sudo systemctl status sshd` - should be `active (running)`
- get your IP with `ifconfig` command

Perform these steps on Jenkins master machine:

- make sure you can ssh into the Jenkins slave machine
  - `ssh <USER>@<SLAVE_IP>`
  - enter your password
  - `exit`
- make sure you can ssh using key based authentication
  - generate ssh key (if you don't have it already) with `ssh-keygen -t rsa`
  - `scp ~/.ssh/id_rsa.pub <USER>@<SLAVE_IP>:~/`
  - `ssh <USER>@<SLAVE_IP>`
  - enter your password
  - make sure `.ssh` folder exists in `~`
  - `cat id_rsa.pub >> .ssh/authorized_keys`
  - `exit`
- now try to ssh again
  - `ssh <USER>@<SLAVE_IP>`
  - you should be logged in without entering your password
  - `exit`

#### Jenkins master initial setup

- [install oc Client Tools](https://www.openshift.org/download.html)
- [install Minishift](https://docs.openshift.org/latest/minishift/getting-started/installing.html)
- start Minishift with `minishift start --cpus 4 --memory 6GB --disk-size 100GB --vm-driver=virtualbox`
- take note of OpenShift address
- [install Ansible](http://docs.ansible.com/ansible/latest/intro_installation.html)
- clone aerogear-digger-installer repo with `git clone https://github.com/jhellar/aerogear-digger-installer.git`
- `cd aerogear-digger-installer`
- checkout `openhouse` branch with `git checkout openhouse`
- open `openhouse-linux-inventory` with text editor
- replace `master_url` and `login_url` values with your OpenShift address
- replace address under `appium` section with your Jenkins slave machine IP address
- replace `linux_user` value with your Jenkins slave user name
- replace `ansible_become_pass` value with your Jenkins slave user password
- generate ssh key
  - `ssh-keygen -t rsa`
  - leave the password empty

#### Setup the rest with ansible

- `ansible-playbook -i openhouse-linux-inventory openhouse-linux.yml -e "jenkins_public_key_path=<SSH_KEY_PATH>/id_rsa.pub" -e "jenkins_private_key_path=<SSH_KEY_PATH>/id_rsa" -e "add_public_key_automatically=true"`
- take note of Jenkins instance address

### Build and test Android application

- enter your Jenkins instance address to your browser
- login with `admin`/`password`
- create `New Item`
- enter a name (e.g. Helloworld)
- select `Pipeline`
- `OK`
- under `Pipeline` section select `Pipeline script from SCM`
- as a `SCM` select `Git`
- set `Repository URL` as `https://github.com/jhellar/android-helloworld`
- `Save`
- `Build Now`

## iOS and Android Testing

### Prerequisites

- machine where Jenkins master will be running
  - [git](https://git-scm.com/) installed
  - [VirtualBox](https://www.virtualbox.org/) installed
  - 10GB memory
- Mac (Jenkins slave)
  - Xcode installed
  - Xcode command line tools installed
  - Homebrew installed ([enabled for all users](https://gist.github.com/jaibeee/9a4ea6aa9d428bc77925))
  - JDK installed
- iOS device connected to Jenkins slave
- Android device (emulator) connected to Jenkins slave
- Apple developer account
  - iOS wildcard certificates for debug

### Installation

#### Mac setup

- create new user for Jenkins
- enable ssh for this user
- make sure you are able to ssh (key based authentication) from your Jenkins master machine

#### Jenkins master initial setup

- follow the same steps as in the Android section
  - instead of `openhouse-linux-inventory` use `openhouse-inventory`
  - instead of `linux_user` use `macos_user`
- create a folder with your iOS certificates (private key, certificate, provisioning profile)
  - names of these files should be `debug.p12`, `debug.cer`, `debug.mobileprovision`
  - zip these files
- open `openhouse-inventory` with text editor
- replace `jdk_version` value with one you have installed (you should be able to find it in `/Library/Java/JavaVirtualMachines`)
- replace `xcode_org_id` value with your ID
- replace `provisioning_profile` value with your provisioning profile name

#### Setup the rest with ansible

- `ansible-playbook -i openhouse-inventory openhouse.yml -e "jenkins_public_key_path=<SSH_KEY_PATH>/id_rsa.pub" -e "jenkins_private_key_path=<SSH_KEY_PATH>/id_rsa" -e "add_public_key_automatically=true" -e "certificates_dir=<IOS_CERTIFICATES_PATH>" -e "key_password=<PRIVATE_KEY_PASSWORD>"`
- take note of Jenkins instance address

### Build and test iOS application

- follow the same steps as in the Android section
  - for `Repository URL` use `https://github.com/jhellar/ios-helloworld`
  - in configuration section of the job check `This project is parameterised`
  - add string parameter with name `BUILD_CREDENTIAL_ID`
  - [add the zip](https://aerogear.org/docs/digger/developer/#build-application) with iOS certificates to Jenkins
