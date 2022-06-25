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

<p>Выходим из системы командой CTRL+D</p>

<pre>[vagrant@kernel-update ~]$ logout
Connection to 127.0.0.1 closed.
[user@localhost manual_kernel_update]$</pre>

<h4>**Packer**</h4>

<p>Теперь необходимо создать свой образ системы, с уже установленым ядром 5й версии. Для это воспользуемся ранее установленной утилитой `packer`. В директории `packer` есть все необходимые настройки и скрипты для создания необходимого образа системы.</p>

<h4>**packer provision config**</h4>

<p>Файл `centos.json` содержит описание того, как произвольный образ. Полное описание можно найти в документации к `packer`. Обратим внимание на основные секции или ключи.<br />
Создаем переменные (`variables`) с версией и названием нашего проекта (artifact):</p>

<pre>[user@localhost manual_kernel_update]$ cd ./packer/
[user@localhost packer]$ vi ./centos.json</pre>

<pre>...
  "variables": {
    "artifact_description": "CentOS 7.9 with kernel 5.x",
    "artifact_version": "7.9.2009",
    "image_name": "centos-7.9"
  }
  ...</pre>

<p>В секции `builders` задаем исходный образ, для создания своего в виде ссылки и контрольной суммы. Параметры подключения к создаваемой виртуальной машине.В конце В конце добавим строку "headless": true, чтобы не возникали ошибки в консольной (без графики) версии.</p>

<pre>...
  "builders": [
    {
  ...
      "iso_checksum": "sha256:07b94e6b1a0b0260b94c83d6bb76b26bf7a310dc78d7a9c7432809fb9bc6194a",
      "iso_url": "https://mirror.yandex.ru/centos/7.9.2009/isos/x86_64/CentOS-7-x86_64-Minimal-2009.iso",
  ...
      "vm_name": "packer-centos-vm",
      "headless": true,
    }
  ],
...</pre>

<p>В секции `provisioners` указываем каким образом и какие действия необходимо произвести для настройки виртуальой машины. Именно в этой секции мы и обновим ядро системы, чтобы можно было получить образ с 5й версией ядра. Настройка системы выполняется несколькими скриптами, заданными в секции `scripts`.</p>

<pre>...
  "provisioners": [
  ...
          "scripts": [
            "scripts/stage-1-kernel-update.sh",
            "scripts/stage-2-clean.sh"
          ]
  ...
  ],

...</pre>

<p>Секция `post-processors` описывает постобработку виртуальной машины при ее выгрузке. Мы указыаем имя файла, в который будет сохранен результат (artifact). Обратите внимание, что имя задается на основе ранее созданной пользовательской переменной `artifact_version` значение которой мы задали ранее:</p>

<pre>...
  "post-processors": [
    {
      "compression_level": "7",
      "output": "centos-{{user `artifact_version`}}-kernel-5-x86_64-Minimal.box",
      "type": "vagrant"
    }
  ],

  ...</pre>

<p>Содержимое скрипта stage-1-kernel-update.sh</p>

<pre>[user@localhost packer]$ cat ./scripts/stage-1-kernel-update.sh 
#!/bin/bash

# Install elrepo
yum install -y http://www.elrepo.org/elrepo-release-7.0-5.el7.elrepo.noarch.rpm
# Install new kernel
yum --enablerepo elrepo-kernel install kernel-ml -y
# Remove older kernels (Only for demo! Not Production!)
rm -f /boot/*3.10*
# Update GRUB
grub2-mkconfig -o /boot/grub2/grub.cfg
grub2-set-default 0
echo "Grub update done."
# Reboot VM
shutdown -r now
[user@localhost packer]$</pre>

<p>Содержимое скрипта stage-1-kernel-update.sh</p>

<pre>[user@localhost packer]$ cat ./scripts/stage-2-clean.sh 
#!/bin/bash

# clean all
yum update -y
yum clean all


# Install vagrant default key
mkdir -pm 700 /home/vagrant/.ssh
curl -sL https://raw.githubusercontent.com/mitchellh/vagrant/master/keys/vagrant.pub -o /home/vagrant/.ssh/authorized_keys
chmod 0600 /home/vagrant/.ssh/authorized_keys
chown -R vagrant:vagrant /home/vagrant/.ssh


# Remove temporary files
rm -rf /tmp/*
rm  -f /var/log/wtmp /var/log/btmp
rm -rf /var/cache/* /usr/share/doc/*
rm -rf /var/cache/yum
rm -rf /vagrant/home/*.iso
rm  -f ~/.bash_history
history -c

rm -rf /run/log/journal/*

# Fill zeros all empty space
dd if=/dev/zero of=/EMPTY bs=1M
rm -f /EMPTY
sync
grub2-set-default 1
echo "###   Hi from secone stage" >> /boot/grub2/grub.cfg
[user@localhost packer]$</pre>

<h4>**packer build**</h4>

<p>Для создания образа системы достаточно перейти в директорию `packer` и в ней выполнить команду:</p>

<pre>[user@localhost packer]$ packer build ./centos.json
...
Build 'centos-7.9' finished after 15 minutes 19 seconds.

==> Wait completed after 15 minutes 19 seconds

==> Builds finished. The artifacts of successful builds are:
--> centos-7.9: 'virtualbox' provider box: centos-7.9.2009-kernel-5-x86_64-Minimal.box
[user@localhost packer]$</pre>

<p>Если все в порядке, то, согласно файла `config.json` будет скачан исходный iso-образ CentOS, установлен на виртуальную машину в автоматическом режиме, обновлено ядро и осуществлен экспорт в указанный нами файл. Если не вносилось изменений в предложенные файлы, то в текущей директории мы увидим файл `centos-7.9.2009-kernel-5-x86_64-Minimal.box`. Он и является результатом работы `packer`.</p>

<pre>[user@localhost packer]$ ls -l
total 832352
-rw-rw-r--. 1 user user 852323872 июн 25 22:03 centos-7.9.2009-kernel-5-x86_64-Minimal.box
-rw-rw-r--. 1 user user      2038 июн 25 21:37 centos.json
drwxrwxr-x. 2 user user        24 июн 25 16:01 http
drwxrwxr-x. 2 user user        62 июн 25 17:46 scripts
[user@localhost packer]$</pre>

<h4>**vagrant init (тестирование)**</h4>

<p>Проведем тестирование созданного образа. Выполним его импорт в `vagrant`:</p>

<pre>[user@localhost packer]$ vagrant box add centos-7-9 ./centos-7.9.2009-kernel-5-x86_64-Minimal.box 
==> box: Box file was not detected as metadata. Adding it directly...
==> box: Adding box 'centos-7-9' (v0) for provider: 
    box: Unpacking necessary files from: file:///home/user/otus/manual_kernel_update/packer/centos-7.9.2009-kernel-5-x86_64-Minimal.box
==> box: Successfully added box 'centos-7-9' (v0) for 'virtualbox'!
[user@localhost packer]$</pre>

<p>Проверим его в списке имеющихся образов:</p>

<pre>[user@localhost packer]$ vagrant box list
centos-7-9  (virtualbox, 0)
[user@localhost packer]$</pre>

<p>Он будет называться `centos-7-9`, данное имя мы задали при помощи параметра `name` при импорте.</p>

<p>Теперь необходимо провести тестирование полученного образа. Для этого создадим новый Vagrantfile или воспользуемся имеющимся. Для нового создадим директорию `test` и перейдём в неё:</p>

<pre>[user@localhost otus]$ mkdir ./test/
[user@localhost otus]$ cd ./test/
[user@localhost test]$</pre>

<p>Скопируем туда имеющийся Vagrantfile:</p>

<pre>[user@localhost test]$ cp ../manual_kernel_update/Vagrantfile .
[user@localhost test]$ ls -l
total 4
-rw-rw-r--. 1 user user 1335 июн 25 23:23 Vagrantfile
[user@localhost test]$</pre>

<p>Для имеющегося Vagrantfile произведем замену значения `box_name` на имя импортированного образа. Соотвествующая строка примет вид:</p>

<pre>...
              :box_name => "centos-7-9",
...</pre>

<p>Теперь запустим виртуальную машину:</p>

<pre>[user@localhost test]$ vagrant up
Bringing machine 'kernel-update' up with 'virtualbox' provider...
==> kernel-update: Importing base box 'centos-7-9'...
==> kernel-update: Matching MAC address for NAT networking...
==> kernel-update: Setting the name of the VM: test_kernel-update_1656189042721_88755
==> kernel-update: Fixed port collision for 22 => 2222. Now on port 2201.
==> kernel-update: Clearing any previously set network interfaces...
==> kernel-update: Preparing network interfaces based on configuration...
    kernel-update: Adapter 1: nat
==> kernel-update: Forwarding ports...
    kernel-update: 22 (guest) => 2201 (host) (adapter 1)
==> kernel-update: Running 'pre-boot' VM customizations...
==> kernel-update: Booting VM...
==> kernel-update: Waiting for machine to boot. This may take a few minutes...
    kernel-update: SSH address: 127.0.0.1:2201
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
[user@localhost test]$</pre>

<p>Подключимся к ней:</p>

<pre>[user@localhost test]$ vagrant ssh
Last login: Sat Jun 25 19:01:09 2022 from 10.0.2.2
[vagrant@kernel-update ~]$</pre>

<p>Проверим, что у нас в ней новое ядро:</p>

<pre>[vagrant@kernel-update ~]$ uname -r
5.18.7-1.el7.elrepo.x86_64
[vagrant@kernel-update ~]$</pre>

<p>Машина запустилась и загрузилась с новым ядром. В данном случае это `5.18.7`.</p>

<p>Удалим тестовый образ из локального хранилища:</p>

<pre>[user@localhost test]$ vagrant box remove centos-7-9 -f
Removing box 'centos-7-9' (v0) with provider 'virtualbox'...
[user@localhost test]$</pre>

<h4>**Vagrant cloud**</h4>

<p>Поделимся полученным образом с сообществом. Для этого зальем его в Vagrant Cloud. Можно залить через web-интерфейс, но так же `vagrant` позволяет это проделать через CLI.</p>

<p>Вернемся в директорию, где находится наш образ centos-7.9.2009-kernel-5-x86_64-Minimal.box:</p>

<pre>[user@localhost test]$ cd ../manual_kernel_update/packer/
[user@localhost packer]$</pre>

<p>Логинимся в `vagrant cloud`, указывая e-mail, пароль и описание выданого токена (можно оставить по-умолчанию)</p>

<pre>[user@localhost packer]$ vagrant cloud auth login
In a moment we will ask for your username and password to HashiCorp's
Vagrant Cloud. After authenticating, we will store an access token locally on
disk. Your login details will be transmitted over a secure connection, and
are never stored on disk locally.

If you do not have an Vagrant Cloud account, sign up at
https://www.vagrantcloud.com

Vagrant Cloud username or email: sergsha
Password (will be hidden): 
Token description (Defaults to "Vagrant login from localhost.localdomain"): 
You are now logged in.
[user@localhost packer]$</pre>

<p>Теперь публикуем полученный бокс,<br />
где:<br />
 - `cloud publish` - загрузить образ в облако;<br />
 - `--no-private` - НЕприватный:<br />
 - `release` - указывает на необходимость публикации образа после загрузки;<br />
 - `sergsha/centos-7-9` - `username`, указаный при публикации и имя образа;<br />
 - `1.0` - версия образа;<br />
 - `virtualbox` - провайдер;<br />
 - `centos-7.9.2009-kernel-5-x86_64-Minimal.box` - имя файла загружаемого образа</p>

<pre>[user@localhost packer]$ vagrant cloud publish --no-private --release sergsha/centos-7-9 1.0 virtualbox ./centos-7.9.2009-kernel-5-x86_64-Minimal.box 
You are about to publish a box on Vagrant Cloud with the following options:
sergsha/centos-7-9:   (v1.0) for provider 'virtualbox'
Automatic Release:     true
Do you wish to continue? [y/N]y
Saving box information...
Uploading provider with file /home/user/otus/manual_kernel_update/packer/centos-7.9.2009-kernel-5-x86_64-Minimal.box
Releasing box...
Complete! Published sergsha/centos-7-9
Box:              sergsha/centos-7-9
Description:      
Private:          no
Created:          2022-06-26T00:02:46.016+03:00
Updated:          2022-06-26T00:02:46.016+03:00
Current Version:  N/A
Versions:         1.0
Downloads:        0
[user@localhost packer]$</pre>

![image](https://user-images.githubusercontent.com/96518320/175790808-637ea942-e4a0-4ea5-b563-1803bf269554.png)

![image](https://user-images.githubusercontent.com/96518320/175790834-a52d1bce-ec99-4b15-bad8-4d99b2c4b36e.png)

<p>В результате создан и загружен в `vagrant cloud` образ виртуальной машины.</p>

<p>В Vagrantfile произведем замену значения `box_name` на `sergsha/centos-7-9`. Соотвествующая строка примет вид:</p>

<pre>...
              :box_name => "sergsha/centos-7-9",
...</pre>
