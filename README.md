# Open House - Mobile Device Testing as a Service

How difficult is to setup an infrastructure to test your mobile applications in Continous Infrastructure?

The presentation demoes what can be achieved with Ansible, Jenkins and few pipelines on standard hardware. It also talks about pros & cons of the solution.

## Android Testing

### Prerequisites

- machine where Jenkins master will be running
  - [git](https://git-scm.com/) installed
  - [VirtualBox](https://www.virtualbox.org/) installed for Minishift
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