# Passos para instalação CKAN via Código Fonte

Passos seguidos conforme [documentação do CKAN](https://docs.ckan.org/en/2.9/maintaining/installing/install-from-source.html#installing-ckan-from-source).

- [Install the required packages](https://docs.ckan.org/en/2.9/maintaining/installing/install-from-source.html#install-the-required-packages):
  - Documentação não orienta rodar código `apt-get update`, 

```
sudo apt-get update
sudo apt-get install python3-dev postgresql libpq-dev python3-pip python3-venv git-core solr-jetty openjdk-8-jdk redis-server
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








