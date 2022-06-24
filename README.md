<h3>### Kernel Update ###</h3>

<h4>Описание домашнего задания</h4>

<p>Обновить ядро в базовой системе.</p>
<br />

<h4># Обновить ядро в базовой системе.</h4>

<p><b>- Установка ПО.</b></p>

<p>В домашней директории создадим директорию kernel_update, в которой будут храниться настройки виртуальной машины:</p>

<pre>[user@localhost]$ mkdir ./kernel_update
[user@localhost otus]$</pre>

<p>Перейдём в директорию systemd:</p>

<pre>[user@localhost otus]$ cd ./kernel_update/
[user@localhost kernel_update]$</pre>

<p></p>

<p>Создадим файл Vagrantfile:</p>

<pre>[user@localhost systemd]$ vi ./Vagrantfile</pre>

<p>Заполним следующим содержимым:</p>

<pre># Describe VMs
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
end</pre>

<p>Запустим систему:</p>

<pre>[user@localhost systemd]$ vagrant up</pre>

<p>и войдём в неё:</p>

<pre>[user@localhost systemd]$ vagrant ssh
[vagrant@systemd ~]$</pre>

<p>Заходим под правами root:</p>

<pre>[vagrant@systemd ~]$ sudo -i
[root@systemd ~]#</pre>

<p>Для начала создаём файл с конфигурацией для сервиса в директории /etc/sysconfig - из неё сервис будет брать необходимые переменные:</p>

<pre>[root@systemd ~]# vi /etc/sysconfig/watchlog</pre>
