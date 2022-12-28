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

```
python3 -m venv /usr/lib/ckan/default
. /usr/lib/ckan/default/bin/activate
pip install setuptools==44.1.0
pip install --upgrade pip
```

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
- Instalação CKAN versão 2.10-dev:

```
pip install -e git+https://github.com/ckan/ckan.git@dev-v2.10#egg=ckan[requirements]
deactivate
. /usr/lib/ckan/default/bin/activate
```

- Postgres:

```
sudo service postgresql start
sudo su - postgres
createuser -S -D -R -P ckan_default
# irá solicitar senha, utilizamos: ckan_default
createdb -O ckan_default ckan_default -E utf-8

# Para sair do modo interativo postgres:
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

- Setup Solr

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

- Alterar o schema do core ckan

```
cp /usr/lib/ckan/default/src/ckan/ckan/config/solr/schema.xml solr-9.1.0/server/solr/ckan/conf/managed-schema.xml
```

- Reiniciar o Solr

```
./solr-9.1.0/bin/solr restart
```

- Inicializar Redis

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
ckan -c /etc/ckan/default/ckan.ini sysadmin add admin
```

- Instalar o plugin datapackage-creator

```
pip install ckanext-datapackage-creator
vi /etc/ckan/default/ckan.ini
```

Alterar a configuração `ckan.plugins` adicionando no fim o seguinte:

```
ckan.plugins = ... datapackage_creator
```

- Crie as tabelas do datapackage_creator

```
ckan db upgrade -p datapackage_creator
```

Para testar basta rodar:

```
ckan -c /etc/ckan/default/ckan.ini run
```

- Instale o supervisor

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
command=/usr/lib/ckan/default/bin/ckan -c /etc/ckan/default/ckan.ini run
directory=/usr/lib/ckan/default/src/ckan
user=codespace
stdout_logfile=/usr/lib/ckan/default/gunicorn_supervisor_ckan.log
redirect_stderr=true
```

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

- Reinicie o nginx

```
sudo service nginx restart
```







