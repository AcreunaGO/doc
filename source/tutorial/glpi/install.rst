Instalação do GLPI 9.4.4
========================

Esta documentação apresenta as instruções de instalação do GLPI.

Atualizar do CentOS 7
~~~~~~~~~~~~~~~~~~~~~

.. note::

Antes de começar a instalação é recomendado manter os pacotes do OS atualizados

.. prompt:: bash #

    yum update -y

Instalar o servidor web Apache
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. prompt:: bash #
	
	yum install httpd -y

Crie os seguintes diretorios para armazenar os virtual hosts

.. prompt:: bash #
	mkdir /etc/httpd/sites-available /etc/httpd/sites-enabled

Informe ao Apache para procurar por virtual hosts no diretório `sites-enabled`. Para isso edite o arquivo de configuração ´rincipal do Apache e adicione uma linha declarando um diretório opcional para arquivos de configuração adicional:

.. prompt:: bash #
	nano /etc/httpd/conf/httpd.conf

Adicione esta linha ao final do arquivo:

.. code-block:: ini
	
	IncludeOptional sites-enabled/*.conf

Salve e feche o arquivo quando terminar de adicionar essa linha.

Crie um novo arquivo no diretório `sites-available`:

.. prompt:: bash #
	
	nano /etc/httpd/sites-available/glpi.conf

Adicione o seguinte bloco de configuração:

.. code-block:: ini
	
	<Directory "/var/www/html/glpi">    
       AllowOverride All
	</Directory>

Salve e feche o arquivo quando terminar

Habilitar o Apache para usar os virtual hosts. Para isso, crie um link simbólico para cada virtual host no diretório `sites-enabled`:

.. prompt:: bash #
	
	ln -s /etc/httpd/sites-available/glpi.conf /etc/httpd/sites-enabled/glpi.conf 

Para inicialização do serviço

.. prompt:: bash #
	
	systemctl start httpd.service

Para fazer com que o serviço inicie com o Linux

.. prompt:: bash #
	
	systemctl enable httpd.service

Para verificar se o serviço está funcionando

.. prompt:: bash #
	
	systemctl status httpd

Caso seja necessário libera a porta 80 no firewall

.. prompt:: bash #
	
	firewall-cmd --permanent --add-port=80/tcp

Recarregue as o firewall para que as altrações surtam efeito

.. prompt:: bash #
	
	firewall-cmd --reload

Instalar o PHP
~~~~~~~~~~~~~~

.. note::

	Recomendamos usar a versão mais recente do PHP para melhores performances.

Adicionar dependências do repositorio
	
.. prompt:: bash #

	yum install epel-release yum-utils -y

Adicionar repositorio do PHP

.. prompt:: bash #

	yum install http://rpms.remirepo.net/enterprise/remi-release-7.rpm
	yum-config-manager --enable remi-php70 (configurando repositório do 7.0)
	yum-config-manager --enable remi-php71 (configurando repositório do 7.1)
	yum-config-manager --enable remi-php72 (configurando repositório do 7.2)
	yum-config-manager --enable remi-php73 (configurando repositório do 7.3)
	
Instalar pacotes do PHP

.. prompt:: bash #

	yum install php php-common php-opcache php-mcrypt php-cli php-gd php-curl php-mysql php-ldap php-imap php-pecl-apcu php-xmlrpc php-pear-CAS php-mbstring php-simplexml -y

Verificar a versão do PHP instalada

.. prompt:: bash #

	php -v

Extensões obrigatórias
~~~~~~~~~~~~~~~~~~~~~~

As seguintes extensões do PHP são necessárias para que o aplicativo funcione corretamente:

* ``curl``: for CAS authentication, GLPI version check, Telemetry, ...;
* ``fileinfo``: to get extra informations on files;
* ``gd``: to generate images;
* ``json``: to get support for JSON data format;
* ``mbstring``:  to manage multi bytes characters;
* ``mysqli``: to connect and query the database;
* ``session``: to get user sessions support;
* ``zlib``: to get backup and restore database functions;
* ``simplexml``;
* ``xml``.

Configuração
~~~~~~~~~~~~

As seguintes variáveis devem ser alteradas no arquivo de configuração do PHP (``php.ini``)

.. prompt:: bash #

	nano /etc/php.ini

.. code-block:: ini

    memory_limit = 64M ;        // max memory limit
    file_uploads = on ;
    max_execution_time = 600 ;  // not mandatory but recommended
    register_globals = off ;    // not mandatory but recommended
    magic_quotes_sybase = off ;
    session.auto_start = off ;
    session.use_trans_sid = 0 ; // not mandatory but recommended
	
Instalar o Bando de Dados MariaDB 10.4
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Crie um arquivo de repositorio chamado `MariaDB.repo` e coloque o conteúdo a seguir:

.. prompt:: bash #
	nano /etc/yum.repos.d/MariaDB.repo
	
.. code-block:: ini
	# MariaDB 10.4 CentOS repository list - created 2019-10-07 13:38 UTC
	# http://downloads.mariadb.org/mariadb/repositories/
	[mariadb]
	name = MariaDB
	baseurl = http://yum.mariadb.org/10.4/centos7-amd64
	gpgkey=https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
	gpgcheck=1

Instale os pacotes do MariaDB server e client usando `yum`:

.. prompt:: bash #

	yum install MariaDB-server MariaDB-client -y

Yum lhe perguntará se deseja importar o MariaDB GPG key:

.. code-block:: ini

	Recuperando chave de https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
	Importing GPG key 0x1BB943DB:
	Userid     : "MariaDB Package Signing Key <package-signing-key@mariadb.org>"
	Fingerprint: 1993 69e5 404b d5fc 7d2f e43b cbcb 082a 1bb9 43db
	From       : https://yum.mariadb.org/RPM-GPG-KEY-MariaDB
	Correto? [s/N]:
	
Digite `s` e tecle `Enter`

Para iniciar e/ou parar o serviço

.. prompt:: bash #

	systemctl start mariadb
	systemctl stop mariadb

Para fazer com que o serviço inicie com o Linux

.. prompt:: bash #

	systemctl enable mariadb

Para verificar o status do serviço:

.. prompt:: bash #

	systemctl status mariadb
	
.. prompt:: bash #

	mysql_secure_installation

.. note::
	
	Aqui serão requisitados alguns parametros conforme segue abaixo
	
.. prompt:: bash
	
	Enter current password for root (enter for none): -> Pressione enter para continuar
	Set root password? [Y/n] -> Pressione Y para definir uma senha
	New password: -> Insira a nova senha, pressione Enter
	Re-enter new password: -> Insira a nova senha, pressione Enter
	Remove anonymous users? [Y/n] -> Pressione Y para remover o usuário anonimo de teste
	Disallow root login remotely? [Y/n] -> Pressione Y para desabilitar acesso remoto com usuario root
	Remove test database and access to it? [Y/n] -> Pressione Y para remover bando de dados teste
	Reload privilege tables now? [Y/n] -> Pressione Y para atualizar as permissões
	
Criar base de dados e usuario GLPI para acesso ao sistema
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Acesse o banco de dados, informando a senha definida anteriormente quando solicitado

.. prompt:: bash #

	mysql -u root -p

.. code:: sql

	mysql> CREATE USER 'glpi'@'localhost'IDENTIFIED BY 'glpi';

	mysql> create database glpi character set utf8 collate utf8_bin;
	mysql> grant all privileges on glpi.* to glpi@localhost identified by 'glpi';

	mysql> flush privileges;

	mysql> quit

Baixando e instalando GLPI
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. prompt:: bash #
	
	yum install net-tools vim wget -y

.. prompt:: bash #
	
	cd /var/www/html/
	wget -c https://github.com/glpi-project/glpi/releases/download/9.4.4/glpi-9.4.4.tgz
	tar -xvzf glpi-9.4.4.tgz

Configuração de permissão das pastas

.. prompt:: bash #

	chown apache:apache -Rf  /var/www/html/glpi
	chmod 775 -Rf /var/www/html/glpi -Rf

Reiniciar servidor web

.. prompt:: bash #

	systemctl restart httpd

Agora vamos desabilitar o SeLinux do servidor. Para isso, digite em seu console esse comando:
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. prompt:: bash #

	nano /etc/selinux/config

Depois, modifique a linha SELINUX=enforcing para SELINUX=disabled:

.. code::
	
	SELINUX = disable

Para finalizar será necessário reinicar o servidor para que todas alterações surtam efeito, para isso digite o comando a seguir:

.. prompt:: bash #

	shutdown -r now

Depois da instalação
~~~~~~~~~~~~~~~~~~~~

Adicionar o bloco de código nas configurações de virtual (host ou no arquivo `glpi/install/.htaccess`:
	
.. prompt:: bash #
		
	nano /var/www/html/glpi/install/.htaccess
	
.. code-block:: ini
	
	<IfModule mod_authz_core.c>
		Require local
	</IfModule>
	<IfModule !mod_authz_core.c>
		order deny, allow
		deny from all
		allow from 127.0.0.1
		allow from ::1
	</IfModule>
	ErrorDocument 403 "<p><b>&Aacute;rea restrita.</b><br />N&atilde;o &eacute; permitido acesso a essa p&aacute;gina.</p>"

Atualize as permissão das pastas

.. prompt:: bash #

	chown apache:apache -Rf  /var/www/html/glpi
	chmod 775 -Rf /var/www/html/glpi -Rf

Altere o nome da pasta `install`:
	
.. prompt:: bash #
	
	mv /var/www/html/glpi/install /var/www/html/glpi/install-old

Desativar os usuários criados automáticamente pelo sistema e/ou alterar as senhas.
	
.. history
.. authors
.. license