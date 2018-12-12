# Ansible  #

### Installing ###
    root@ubuntuc:/home/vagrant# sudo apt-get update
    root@ubuntuc:/home/vagrant# apt-get install ansible

### Check version ###
    root@ubuntuc:/home/vagrant# ansible --version
    ansible 2.7.2
      config file = /etc/ansible/ansible.cfg
      configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
      ansible python module location = /usr/lib/python2.7/dist-packages/ansible
      executable location = /usr/bin/ansible
      python version = 2.7.12 (default, Nov 12 2018, 14:36:49) [GCC 5.4.0 20160609]
    root@ubuntuc:/home/vagrant#

### Validating Instalation ###
    root@ubuntuc:/home/Ansible# touch /tmp/ANSIBLE_CONFIG.cfg
    root@ubuntuc:/home/Ansible# export ANSIBLE_CONFIG=/tmp/ANSIBLE_CONFIG.cfg
    root@ubuntuc:/home/Ansible# 

Pastikan semua host bisa jalan dengan enable ssh ke semua host
kemudian coba cek modul ping apakah sudah sukses apa belum

    vi hosts
    vi ansible.cfg

    root@ubuntuc:/home/Ansible# cat hosts
	    [all]
	    centos1
    root@ubuntuc:/home/Ansible# cat ansible.cfg
	    [defaults]
	    inventory = hosts
    root@ubuntuc:/home/Ansible# 

Kemudian jalankan perintah berikut (pastikan python sudah terinstall di masing masing OS)

    vagrant@ubuntuc:~/Ansible$ ansible all -m ping

Perintah untuk cek module ping

atau 

    vagrant@ubuntuc:~/Ansible$ ansible all -i centos1, -m ping
    centos1 | SUCCESS => {
	    "changed": false,
	    "ping": "pong"
    }

Kemudian pastikan sudah sukses juga dengan debug

    vagrant@ubuntuc:~/Ansible$ ansible all -m debug
    centos1 | SUCCESS => {
    	"msg": "Hello world!"
    }
    centos2 | SUCCESS => {
    	"msg": "Hello world!"
    }
    ubuntu1 | SUCCESS => {
    	"msg": "Hello world!"
    }
    centos3 | SUCCESS => {
    	"msg": "Hello world!"
    }
    ubuntu2 | SUCCESS => {
    	"msg": "Hello world!"
    }
    ubuntu3 | SUCCESS => {
    	"msg": "Hello world!"
    }

Atau dengan cara 
 
    vagrant@ubuntuc:~/Ansible$ ansible all -m debug --args='msg=“This is custom debug"'
    centos1 | SUCCESS => {
    "msg": “This is custom debug"
    }

    vagrant@ubuntuc:~/Ansible$ ansible all -m debug --args='msg="this is custom debugg" verbosity=3'
    centos1 | SKIPPED
    
    vagrant@ubuntuc:~/Ansible$ ansible '*' -m ping
    ubuntu2 | SUCCESS => {
    "changed": false,
    "ping": "pong"
    }
    ubuntu1 | SUCCESS => {
    "changed": false,
    "ping": "pong"
    }

# Ansible Inventory #
1. dipakai untuk konektivitas antar host
1. isinya daftar host mana saja 
1. bisa dibuat group dan children

```ansible all -m ping -o```

    ubuntu2 | SUCCESS => {"changed": false, "ping": "pong"}
    ubuntu1 | SUCCESS => {"changed": false, "ping": "pong"}
    centos3 | SUCCESS => {"changed": false, "ping": "pong"}
    centos2 | SUCCESS => {"changed": false, "ping": "pong"}
    centos1 | SUCCESS => {"changed": false, "ping": "pong"}
    ubuntu3 | SUCCESS => {"changed": false, "ping": "pong"}


Ansible bisa dikonfigurasi user, password, dan port

    cat hosts
    [centos]
    centos1 ansible_user=vagrant ansible_port=2222
    centos2 ansible_user=vagrant
    centos3 ansible_user=vagrant
    [ubuntu]
    ubuntu1 ansible_become=true ansible_become_pass=password
    ubuntu2 ansible_become=true ansible_become_pass=password
    ubuntu3 ansible_become=true ansible_become_pass=password

Ansible bisa baca konfig local dengan tambahan ansible_connection=local

    cat hosts
    [control]
    ubuntuc ansible_connection=local
    ubuntuc | SUCCESS => {
    "changed": false,
    "ping": "pong"
    }

Bisa juga list `hostnya` dibuat range

    [control]
    ubuntuc ansible_connection=local
    [centos]
    centos1 
    centos[2:3] ansible_user=root
    [ubuntu]
    ubuntu[1:3]

vagrant@ubuntuc: $ ansible all --list-hosts
    
	hosts (7):
    ubuntuc
    centos1
    centos2
    centos3
    ubuntu1
    ubuntu2
    ubuntu3

Ketika di ping bagian control group juga akan sukses

    vagrant@ubuntuc: $ ansible control -m ping
    ubuntuc | SUCCESS => {
    "changed": false,
    "ping": "pong"
    }
    
Grouping dengan semua variable

    vagrant@ubuntuc: $ cat hosts
    [control]
    ubuntu-c ansible_connection=local
    [centos]
    centos1 ansible_port=2222
    centos[2:3]
    
    [centos:vars]
    ansible_user=root
    
    <semua group centos akan diloginkan dengan root>
    
    [ubuntu]
    ubuntu[1:3]
    
    [ubuntu:vars]
    ansible_become=true
    ansible_become_pass=password
    <semua group ubuntu akan dilogin prompt dan pswd password>

Cek hasil

    vagrant@ubuntuc: $ ansible all -m ping -o
	    ubuntu-c | SUCCESS => {"changed": false, "ping": "pong"}
	    ubuntu1 | SUCCESS => {"changed": false, "ping": "pong"}
	    ubuntu2 | SUCCESS => {"changed": false, "ping": "pong"}
	    ubuntu3 | SUCCESS => {"changed": false, "ping": "pong"}
	    centos1 | SUCCESS => {"changed": false, "ping": "pong"}
	    centos2 | SUCCESS => {"changed": false, "ping": "pong"}
	    centos3 | SUCCESS => {"changed": false, "ping": "pong”}

Kemudian bisa dibuat dalam children linux

    vagrant@ubuntuc:
    [linux:children]
    centos
    ubuntu

Penulisan hosts bisa dibuat dalam bentuk `yml, JSON,` dan didefinisikan juga pada `file ansible.cfg`

    vagrant@ubuntuc: $ cat hosts.yml
    ---
    control:
      hosts:
    ubuntu-c:
      ansible_connection: local
    centos:
      hosts:
    	centos1:
      ansible_port: 2222
    centos2:
    centos3:
      vars:
    ansible_user: root
    ubuntu:
      hosts:
    ubuntu1:
    ubuntu2:
    ubuntu3:
      vars:
    ansible_become: true
    ansible_become_pass: password
    linux:
      children:
    centos:
    ubuntu:
    ...

vagrant@ubuntuc: $ cat hosts.json
{
    "control": {
        "hosts": {
            "ubuntu-c": {
                "ansible_connection": "local"
            }
        }
    },
    "ubuntu": {
        "hosts": {
            "ubuntu1": null,
            "ubuntu2": null,
            "ubuntu3": null
        },
        "vars": {
            "ansible_become": true,
            "ansible_become_pass": "password"
        }
    },
    "centos": {
        "hosts": {
            "centos3": null,
            "centos2": null,
            "centos1": {
                "ansible_port": 2222
            }
        },
        "vars": {
            "ansible_user": "root"
        }
    },
    "linux": {
        "children": {
            "centos": null,
            "ubuntu": null
        }
    }
}

Pemanggilan inventory bisa dipakai dengan perintah 
vagrant@ubuntuc: $ ansible all -i hosts.yml -m ping -o
ubuntu-c | SUCCESS => {"changed": false, "ping": "pong"}

Atau bisa dideklarasikan di command line
vagrant@ubuntuc: $ ansible linux -m ping -e 'ansible_port=22'
