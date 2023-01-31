# Passos para instalação CKAN via Código Fonte

Passos seguidos conforme [documentação do CKAN](https://docs.ckan.org/en/2.9/maintaining/installing/install-from-source.html#installing-ckan-from-source).

- [Install the required packages](https://docs.ckan.org/en/2.9/maintaining/installing/install-from-source.html#install-the-required-packages):
  - Documentação não orienta rodar código `apt-get update`, 

```
sudo apt-get update
sudo apt-get install python3-dev postgresql libpq-dev python3-pip python3-venv git-core openjdk-8-jdk redis-server
```

- [Install CKAN into a Python virtual environment](https://docs.ckan.org/en/2.9/maintaining/installing/install-from-source.html#install-ckan-into-a-python-virtual-environment):

```
mkdir -p ~/ckan/lib
sudo ln -s ~/ckan/lib /usr/lib/ckan
mkdir -p ~/ckan/etc
sudo ln -s ~/ckan/etc /etc/ckan

# Create a Virtual Env
sudo mkdir -p /usr/lib/ckan/default
sudo chown `whoami` /usr/lib/ckan/default

# Check python version
python3 --version
```

- Como minha versão python instalada era 3.10 segui os seguintes passos para instalação desta versão utilizando pyenv:

```
# Install the required dependencies for pyenv
sudo apt-get update
sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev xz-utils tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev

# Install pyenv using the official installation script
curl https://pyenv.run | bash

# Configure your shell to use pyenv
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init -)"

# Restart your shell to apply the changes
source ~/.bashrc

# Use pyenv to install a version of Python and set it as global
pyenv install 3.6
pyenv global 3.6

# Check python3 version
python3 --version
```
> Use pyenv to manage your Python versions. You can use the pyenv versions command to list all of the Python versions installed on your system, and the pyenv which command to see which version is currently being used. You can also use the pyenv local command to set the Python version for a specific project, and the pyenv shell command to temporarily switch to a different Python version for the current shell session.

- Após a instalação do python3.6 continuei seguindo a documentação para finalizar instalação do ambiente virtual e pacotes necessários:

* entre o install e o setuptools = 

````
pip install --proxy http://<ip_usuariocamg>:<porta>
````
(igual ao proxy do git config)

```
python3 -m venv /usr/lib/ckan/default
. /usr/lib/ckan/default/bin/activate
pip install setuptools==44.1.0
pip install --upgrade pip
```
* WARNING (proxy?) = retrying/NewConnectionError urllib3 (msg para Diogo)

* versão Ubuntu = 20.04.5 LTS (Focal Fossa)



> You can tell when the virtualenv is active because its name appears in front of your shell prompt

- `pip list`:

```
(default) @gabrielbdornas ➜ ~ $ pip list
Package    Version
---------- -------
pip        21.3.1
setuptools 44.1.0
```

- Após conversa com Gileno resolvemos manter o setup com a versão python3.10.

# Instalação CKAN versão 2.10-dev:

* configuração global do proxy para o git:
````
$ git config --global http://<ip_usuariocamg>:<porta>
$ git config --list
````

* configurar o proxy na máquina virtual:
````
export HTTP_PROXY=user:pass@my.proxy.server:8080
source ~/.bash_profile
echo $HTTP_PROXY
````

* ativação do ambiente
```
deactivate
. /usr/lib/ckan/default/bin/activate
```

* clonar o repositório, criar pasta, passar para a branch dev e instalar dependências e ckan via setup.py:
````
$ git clone https://github.com/ckan/ckan.git ckan_source
$ cd ckan_source/
$ git checkout dev-v2.10
$ pip install - r requirements.txt
$ python setup.py install
````

# Postgres:

```
sudo service postgresql start
sudo su - postgres
createuser -S -D -R -P ckan_default
# irá solicitar senha, utilizamos: ckan_default
createdb -O ckan_default ckan_default -E utf-8

- Para sair do modo interativo postgres:
exit
```

- Create a CKAN config file:

```
sudo mkdir -p /etc/ckan/default
sudo chown -R `whoami` /etc/ckan/
```

- Create the CKAN config file:

```
ckan generate config /etc/ckan/default/ckan.ini
```

# Setup Solr

Baixar a versão binária: [https://solr.apache.org/downloads.html](https://solr.apache.org/downloads.html)

```
wget https://www.apache.org/dyn/closer.lua/solr/solr/9.1.0/solr-9.1.0.tgz?action=download
tar -xvzf 'solr-9.1.0.tgz?action=download'
```

- Criar um novo solr core

```
./solr-9.1.0/bin/solr start
./solr-9.1.0/bin/solr create -c ckan
```
* tornar `of` o proxy dentro do arquivo de configuração, pois ele já está no global da máquina virtual:
````
$sudo vi /etc/wgetrc
````
teste para ver se está rodando
````
$ wget http://:localhost:8983/solr
$ cat ...
````

**caso apareça a msg de erro abaixo no start do solr** 
````
# Your current version of Java is too old to run this version of Solr.
# We found major version 8, using command 'java -version', with response:
# openjdk version "1.8.0_352"
# OpenJDK Runtime Environment (build 1.8.0_352-8u352-ga-1~18.04-b08)
# OpenJDK 64-Bit Server VM (build 25.352-b08, mixed mode)

Please install latest version of Java 11 or set JAVA_HOME properly`:
````

**comandos para atualizar o JAVA*

````
$ sudo apt update
$ sudo apt install default-jdk
$ sudo apt update
$ sudo apt install default-jre
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt update
$ sudo apt install oracle-java11-installer
# https://phoenixnap.com/kb/how-to-install-java-ubuntu
# usar o comando abaixo se o último acima não funcionar (`$ Package oracle-java11-installer is not available, but is referred to by another package.
This may mean that the package is missing, has been obsoleted, or is only available from another source`)
$ sudo apt-get install openjdk-11-jdk
# https://stackoverflow.com/questions/52504825/how-to-install-jdk-11-under-ubuntu
````

- Alterar o schema do core ckan

```
cp /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema.xml solr-9.1.0/server/solr/ckan/conf/managed-schema.xml
```

- Reiniciar o Solr

```
./solr-9.1.0/bin/solr restart
```

# Inicializar Redis

```
sudo service redis-server start
```

- Configurar o banco dados no ckan.ini

```
cd /usr/lib/ckan/default/src/ckan
vi /etc/ckan/default/ckan.ini
```

Alterar a linha com o `sqlalchemy.url` para ficar como abaixo:

```
sqlalchemy.url = postgresql://ckan_default:ckan_default@localhost/ckan_default
```

- Criar as tabelas

```
ckan -c /etc/ckan/default/ckan.ini db init
```

* workaround = tirar o proxy para rodar a database no ckan.ini (provavelmente, o ckan run do supervisor tb tem que tira o proxy):

````
$ NO_PROXY="*" ckan -c /etc/ckan/default/ckan.ini db init
````

- Criar pasta storage e alterar o ckan.ini

```
mkdir /etc/ckan/default/storage
vi /etc/ckan/default/ckan.ini
```

Alterar a senha `ckan.storage_path` para ficar assim:

```
ckan.storage_path = /etc/ckan/default/storage
```

- Criar um usuário administrador:

```
$ NO_PROXY="*" ckan -c /etc/ckan/default/ckan.ini db init
$ ckan -c /etc/ckan/default/ckan.ini sysadmin add admin
```

# Instalar o plugin datapackage-creator

```
pip install --proxy http://<ip_usuariocamg>:<porta> ckanext-datapackage-creator
vi /etc/ckan/default/ckan.ini
```

Alterar a configuração `ckan.plugins` adicionando no fim o seguinte:

```
ckan.plugins = ... datapackage_creator
datapackage_creator = /etc/ckan/default/datapackage_creator.json
```

* incluir arquivos de configuração: [vide repositório do plugin](https://github.com/transparencia-mg/ckanext-datapackage-creator#datapackage-creator-configuration)

The plugin allows you to configure which fields of the resource and package are mandatory and/or 'readonly', for this you just need to add a configuration in your INI file.

datapackage_creator = /path/to/datapackage_creator.json
We suggest that the file path would be /etc/ckan/default/datapackage_creator.json or in the same folder as ckan.ini file.

Configuration example file:

{
    "package": {
        "required": [],
        "readonly": []
    },
    "resource": {
        "required": [],
        "readonly": []
    }
}

- Crie as tabelas do datapackage_creator

```
$ NO_PROXY="*" ckan -c /etc/ckan/default/ckan.ini db init
$ ckan -c /etc/ckan/default/ckan.ini datapackage-creator-init-db
```

Para testar basta rodar:

```
ckan -c /etc/ckan/default/ckan.ini run
```

# Instale o supervisor

```
sudo apt install supervisor
```
`
- Configurar o supervisor

```
sudo vi /etc/supervisor/conf.d/ckan.conf
```

Colocar o seguinte conteúdo neste arquivo:

```
[program:ckan]
command= usr/lib/ckan/default/bin/ckan -c /etc/ckan/default/ckan.ini run
environment=NO_PROXY="*"
directory=/usr/lib/ckan/default/src/ckan
user=<user>
stdout_logfile=/usr/lib/ckan/default/gunicorn_supervisor_ckan.log
redirect_stderr=true
```
**se utilizando o Vagrant**:

````
[program:ckan]
command=/usr/lib/ckan/default/bin/ckan -c /etc/ckan/default/ckan.ini run
environment=NO_PROXY="*"
directory=/usr/lib/ckan/default/src/ckan
user=vagrant
stdout_logfile=/usr/lib/ckan/default/gunicorn_supervisor_ckan.log
redirect_stderr=true
````
Em seguida se faz necessário inicializar o supervisor e carregar o processo do ckan

```
sudo service supervisor start
sudo supervisorctl reread
sudo supervisorctl reload
```

- Instale e configure o nginx

```
sudo apt install nginx
sudo rm /etc/nginx/sites-enabled/default
sudo vi /etc/nginx/sites-enabled/ckan.conf
```

Neste arquivo do nginx coloque:

```
server {
    listen 80;
    server_name projetockan.cge.mg.gov.br;
    client_max_body_size 200m;
    access_log /usr/lib/ckan/default/src/ckan/nginx-access-ckan.log;
    error_log /usr/lib/ckan/default/src/ckan/nginx-error-ckan.log;
    location / {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        if (!-f $request_filename) {
            proxy_pass http://127.0.0.1:5000;
            break;
        }
    }
}
```
- COLOCAR DOMÍNIO NO ckan.ini
````
$ sudo vi /etc/ckan/default/ckan.ini
ckan.site_url = http://projetockan.cge.mg.gov.br/dataset/new
````
-  para conferir se o NO_PROXY saiu:

````
$ sudo supervisorctl status
````
depois
````
$ sudo supervisorctl reload
$ sudo supervisorctl status
$ sudo supervisorctl restart ckan
````



- para reiniciar o nginx (se mudar o arquivo de configuração dele)

```
sudo service nginx restart
```

## Reiniciar a máquina

```
# Ativar ambiente
. /usr/lib/ckan/default/bin/activate

# Atualizar pacote ckanext-datapackage-creator (caso necessário)
# Atualiza para versão mais atual disponível no Pypi 
pip install --proxy http://<ip_usuariocamg>:<porta> -U ckanext-datapackage-creator

# Iniciar postgres
sudo service postgresql start

# Iniciar Solr
./solr-9.1.0/bin/solr start

# Iniciar Redis
sudo service redis-server start

# Iniciar Supervisor
sudo service supervisor start

# Reiniciar CKAN (Caso alguma biblioteca seja atualizada/modificada no ambiente)
sudo supervisorctl restart ckan

# Iniciar nginx
sudo service nginx start
```

# when the CSRF Token is missing

## CSRF Protection
WTF_CSRF_ENABLED = false




