![](https://cdn-images-1.medium.com/max/697/1*qykIU2PUoEnkdoPuYvIlOg.png)

# Infrastructur as a Code  #


Infrastructure as Code adalah proses penyediaan IT infrastruktur dimana sistem dibangun dan dikelola melalui kode secara automasi (otomatis), bukan secara manual. Atau bisa disebut juga bahasa kerennya Programmable Infrastructure.

Dengan menggunakan kode dan meng-automasi, proses setting dan konfigurasi baremetal, virtual mesin, cloud computing baik itu instalasi baru, perubahan konfigurasi dapat dilakukan secara cepat, mudah, dan berulang. Di sisi lain juga, ada manfaat lainnya yaitu dokumentasi. Jadi siapapun tahu konfigurasi server, kebutuhan aplikasi server dan sebagainya.

Untuk menerapkan IAC tools/alat yang digunakan adalah 	Packer (build image), Terraform (build vm dan privisioning instance/vm/server), Ansible, Chef, Puppet & Salt Stack (Configurations Management).

Artikel kali ini menggunakan `ansible` untuk demo `IAAC`

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


Ansible bisa dikonfigurasi `user, password, dan port`

    cat hosts
    [centos]
	    centos1 ansible_user=vagrant ansible_port=2222
	    centos2 ansible_user=vagrant
	    centos3 ansible_user=vagrant
    [ubuntu]
	    ubuntu1 ansible_become=true ansible_become_pass=password
	    ubuntu2 ansible_become=true ansible_become_pass=password
	    ubuntu3 ansible_become=true ansible_become_pass=password

Ansible bisa baca konfig local dengan tambahan `ansible_connection=local`

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

Ketika di ping bagian `control group` juga akan sukses

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

Kemudian bisa dibuat dalam `children linux`

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

vagrant@ubuntuc: $ `cat hosts.json`
    
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

## Ansible Module ##

**The Setup Module**
	
1. Digunakan untuk mengambil fact ketika mengeksekusi playbook
1. Otomatis dipanggil oleh playbook untuk mendapatkan variable dari remote hosts
1. Bisa dipakai langsung di binary ansible
1. Ansible menyediakan beberapa facts tentang system



**Coba panggil**

    vagrant@ubuntuc: $ ansible centos1 -m setup | more

modul setup dipakai untuk mendapatkan info os centos1

Outputnya akan berupa json format yang berupa facts

    vagrant@ubuntuc: $ ansible centos1 -m setup | more
    centos1 | SUCCESS => {
	    "ansible_facts": {
	    "ansible_all_ipv4_addresses": [
	    "192.168.50.21",
	    "10.0.2.15"
    ],
	    "ansible_all_ipv6_addresses": [
	    "fe80::a00:27ff:fe21:3069",
	    "fe80::5054:ff:fec0:42d5"
    ],
    
## The File module ##

1. Kumpulan atribute files, symlinks dan directory atau penghapusan `files/symlinks/directory`
1. Banyak module yang support seperti files module termasuk `copy, template, dan assemble`
1. Untuk windows menggunakan `win_file`



Contoh membuat file testing di /tmp

    vagrant@ubuntuc: $ ansible all -m file -a 'path=/tmp/testing state=touch'
    centos1 | CHANGED => {
	    "changed": true,
	    "dest": "/tmp/testing",
	    "gid": 1000,
	    "group": "vagrant",
	    "mode": "0664",
	    "owner": "vagrant",
	    "secontext": "unconfined_u:object_r:user_tmp_t:s0",
	    "size": 0,
	    "state": "file",
	    "uid": 1000
    }

Ketika diulangi lagi maka akan warnanya berubah jadi hijau karena hanya sukses saja

    	vagrant@ubuntuc: $ ansible all -m file -a 'path=/tmp/testing state=file mode=600'
    	ubuntu1 | SUCCESS => {
	    	"changed": false,
	    	"gid": 1000,
	    	"group": "vagrant",
	    	"mode": "0600",
	    	"owner": "vagrant",
	    	"path": "/tmp/testing",
	    	"size": 0,
	    	"state": "file",
	    	"uid": 1000
    	}

Cek directory /tmp pada masing masing host

    vagrant@ubuntu3:~$ ls /tmp/
    testing  vagrant-shell
    vagrant@ubuntu3:~$ 

## File Permission ##

    vagrant@ubuntuc: $ ansible all -m file -a 'path=/tmp/testing state=file mode=600'
    ubuntu1 | CHANGED => {
	    "changed": true,
	    "gid": 1000,
	    "group": "vagrant",
	    "mode": "0600",
	    "owner": "vagrant",
	    "path": "/tmp/testing",
	    "size": 0,
	    "state": "file",
	    "uid": 1000
    }

## Color Notations ##
![#c5f015](https://placehold.it/15/00FF00/000000?text=+) **Green : Success**

![#c5f015](https://placehold.it/15/ffff00/000000?text=+) **Yellow : Success with change**

![#f03c15](https://placehold.it/15/f03c15/000000?text=+)  **Red : Failure**

## Idempotence ##

artinya operasi yang jika dijalankan menghasilkan hasil yang bisa sama dan tanpa adanya tabrakan 
Simulasi, coba ubah permission file testing

    vagrant@ubuntu1:~$ chmod 644 /tmp/testing
    vagrant@ubuntu1:~$ ls -latrh /tmp/testing
    -rw-r--r-- 1 vagrant vagrant 0 Nov 17 15:07 /tmp/testing
    vagrant@ubuntu1:~$ 

Kemudian jalankan ulang perintah, maka akan ditampilkan hijau dan kuning

    vagrant@ubuntuc: $ ansible all -m file -a 'path=/tmp/testing state=file mode=600'
    ubuntu1 | CHANGED => {
	    "changed": true,
	    "gid": 1000,
	    "group": "vagrant",
	    "mode": "0600",
	    "owner": "vagrant",
	    "path": "/tmp/testing",
	    "size": 0,
	    "state": "file",
	    "uid": 1000
    }
    ubuntu2 | SUCCESS => {
	    "changed": false,
	    "gid": 1000,
	    "group": "vagrant",
	    "mode": "0600",
	    "owner": "vagrant",
	    "path": "/tmp/testing",
	    "size": 0,
	    "state": "file",
	    "uid": 1000
    }

## The Copy module ##

Dipakai untuk menyalin file dari local/remote maching ke lokasi remote machine. Menggunakan fetch untuk copy, jika dibutuhkan module tambahan gunakan [template] module. Untuk windows target gunakan `[win_copy]`

    vagrant@ubuntuc: $ touch /tmp/x.txt
    vagrant@ubuntuc: $ ansible all -m copy -a 'src=/tmp/x.txt dest=/tmp/x.txt’
    ubuntu2 | CHANGED => {
	    "changed": true,
	    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
	    "dest": "/tmp/x.txt",
	    "gid": 0,
	    "group": "root",
	    "md5sum": "d41d8cd98f00b204e9800998ecf8427e",
	    "mode": "0644",
	    "owner": "root",
	    "size": 0,
	    "src": "/home/vagrant/.ansible/tmp/ansible-tmp-1542492544.47-252278220394494/source",
	    "state": "file",
	    "uid": 0
    }

## The Command module ##

Digunakan untuk mengeksekusi command remote. Perintah ini dipakai untuk default command di ansible

    vagrant@ubuntuc: $ ansible all -m command -a 'hostname' -o
    ubuntu-c | CHANGED | rc=0 | (stdout) ubuntuc
    centos2 | CHANGED | rc=0 | (stdout) centos2
    centos1 | CHANGED | rc=0 | (stdout) centos1
    ubuntu1 | CHANGED | rc=0 | (stdout) ubuntu1
    centos3 | CHANGED | rc=0 | (stdout) centos3
    ubuntu2 | CHANGED | rc=0 | (stdout) ubuntu2
    ubuntu3 | CHANGED | rc=0 | (stdout) ubuntu3
    vagrant@ubuntuc: $ ansible all -m command -a 'df -h'
    ubuntu-c | CHANGED | rc=0 >>
    Filesystem  Size  Used Avail Use% Mounted on
    udev488M 0  488M   0% /dev
    tmpfs   100M  3.1M   97M   4% /run
    /dev/sda1   9.7G  1.2G  8.5G  13% /
    tmpfs   497M  124K  496M   1% /dev/shm
    tmpfs   5.0M 0  5.0M   0% /run/lock
    tmpfs   497M 0  497M   0% /sys/fs/cgroup
    vagrant  17G   15G  2.1G  89% /vagrant
    tmpfs   100M 0  100M   0% /run/user/1000

Coba buat file

    vagrant@ubuntuc: $ ansible all -m command -a 'touch /tmp/test_copy_module creates=/tmp/test_copy_module’
    ubuntu1 | CHANGED | rc=0 >>

Ulangi kembali perintah buat file 

    vagrant@ubuntuc: $ ansible all -m command -a 'touch /tmp/test_copy_module creates=/tmp/test_copy_module'
    ubuntu2 | SUCCESS | rc=0 >>
    skipped, since /tmp/test_copy_module exists

Coba hapus file

    vagrant@ubuntuc: $ ansible all -m command -a 'rm /tmp/test_copy_module removes=/tmp/test_copy_module’
    ubuntu2 | CHANGED | rc=0 >>

Ulangi kembali hapus file

    vagrant@ubuntuc: $ ansible all -m command -a 'rm /tmp/test_copy_module removes=/tmp/test_copy_module'
    ubuntu-c | SUCCESS | rc=0 >>
    skipped, since /tmp/test_copy_module does not exist

Menghapus equivalen dengan state absent

    vagrant@ubuntuc: $ ansible all -m file -a 'path=/tmp/test_copy_module state=absent'
    ubuntu1 | SUCCESS => {
	    "changed": false,
	    "path": "/tmp/test_copy_module",
	    "state": "absent"
    }

## Challege ##
gunakan file module untuk membuat file di /tmp/test_module.txt dengan permission 600

    vagrant@ubuntuc: $ ansible centos1 -m file -a 'path=/tmp/test_module.txt state=touch mode=600'
    centos1 | CHANGED => {
	    "changed": true,
	    "dest": "/tmp/test_module.txt",
	    "gid": 1000,
	    "group": "vagrant",
	    "mode": "0600",
	    "owner": "vagrant",
	    "secontext": "unconfined_u:object_r:user_tmp_t:s0",
	    "size": 0,
	    "state": "file",
	    "uid": 1000
    }

gunakan fetch module, copy file dari remote ke local

vagrant@ubuntuc: $ ansible centos1 -m fetch -a 'src=/tmp/test_module.txt dest=/tmp/text_module.txt'
centos1 | CHANGED => {
    "changed": true,
    "checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
    "dest": "/tmp/text_module.txt/centos1/tmp/test_module.txt",
    "md5sum": "d41d8cd98f00b204e9800998ecf8427e",
    "remote_checksum": "da39a3ee5e6b4b0d3255bfef95601890afd80709",
    "remote_md5sum": null
}

## Ansible-doc ##

vagrant@ubuntuc: $ `ansible-doc file`
> FILE(/usr/lib/python2.7/dist-packages/ansible/modules/files/file.py)
Sets attributes of files, symlinks, and directories, or
removes files/symlinks/directories. Many other modules support
the same options as the `file' module - including [copy],
[template], and [assemble]. For Windows targets, use the
[win_file] module instead.
OPTIONS (= is mandatory):
