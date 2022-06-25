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

<pre>[user@localhost otus]$ git clone git@github.com:SergSha/manual_kernel_update.git
Cloning into 'manual_kernel_update'...
remote: Enumerating objects: 43, done.
remote: Counting objects: 100% (19/19), done.
remote: Compressing objects: 100% (16/16), done.
remote: Total 43 (delta 6), reused 3 (delta 3), pack-reused 24
Receiving objects: 100% (43/43), 20.82 KiB | 0 bytes/s, done.
Resolving deltas: 100% (10/10), done.
[user@localhost otus]$</pre>

<p>В текущей директории появится папка с именем репозитория. В данном случае `manual_kernel_update`. Ознакомимся с содержимым:</p>

<pre>[user@localhost otus]$ cd ./manual_kernel_update/
[user@localhost manual_kernel_update]$ ls -l
total 20
drwxrwxr-x. 2 user user    23 июн 25 16:01 manual
drwxrwxr-x. 4 user user    52 июн 25 16:01 packer
-rw-rw-r--. 1 user user 15771 июн 25 16:01 README.md
-rw-rw-r--. 1 user user  1335 июн 25 16:01 Vagrantfile
[user@localhost manual_kernel_update]$</pre>

<p>Здесь:<br />
- `manual` - директория с данным руководством<br />
- `packer` - директория со скриптами для `packer`'а<br />
- `Vagrantfile` - файл описывающий виртуальную инфраструктуру для `Vagrant`</p>

<p>Содержимое Vagrantfile:</p>

<pre>[user@localhost manual_kernel_update]$ cat Vagrantfile 
# Describe VMs
MACHINES = {
  # VM name "kernel update"
  :"kernel-update" => {
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
end
[user@localhost manual_kernel_update]$</pre>

<p>Запустим систему:</p>

<pre>[user@localhost manual_kernel_update]$ vagrant up
Bringing machine 'kernel-update' up with 'virtualbox' provider...
==> kernel-update: Importing base box 'centos/7'...
==> kernel-update: Matching MAC address for NAT networking...
==> kernel-update: Checking if box 'centos/7' version '2004.01' is up to date...
==> kernel-update: There was a problem while downloading the metadata for your box
==> kernel-update: to check for updates. This is not an error, since it is usually due
==> kernel-update: to temporary network problems. This is just a warning. The problem
==> kernel-update: encountered was:
==> kernel-update: 
==> kernel-update: The requested URL returned error: 404
==> kernel-update: 
==> kernel-update: If you want to check for box updates, verify your network connection
==> kernel-update: is valid and try again.
==> kernel-update: Setting the name of the VM: manual_kernel_update_kernel-update_1656162923648_65537
==> kernel-update: Fixed port collision for 22 => 2222. Now on port 2200.
==> kernel-update: Clearing any previously set network interfaces...
==> kernel-update: Preparing network interfaces based on configuration...
    kernel-update: Adapter 1: nat
==> kernel-update: Forwarding ports...
    kernel-update: 22 (guest) => 2200 (host) (adapter 1)
==> kernel-update: Running 'pre-boot' VM customizations...
==> kernel-update: Booting VM...
==> kernel-update: Waiting for machine to boot. This may take a few minutes...
    kernel-update: SSH address: 127.0.0.1:2200
    kernel-update: SSH username: vagrant
    kernel-update: SSH auth method: private key
    kernel-update: 
    kernel-update: Vagrant insecure key detected. Vagrant will automatically replace
    kernel-update: this with a newly generated keypair for better security.
    kernel-update: 
    kernel-update: Inserting generated public key within guest...
    kernel-update: Removing insecure key from the guest if it's present...
    kernel-update: Key inserted! Disconnecting and reconnecting using new SSH key...
==> kernel-update: Machine booted and ready!
==> kernel-update: Checking for guest additions in VM...
    kernel-update: No guest additions were detected on the base box for this VM! Guest
    kernel-update: additions are required for forwarded ports, shared folders, host only
    kernel-update: networking, and more. If SSH fails on this machine, please install
    kernel-update: the guest additions and repackage the box to continue.
    kernel-update: 
    kernel-update: This is not an error message; everything may continue to work properly,
    kernel-update: in which case you may ignore this message.
==> kernel-update: Setting hostname...
[user@localhost manual_kernel_update]$</pre>

<p>Войдём в неё с помощью SSH:</p>

<pre>[user@localhost manual_kernel_update]$ vagrant ssh
[vagrant@kernel-update ~]$</pre>

<p>Заходим под правами root:</p>

<pre>[vagrant@kernel-update ~]$ sudo -i
[root@kernel-update ~]#</pre>

<p>Определим версию ядра:</p>

<pre>[root@kernel-update ~]# uname -r
3.10.0-1127.el7.x86_64
[root@kernel-update ~]#</pre>

<p>Теперь приступим к обновлению ядра.</p>

<h4>**kernel update**</h4>

<p>Подключаем репозиторий, откуда возьмем необходимую версию ядра:</p>

<pre>[root@kernel-update ~]# yum -y install http://www.elrepo.org/elrepo-release-7.0-5.el7.elrepo.noarch.rpm
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
elrepo-release-7.0-5.el7.elrepo.noarch.rpm               | 8.6 kB     00:00     
Examining /var/tmp/yum-root-ryjycS/elrepo-release-7.0-5.el7.elrepo.noarch.rpm: elrepo-release-7.0-5.el7.elrepo.noarch
Marking /var/tmp/yum-root-ryjycS/elrepo-release-7.0-5.el7.elrepo.noarch.rpm to be installed
Resolving Dependencies
--> Running transaction check
---> Package elrepo-release.noarch 0:7.0-5.el7.elrepo will be installed
--> Finished Dependency Resolution

Dependencies Resolved

================================================================================
 Package  Arch   Version          Repository                               Size
================================================================================
Installing:
 elrepo-release
          noarch 7.0-5.el7.elrepo /elrepo-release-7.0-5.el7.elrepo.noarch 5.0 k

Transaction Summary
================================================================================
Install  1 Package

Total size: 5.0 k
Installed size: 5.0 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  Installing : elrepo-release-7.0-5.el7.elrepo.noarch                       1/1 
  Verifying  : elrepo-release-7.0-5.el7.elrepo.noarch                       1/1 

Installed:
  elrepo-release.noarch 0:7.0-5.el7.elrepo                                      

Complete!
[root@kernel-update ~]#</pre>

<p>В репозитории есть две версии ядер **kernel-ml** и **kernel-lt**. Первая является наиболее свежей стабильной версией, вторая это стабильная версия с длительной поддержкой, но менее свежая, чем первая. В данном случае ядро 5й версии будет в  **kernel-ml**.</p>

<p>Поскольку мы ставим ядро из репозитория, то установка ядра похожа на установку любого другого пакета, но потребует явного включения репозитория при помощи ключа ```--enablerepo```.</p>

<p>Ставим последнее ядро:</p>

<pre>[root@kernel-update ~]# yum --enablerepo elrepo-kernel install kernel-ml -y
Failed to set locale, defaulting to C
Loaded plugins: fastestmirror
Determining fastest mirrors
 * base: mirror.docker.ru
 * elrepo: ftp.nluug.nl
 * elrepo-kernel: ftp.nluug.nl
 * extras: mirror.corbina.net
 * updates: mirror.corbina.net
base                                                     | 3.6 kB     00:00     
elrepo                                                   | 3.0 kB     00:00     
elrepo-kernel                                            | 3.0 kB     00:00     
extras                                                   | 2.9 kB     00:00     
updates                                                  | 2.9 kB     00:00     
(1/6): base/7/x86_64/group_gz                              | 153 kB   00:00     
(2/6): extras/7/x86_64/primary_db                          | 247 kB   00:00     
(3/6): elrepo/primary_db                                   | 580 kB   00:00     
(4/6): base/7/x86_64/primary_db                            | 6.1 MB   00:01     
(5/6): elrepo-kernel/primary_db                            | 2.1 MB   00:02     
(6/6): updates/7/x86_64/primary_db                         |  16 MB   00:02     
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
kernel-ml-5.18.6-1.el7.elrepo.x86_64.rpm                   |  56 MB   00:26     
Retrieving key from file:///etc/pki/rpm-gpg/RPM-GPG-KEY-elrepo.org
Importing GPG key 0xBAADAE52:
 Userid     : "elrepo.org (RPM Signing Key for elrepo.org) <secure@elrepo.org>"
 Fingerprint: 96c0 104f 6315 4731 1e0b b1ae 309b c305 baad ae52
 Package    : elrepo-release-7.0-5.el7.elrepo.noarch (@/elrepo-release-7.0-5.el7.elrepo.noarch)
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
[root@kernel-update ~]#</pre>

<h4>**grub update**</h4>

<p>После успешной установки нам необходимо сказать системе, что при загрузке нужно использовать новое ядро. В случае обновления ядра на рабочих серверах необходимо перезагрузиться с новым ядром, выбрав его при загрузке. И только при успешно прошедших загрузке нового ядра и тестах сервера переходить к загрузке с новым ядром по-умолчанию. В тестовой среде можно обойти данный этап и сразу назначить новое ядро по-умолчанию.</p>

<p>Обновляем конфигурацию загрузчика:</p>

<pre>[root@kernel-update ~]# grub2-mkconfig -o /boot/grub2/grub.cfg
Generating grub configuration file ...
Found linux image: /boot/vmlinuz-5.18.6-1.el7.elrepo.x86_64
Found initrd image: /boot/initramfs-5.18.6-1.el7.elrepo.x86_64.img
Found linux image: /boot/vmlinuz-3.10.0-1127.el7.x86_64
Found initrd image: /boot/initramfs-3.10.0-1127.el7.x86_64.img
done
[root@kernel-update ~]#</pre>

<p>Выбираем загрузку с новым ядром по-умолчанию:</p>

<pre>[root@kernel-update ~]# grub2-set-default 0
[root@kernel-update ~]#</pre>

<p>Перезагружаем виртуальную машину:</p>

<pre>[root@kernel-update ~]# shutdown -r now
Connection to 127.0.0.1 closed by remote host.
Connection to 127.0.0.1 closed.
[user@localhost manual_kernel_update]$</pre>

<p>После перезагрузки виртуальной машины заходим в нее:</p>

<pre>[user@localhost manual_kernel_update]$ vagrant ssh
Last login: Sat Jun 25 13:17:02 2022 from 10.0.2.2
[vagrant@kernel-update ~]$</pre>

<p>Снова определяем версию ядра:</p>

<pre>[vagrant@kernel-update ~]$ uname -r
5.18.6-1.el7.elrepo.x86_64
[vagrant@kernel-update ~]$</pre>

<p>Как видим, ядро обновилось до версии 5.</p>



