<h1 align="center"><i>Модуль С: Автоматизация развёртывания приложения</i></h1>
<i>В рамках данного задания по автоматизации развёртывания приложения от Вас требуется создание сценария  настройки инфраструктуры с использованием программного обеспечения Ansible. Конечной целья является функционирование обозначенных сервисов при развёртывании вашего сценария на эталонном стенде.</i>
<p></p>
<p></p>
<p>В вашем распоряжении тестовый стенд состоящий из 4 виртуальных маштн под управлением Ubuntu 20.04: Project_1, Project_2, Project_3, Project_Deployer.</p>

<p>Исходные файлы размещены в Git-репозитории: </p>

<p align="center"><strong><u>https://github.com/Jeniston14/wsr-skillcloud</U></strong></p>
<p> Обратите внимание, что проверка будет производиться путём клонирования вашего репозитория на проверочный стенд, с последующим запуском playbook-сценария. Продумайте то, что необходимо для его выполнения. </p>
<p>После выполнения задания, вам требуется собрать все необходимые файлы, который необходимы для выполнения первой части задания и отправить плейбук с зависимостями на свой GitHub аккаунт.</p>
<p>Создайте в вашем репозитории файл read.me и укажите дополнительную информацию по запуску вашего плейбука. Реализация второй части задания допустима на текущей инфраструктуре, но никто не возражает в качестве её автоматизации.</p>
<h1>Автоматизация базовой конфигурации виртуальных машин:</h1>

<p>В первой части вам необходимо реализовать Ansible playbook для автоматизации базовой конфигурации виртуальных машин. Контролирубщим является Project_deployer.</p>
<p>Выполните клонирование Git-репозитория в корневую директорию узла Project_Deployer.</p>
<p></p>
<p>Ansible playbook должен выполнять следующие задачи конфигурации:</p>

<ul>
    <li><strong>Обновить списки пакетов, выполнить установку curl.</strong></li>
</ul>
    
    - name: Update package lists, perform curl installation
      apt: 
        name: 'curl'
        state: latest
        update_cache: true
        
<ul>
    <li><strong>Активировать межсетевой экран UFW. Разрешить порты TCP 80,8080,1834</strong></li>
</ul>
    
    - name: Activate the UFW firewall, allow TCP ports 80,8080,1834
      ufw:
        rule: allow
        port: '{{ item }}'
        proto: tcp 
        state: enabled
      loop: 
        - '22'
        - '80'
        - '8080'
        - '1834'
<i> Так же необходимо разрешить 22 порт, чтобы не потерять доступ к виртуальным машинам на момент выполнения сценария! </i>
<ul>
    <li><strong>Изменить конфигурационный файл SSH.</strong></li>
    <ul type="circle">
<li>Порт подключения должен быть 1834</li>

    - name: Change the SSH configuration file | Connection port should be 1834
      lineinfile:
        dest: "/etc/ssh/sshd_config"
        regexp: "^Port"
        line: 'Port 1834'

<li>Разрешить авторизацию по публичному ключу</li>
</ul> 

    - name: Change SSH configuration file | Allow public key authorization
      lineinfile:
        dest: "/etc/ssh/sshd_config"
        line: '{{ item }}'
      loop: 
        - RSAAuthentication yes
        - PubkeyAuthentication yes
        - AuthorizedKeysFile     %h/.ssh/authorized_keys
</ul>

<ul>
    <li><strong>Создать пользователей для доступа к виртуальным машинам. Для каждого пользователя необходимо разместить публичный ключ</strong></li>
</ul>
<i>Исходя из соображений безопасности, файл со списком пользователей, паролей и публичными ключами расположенный в директории Users/privileges.yml Git-репозитория, зашифрован при помощи Ansible Vault. Используйте пароль gK2VEOxxEK9n для дешифрования файла.</i>

<p># ansible-vault decrypt Users/privileges.yml   - команда для дешифрования файла</p>

    - name: Create users to access virtual machines
      ansible.builtin.user:
        name: "{{ item.username }}"
        password: "{{ item.userpass }}"
      loop:
         - { username: 'Webdeveloper', userpass: 'S52we9V6QTp7' }
         - { username: 'Devopsengineer', userpass: 'dHy6sKGHsj2T' }
         - { username: 'Projectmanager', userpass: 'oP92ugMSaCbe' }
         
    - name: A public key must be placed for each user
      authorized_key:
        user: "{{ item }}"
        state: present
        key: "{{ id_rsa_pub }}"
      loop: 
        - Webdeveloper
        - Devopsengineer
        - Projectmanager
      vars:
        id_rsa_pub: "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCovvBkeLIDsvbCyQsMWtPWVgGVKcwAqRONiBJ9JyrVCQbruyMPutatjSlhNpYXKLlP4BXHrGrmAVqeI155li1fkNP5Il7viHRE0HvA3M2extNGDDCiX5f5OlIeT9p/D9OHvWWozLjN2NAGEW24feuzKPZb6Kyv2W3yHbiIU3wt8v50VAIA2+PAfElHp1jplGHQLmYuT6Cc26Pn3WYXZ8t8oU77T6Ki5qDG5V5DVZI3Ym5gqqXXtJYzET9piJvO6qiIcgljtOGlUH9H9QNLEbuF+RKIhL3pFAnF8S79Km2A3j9KFZw6prDR6/VeMffMrNSZLeYztzDGEzm35uz5q6j+qrKsuA4SfpbSOcBwaariOoKpb6JfogoJRgCqxR5O1AKR/Oqhdk6JOlKJk+tIXFmOczH7da/W93f8KGGve4iHRvz/e3vYA7exXIVkD8mc/VmIoT1kqh/uNGia/adnyCgvMpL8JXLJgY2DThpjHslUr0RTEpQJTLTka3D43YV37kM=
<ul>
    <li><strong>Выполнить установку пакета Docker</strong></li>
</ul>        

    - name: Install the Docker package | Install aptitude
      apt:
        name: aptitude
        state: latest
        update_cache: true

    - name: Install the Docker package | Install required system packages
      apt:
        pkg:
          - apt-transport-https
          - ca-certificates
          - curl
          - software-properties-common
          - python3-pip
          - virtualenv
          - python3-setuptools
        state: latest
        update_cache: true

    - name: Install the Docker package | Add Docker GPG apt Key
      apt_key:
        url: https://download.docker.com/linux/ubuntu/gpg
        state: present

    - name: Install the Docker package | Add Docker Repository
      apt_repository:
        repo: deb https://download.docker.com/linux/ubuntu focal stable
        state: present

    - name: Install the Docker package | Update apt and install docker-ce
      apt:
        name: docker-ce
        state: latest
        update_cache: true

    - name: Install the Docker package | Install Docker Module for Python
      pip:
        name: docker
        
 <ul>
    <li><strong>Запустить скрипт установки docker-compose</strong></li>
</ul>   

    - name: Run the docker-compose installation script
      script: ./wsr-skillcloud/docker-compose.sh  
      
 <ul>
    <li><strong>Создайте директорию /skillcloud-nginx, скопируйте файлы необходимые для сборки образов, а также index.html и nginx.conf файлы</strong></li>
</ul>
<i>Поскольку в дальнейшем планируется автоматизация развёртывания приложения, то необходимо зарание заполнить файл "nginx.conf" ip-адресами виртуальных машин. И создать необходимую структуру каталогов в директории /skillcloud-nginx</i>

    - name: Create a directory /skillcloud-nginx
      file:
        path: /skillcloud-nginx
        state: directory

    - name: Copying files needed to build images
      copy:
        src: ./wsr-skillcloud/docker-compose.yml
        dest: /skillcloud-nginx

    - name: Create a directory /skillcloud-nginx/{site,balance}
      file:
        path: /skillcloud-nginx/{{ item }}
        state: directory
      loop:
        - site
        - balance

    - name: Copying files needed to build images
      copy:
        src: ./wsr-skillcloud/{{ item.src }}
        dest: /skillcloud-nginx/{{ item.dest }}/Dockerfile
      loop:
         - { src: Dockerfile-site, dest: site }
         - { src: Dockerfile-balance, dest: balance }
         

    - name: Copying files needed to build images
      copy:
        src: ./wsr-skillcloud/{{ item.src }}
        dest: /skillcloud-nginx/{{ item.dest }}
      loop:
        - { src: index.html, dest: site}
        - { src: nginx.conf, dest: balance }


<ul>
    <li><strong>Перезапустить SSH и UFW</strong></li>
</ul>
<i>Внимание! Ранее порт для подключения по SSH был поменен на 1834, поэтому после перезапуска службы SSH доступ будет потерян. Что бы этого недотустить необходимо после перезагрузки службы явным образом указать новый порт, через set_fact </i>


    - name: Restart UFW 
      service: 
        name: ufw
        state: restarted 

    - name: Restart SSH 
      service: 
        name: sshd 
        state: restarted 

    - name: Change ssh port to 1834
      set_fact:
        ansible_port: 1834

<ul>
    <li><strong>Выполнить перезагрузку виртуальных машин</strong></li>
</ul>


    - name: Reboot virtual machines
      reboot:
      
      
<h1>Развёртывание отказоустойчивости веб-сервера на базе Nginx: </h1>
<i>Допускается ручное выполнения данной части задания, посредством сборки docker образов из Dockerfile, а также запуск docker контейнеров при помощи docker-compose</i>
<p><strong>Но никто не запрещает автоматизировать данный процесс.</strong></p>
<p></p>
<p></p>
<ul>
    <li><strong>Выполнить сборку образов на виртуальных машинах, используя Dockerfile в директории skillcloud-nginx.</strong></li>

<ul type="circle">
<li>Задайте теги при сборке образов, site:site и balance:balance</li>
</ul>
</ul>
<i>Для автоматизации данной части задания создаётся второй сценарий, поэтому необходимо явно указать порт для подключения по ssh,а так же для работы с модулем docker_image указать интерпретатор python</i>


    - name: Change ssh port to 1834
      set_fact:
        ansible_port: 1834

    - name: Change ansible_python_interpreter 
      set_fact:
        ansible_python_interpreter: /usr/bin/python3

    - name: Build images on a VM using Dockerfile in the skillcloud-nsinx directory
      docker_image:
        name: '{{ item.name }}'
        build:
          path: /skillcloud-nginx/{{ item.path }}
      loop: 
        - { name: site, path: site }
        - { name: balance, path: balance }


<ul>
    <li><strong>Запустить docker-compose.yml для сборки контейнеров.</strong></li>
</ul>
<i>Для работы модуля docker_compose необходимо установить docker-compose для python. Ранее в первом сценарии была выполнена установка пакета "pip", поэтому выполнить можно черег него.</i>


    - name: Pip install docker-compose
      pip: 
        name: docker-compose

    - name: Launching docker-compose.yml for container assembly
      docker_compose:
        project_src: /skillcloud-nginx
        files:
          - docker-compose.yml 

<h2>Таким образом, было продемонстрировано задание с последующим его выполнение и разбором основных моментов.</h2>
<p></p>
<p>В качестве демонстрации выполнения задания "Модуля С: Автоматизация развёртывания приложения." посредством полной автоматизации, необходимо склонировать данный репозиторий на эталонный стенд и выполнить ряд шагов по запуску сценариев:  </p>

<p><strong>Шаг 1. склонировать репозиторий: https://github.com/NewErr0r/Automating_application_deployment.git </strong><p>
<p># git clone https://github.com/NewErr0r/Automating_application_deployment.git </p>

<p><strong>Шаг 2. Отредактировать файлы "hosts" и "nginx.conf" - указать ip-адреса виртуальных машин эталонного стенда: </strong><p>
<p># cd Automating_application_deployment</p>
<p># vi hosts</p>
<p></p>
[app]<br>
Project1 ansible_ssh_host={{ IP_address_project_1 }}<br>
Project2 ansible_ssh_host={{ IP_address_project_2 }}<br>
Project3 ansible_ssh_host={{ IP_address_project_3 }}<br>
<p></p>
<p># vi wsr-skillcloud/nginx.conf</p>

    server {{ IP_address_project_1 }}:8080;
    server {{ IP_address_project_2 }}:8080;
    server {{ IP_address_project_3 }}:8080;

<p><strong>Шаг 3. Выполнить запуск сценария "Автоматизации базовой конфигурации виртуальных машин" </strong><p>
<p># ansible-playbook playbook_based_configurations_vm.yaml</p>
<i> Дождаться окончания выполнения сценария! </i>

<p><strong>Шаг 4. Выполнить запуск сценария "Развёртывание отказоустойчивости веб-сервера на базе Nginx" </strong><p>
<p># ansible-playbook playbook_deploy_web_server.yaml</p>
