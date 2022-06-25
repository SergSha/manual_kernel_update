<h3>### Kernel Update ###</h3>

<h4>Описание домашнего задания</h4>

<p>Обновить ядро в базовой системе.</p>
<br />

<h4>**Установка необходимого ПО**</h4>

<p>Для выполнения работы потребуются следующие инструменты:</p>

<ul>
<li>VirtualBox - среда виртуализации, позволяет создавать и выполнять виртуальные машины;</li>
<li>Vagrant - ПО для создания и конфигурирования виртуальной среды. В данном случае в качестве среды виртуализации используется VirtualBox;</li>
<li>Packer - ПО для создания образов виртуальных машин;</li>
<li>Git - система контроля версий</li>
</ul>

<p>А так же аккаунты:</p>

<ul>
<li>GitHub - https://github.com/</li>
<li>Vagrant Cloud - https://app.vagrantup.com</li>
</ul>

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

<p>&#8226; Git</p>
<p>Установим Git:</p>

<pre>sudo yum -y install git</pre>

<p>После успешного окончания будет установлен Git.</p>

<h4>**Клонирование и запуск**</h4>

<p>Для запуска рабочего виртуального окружения необходимо зайти через браузер в GitHub под своей учетной записью и выполнить `fork` данного репозитория: https://github.com/dmitry-lyutenko/manual_kernel_update</p>

<p>После этого данный репозиторий необходимо склонировать к себе на рабочую машину. Для этого воспользуемся ранее установленным приложением `git`, при этом в `<user_name>` будет имя уже моего репозитрия:</p>





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

<h4>**kernel update**</h4>

<p>Подключаем репозиторий, откуда возьмем необходимую версию ядра:</p>

<pre>[root@kernelupdate ~]# yum -y install http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
elrepo-release-7.0-3.el7.elrepo.noarch.rpm               | 8.5 kB     00:00     
Examining /var/tmp/yum-root-5WdsAi/elrepo-release-7.0-3.el7.elrepo.noarch.rpm: elrepo-release-7.0-3.el7.elrepo.noarch
Marking /var/tmp/yum-root-5WdsAi/elrepo-release-7.0-3.el7.elrepo.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package elrepo-release.noarch 0:7.0-3.el7.elrepo will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package  Arch   Version          Repository                               Size
================================================================================
Installing:
 elrepo-release
          noarch 7.0-3.el7.elrepo /elrepo-release-7.0-3.el7.elrepo.noarch 5.2 k

Transaction Summary
================================================================================
Install  1 Package

Total size: 5.2 k
Installed size: 5.2 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : elrepo-release-7.0-3.el7.elrepo.noarch                       1/1 
  Verifying  : elrepo-release-7.0-3.el7.elrepo.noarch                       1/1 

Installed:
  elrepo-release.noarch 0:7.0-3.el7.elrepo                                      

Complete!
[root@kernelupdate ~]#</pre>

<p>В репозитории есть две версии ядер **kernel-ml** и **kernel-lt**. Первая является наиболее свежей стабильной версией, вторая это стабильная версия с длительной поддержкой, но менее свежая, чем первая. В данном случае ядро 5й версии будет в  **kernel-ml**.</p>

<p>Поскольку мы ставим ядро из репозитория, то установка ядра похожа на установку любого другого пакета, но потребует явного включения репозитория при помощи ключа ```--enablerepo```.</p>

<p>Ставим последнее ядро:</p>

<pre>[root@kernelupdate ~]# yum --enablerepo elrepo-kernel install kernel-ml -y
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.surf
 * elrepo: mirrors.colocall.net
 * elrepo-kernel: mirrors.colocall.net
 * extras: mirror.surf
 * updates: mirror.surf
base                                                     | 3.6 kB     00:00     
elrepo                                                   | 3.0 kB     00:00     
elrepo-kernel                                            | 3.0 kB     00:00     
extras                                                   | 2.9 kB     00:00     
updates                                                  | 2.9 kB     00:00     
(1/6): base/7/x86_64/group_gz                              | 153 kB   00:00     
(2/6): extras/7/x86_64/primary_db                          | 247 kB   00:00     
(3/6): base/7/x86_64/primary_db                            | 6.1 MB   00:02     
(4/6): updates/7/x86_64/primary_db                         |  16 MB   00:02     
(5/6): elrepo/primary_db                                   | 580 kB   00:02     
(6/6): elrepo-kernel/primary_db                            | 2.1 MB   00:02     
Resolving Dependencies
--> Running transaction check
---> Package kernel-ml.x86_64 0:5.18.6-1.el7.elrepo will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package        Arch        Version                    Repository          Size
================================================================================
Installing:
 kernel-ml      x86_64      5.18.6-1.el7.elrepo        elrepo-kernel       56 M

Transaction Summary
================================================================================
Install  1 Package

Total download size: 56 M
Installed size: 257 M
Downloading packages:
warning: /var/cache/yum/x86_64/7/elrepo-kernel/packages/kernel-ml-5.18.6-1.el7.elrepo.x86_64.rpm: Header V4 DSA/SHA256 Signature, key ID baadae52: NOKEY
Public key for kernel-ml-5.18.6-1.el7.elrepo.x86_64.rpm is not installed
kernel-ml-5.18.6-1.el7.elrepo.x86_64.rpm                   |  56 MB   00:18     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
Importing GPG key 0xBAADAE52:
 Userid     : "elrepo.org (RPM Signing Key for elrepo.org) <secure@elrepo.org>"
 Fingerprint: 96c0 104f 6315 4731 1e0b b1ae 309b c305 baad ae52
 Package    : elrepo-release-7.0-3.el7.elrepo.noarch (@/elrepo-release-7.0-3.el7.elrepo.noarch)
 From       : /etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : kernel-ml-5.18.6-1.el7.elrepo.x86_64                         1/1 
  Verifying  : kernel-ml-5.18.6-1.el7.elrepo.x86_64                         1/1 

Installed:
  kernel-ml.x86_64 0:5.18.6-1.el7.elrepo                                        

Complete!
[root@kernelupdate ~]#</pre>

<h4>**grub update**</h4>

<p>После успешной установки нам необходимо сказать системе, что при загрузке нужно использовать новое ядро. В случае обновления ядра на рабочих серверах необходимо перезагрузиться с новым ядром, выбрав его при загрузке. И только при успешно прошедших загрузке нового ядра и тестах сервера переходить к загрузке с новым ядром по-умолчанию. В тестовой среде можно обойти данный этап и сразу назначить новое ядро по-умолчанию.</p>

<p>Обновляем конфигурацию загрузчика:</p>

<pre>[root@kernelupdate ~]# grub2-mkconfig -o /boot/grub2/grub.cfg 
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.18.6-1.el7.elrepo.x86_64
Found initrd image: /boot/initramfs-5.18.6-1.el7.elrepo.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1127.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1127.el7.x86_64.img
done
[root@kernelupdate ~]#</pre>

<p>Выбираем загрузку с новым ядром по-умолчанию:</p>

<pre>[root@kernelupdate ~]# grub2-set-default 0
[root@kernelupdate ~]#</pre>

<p>Перезагружаем виртуальную машину:</p>

<pre>[root@kernelupdate ~]# shutdown -r now
Connection to 127.0.0.1 closed by remote host.
Connection to 127.0.0.1 closed.
[user@localhost kernelupdate]$</pre>

<p>После перезагрузки виртуальной машины заходим в нее:</p>

<pre>[user@localhost kernelupdate]$ vagrant ssh
Last login: Sat Jun 25 11:28:00 2022 from 10.0.2.2
[vagrant@kernelupdate ~]$</pre>

<p>Снова определяем версию ядра:</p>

<pre>[vagrant@kernelupdate ~]$ uname -r 
5.18.6-1.el7.elrepo.x86_64
[vagrant@kernelupdate ~]$</pre>

<p>Как видим, ядро обновилось до версии 5.</p>



