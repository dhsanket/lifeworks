# lifeworks

### getting the vm up and installing nginx, python-pip, 
1. clone git repository - it is public
> vagrant up - should bring up vagrant guest machine
3. can ssh into vagrant machine as vagrant@127.0.0.1 - double check ssh port as vagrant swaps it on various loads
4. adjust location of your private key with below command.
> ssh -i /home/sanket-rhythmone/lifeworks/.vagrant/machines/default/virtualbox/private_key -p 2200 vagrant@127.0.0.1
> ansible ssh port config can be found in the lifeworks-inventory file

5. you will need ansible version 2.5.5. preferably inside an ansible virtualenv so that you do not pollute your environment
to provisin the server use ansible-playbook module. and call it like this.
> ansible-playbook -i lifeworks-inventory site.yml

after ansible provisions the server. nginx will be available on host machine at localhost:1234


### my work log and comments on troubleshooting and problems faced

 downloading Vagrant \
 also considering using jupyter-notebook to serve a python code interpreter which will have the fizzbuzz test easy to run
 vagarant will have to port forward jupyter-notebook to host port so that notebook can be accessed
 wrote fizzbuzz in python using ipython notebook interpreter locally\
 
> def fizzbuzz():\
> ....for i in range(1,101):\  
> ........if i%3==0 and i%5==0:\
> ..........print "fizubuzz"\
> ........elif i%3==0:\
> ..........print "fizz"\
> ........elif i%5==0:\
> ..........print "buzz"\
> ........else:\
> ..........print str(i)\

 checked results of fizzbuzz to reasonable depth\
 1\
 2\
 fizz\
 4\
 buzz\
 fizz\
 7\
 8\
 fizz\
 buzz\
 11\
 fizz\
 13\
 14\
 fizubuzz\
 16\
 17\
 fizz\
 19\

tried vagrant up. it fails with plugin conflicting dependecy errors.  \ 
 trying vagrant plugin expunge --reinstall
 "vagrant expunge has been declined"
uninstalled and reinstalled vagrant.
same results as above. 

installing hostmanager pluging, it manages hosts file

vagrant plugin install vagrant-hostmanager
 
 trying vagrant again
 1. vagrant up 
 vagrant comes up !!!
> default: SSH address: 127.0.0.1:2222
    default: SSH username: vagrant
    default: SSH auth method: private key
==> default: Machine booted and ready!
 default: Mounting shared folders...
    default: /vagrant => /home/sanket-rhythmone/lifeworks

trying vagrant ssh default
Errors - OpenSSL version mismatch. Built against 1000200f, you have 1000105f

inspecting vagrantbox ssh config info using 
vagrant ssh-config
gives details about the vagrant box, and there is a ssh private key location too.

trying alternative ssh login vagrant ssh using ssh key and 127.0.0.1 port2222
WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!
Host key verification failed.
remove known host for vagrant 
ssh-keygen -f "/home/sanket-rhythmone/.ssh/known_hosts" -R "[127.0.0.1]:2222"

now trying ssh again 
ssh -i /home/sanket-rhythmone/lifeworks/.vagrant/machines/default/virtualbox/private_key -p 2222 vagrant@127.0.0.1
Welcome to your Scale Factory virtual machine.
[vagrant@centos6 ~]$
success loggin into vagrant

Using ansible to collect official nginx role to install onto vagrant box. \
nginx role needs various variables to be configured to make installation for CentOS. 

Vagrant trouble again. "insecure private key, making new one" then asks for password at login.
--solution - vagrant sets up ssh port different each time I do vagrant up. so ssh login command has to be updated each time.
> ansible ssh port config can be found in the lifeworks-inventory file
 
and again - Vagrant cannot forward the specified ports on this VM, since they
would collide...Solution - discovered I need to destroy previous running vagrant instances to eliminate this error.

### RUN ANSIBLE using this command - ansible-playbook -i lifeworks-inventory site.yml

Ansible site.yml is configured to install nginx and run it.

successfully started nginx on vagrant box. port forwarded to localhost:1234. nginx webpage shows up from browser.

successfully added ansible tasks to 
1. install required libraries for development
2. python development modules are needed aswell, python-devel.x86_64, lots of errors when installing
3. it was tricky to find and install the right python-pip version because CentOS does not come with python dependencies
4. installed and extracted python setup.py
5. found an older more compatible version of python-pip after much googling https://bootstrap.pypa.io/2.6/get-pip.py
6. for webserver stack installing jupyter-notebook on CentOS would involve compiling and Building the pyzmq. so decided to tackle nginx-uwsgi with python flask
7. currently figuring out an error causing timeout from upstream uwsgi to nginx proxy. proxy_pass looks fine, issue is elsewhere.

### Present State- Nginx is brought up, python-flask webapp files are made to work with "uwsgi", but not yet serving through nginx proxy
 
uwsgi --socket 127.0.0.1:8080 --protocol=http -w WSGI:app &
uwsgi --ini myconf.ini --socket 127.0.0.1:8080 --protocol=http -w WSGI:app &


