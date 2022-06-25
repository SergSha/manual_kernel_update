<h3>### Kernel Update ###</h3>

<h4>Описание домашнего задания</h4>

<p>Обновить ядро в базовой системе.</p>
<br />

<h4># Установка необходимого ПО</h4>

<p>&#8226; Vagrant</p>
<p>Переходим на https://www.vagrantup.com/downloads.html выбираем соответствующую версию. В данном случае CentOS/RHEL. Копируем ссылку и в консоли выполняем:</p>

<pre>sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install vagrant</pre>

После успешного окончания будет установлен Vagrant.

<p>&#8226; Packer</p>
<p>Переходим на https://www.packer.io/downloads.html выбираем соответствующую версию. В данном случае CentOS/RHEL. Копируем ссылку и в консоли выполняем:</p>

<pre>sudo yum install -y yum-utils
sudo yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
sudo yum -y install packer</pre>

<p>После успешного окончания будет установлен Packer.</p>

<p>В домашней директории создадим директорию kernelupdate, в которой будут храниться настройки виртуальной машины:</p>

<pre>[user@localhost otus]$ mkdir ./kernelupdate
[user@localhost otus]$</pre>

<p>Перейдём в директорию kernelupdate:</p>

<pre>[user@localhost otus]$ cd ./kernelupdate/
[user@localhost kernelupdate]$</pre>

<p>Создадим файл Vagrantfile:</p>

<pre>[user@localhost systemd]$ vi ./Vagrantfile</pre>

<p>Заполним следующим содержимым:</p>

<pre># Describe VMs
MACHINES = {
  # VM name "kernel update"
  :"kernelupdate" => {
              # VM box
              :box_name => "centos/7",
              # VM CPU count
              :cpus => 2,
              # VM RAM size (Mb)
              :memory => 1024,
              # networks
              :net => [],
              # forwarded ports
              :forwarded_port => []
            }
}

Vagrant.configure("2") do |config|
  MACHINES.each do |boxname, boxconfig|
    # Disable shared folders
    config.vm.synced_folder ".", "/vagrant", disabled: true
    # Apply VM config
    config.vm.define boxname do |box|
      # Set VM base box and hostname
      box.vm.box = boxconfig[:box_name]
      box.vm.host_name = boxname.to_s
      # Additional network config if present
      if boxconfig.key?(:net)
        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
      end
      # Port-forward config if present
      if boxconfig.key?(:forwarded_port)
        boxconfig[:forwarded_port].each do |port|
          box.vm.network "forwarded_port", port
        end
      end
      # VM resources config
      box.vm.provider "virtualbox" do |v|
        # Set VM RAM size and CPU count
        v.memory = boxconfig[:memory]
        v.cpus = boxconfig[:cpus]
      end
    end
  end
end</pre>

<p>Запустим систему:</p>

<pre>[user@localhost kernelupdate]$ vagrant up
Bringing machine 'kernelupdate' up with 'virtualbox' provider...
==> kernelupdate: Importing base box 'centos/7'...
==> kernelupdate: Matching MAC address for NAT networking...
==> kernelupdate: Checking if box 'centos/7' version '2004.01' is up to date...
==> kernelupdate: There was a problem while downloading the metadata for your box
==> kernelupdate: to check for updates. This is not an error, since it is usually due
==> kernelupdate: to temporary network problems. This is just a warning. The problem
==> kernelupdate: encountered was:
==> kernelupdate: 
==> kernelupdate: The requested URL returned error: 404
==> kernelupdate: 
==> kernelupdate: If you want to check for box updates, verify your network connection
==> kernelupdate: is valid and try again.
==> kernelupdate: Setting the name of the VM: kernelupdate_kernelupdate_1656155988714_23553
==> kernelupdate: Clearing any previously set network interfaces...
==> kernelupdate: Preparing network interfaces based on configuration...
    kernelupdate: Adapter 1: nat
==> kernelupdate: Forwarding ports...
    kernelupdate: 22 (guest) => 2222 (host) (adapter 1)
==> kernelupdate: Running 'pre-boot' VM customizations...
==> kernelupdate: Booting VM...
==> kernelupdate: Waiting for machine to boot. This may take a few minutes...
    kernelupdate: SSH address: 127.0.0.1:2222
    kernelupdate: SSH username: vagrant
    kernelupdate: SSH auth method: private key
    kernelupdate: 
    kernelupdate: Vagrant insecure key detected. Vagrant will automatically replace
    kernelupdate: this with a newly generated keypair for better security.
    kernelupdate: 
    kernelupdate: Inserting generated public key within guest...
    kernelupdate: Removing insecure key from the guest if it's present...
    kernelupdate: Key inserted! Disconnecting and reconnecting using new SSH key...
==> kernelupdate: Machine booted and ready!
==> kernelupdate: Checking for guest additions in VM...
    kernelupdate: No guest additions were detected on the base box for this VM! Guest
    kernelupdate: additions are required for forwarded ports, shared folders, host only
    kernelupdate: networking, and more. If SSH fails on this machine, please install
    kernelupdate: the guest additions and repackage the box to continue.
    kernelupdate: 
    kernelupdate: This is not an error message; everything may continue to work properly,
    kernelupdate: in which case you may ignore this message.
==> kernelupdate: Setting hostname...
[user@localhost kernelupdate]$</pre>

<p>Войдём в неё с помощью SSH:</p>

<pre>[user@localhost kernelupdate]$ vagrant ssh
[vagrant@kernelupdate ~]$</pre>

<p>Заходим под правами root:</p>

<pre>[vagrant@kernelupdate ~]$ sudo -i
[root@kernelupdate ~]#</pre>

<p>Определим версию ядра:</p>

<pre>[root@kernelupdate ~]# uname -r
3.10.0-1127.el7.x86_64
[root@kernelupdate ~]#</pre>

<p>Теперь приступим к обновлению ядра.</p>

<h4># Kernel Update</h4>


