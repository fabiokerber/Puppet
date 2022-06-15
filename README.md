# Puppet v7.17.0

Resource Types:<br>
- Package
    - Provider: yum, apt, ...
- User
    - Provider: useradd, userdel, ...
- Group
- Service
- File
<br />
Exemplo:<br>

```
package { 'php5':
    ensure => installed,
}

user { 'bob':
    ensure => present,
}
```

**RESOURCES**

Exemplo utilizando o recurso "user".<br>
Executado no agent.<br>
```
# puppet resource user bob
```

O resultado será "absent" (pois o usuário bob não existe), e o provider utilizado por este recurso.<br>

<br />

Exemplo adicionando o usuario bob.<br>
```
# puppet resource user bob ensure=present
```
```
# puppet resource user bob

user { 'bob':
  ensure             => 'present',
  gid                => 1002,
  home               => '/home/bob',
  password           => '!',
  password_max_age   => 99999,
  password_min_age   => 0,
  password_warn_days => 7,
  provider           => 'useradd',
  shell              => '/bin/sh',
  uid                => 1002,
}
```

Para acessar documentação de um determinado "resource", utilize o describe.
```
# puppet describe user | more
```

Alterarando o uid do usuario bob.<br>
```
# puppet resource user bob ensure=present uid=9999
```

Lista completa de todos os "resources".<br>
```
# puppet describe --list
```

Criando grupo e adicionando o usuario bob ao grupo.<br>
```
# puppet resource group sysadmins ensure=present
# puppet resource user bob ensure=present groups=sysadmins
```

**PUPPET DSL (DOMAIN SPECIFIC LANGUAGE)**

DSL é composto de três componentes:<br>
- "Resource Type"
- "Name of the Resource"
- "Attributes to Manage"
<br />

Basic Syntax:<br>
```
type { 'title':
    attribute => 'value',
    attribute => 'value',
    attribute => 'value',
}

user { 'bob':
    ensure => present,
    uid => '9999',
    groups => 'sysadmins',
}
```

**MANIFESTS**

"Manifests" são arquivos com extensões .pp<br>
Deve-se utilizar o comando "puppet apply" para executar o que está descrito no arquivo .pp<br>

Exemplo bob.pp com dois "resources" group e user:<br>
```
group { 'sysadmins':
    ensure => present,
}

user { 'bob':
    ensure => present,
    uid => '9999',
    groups => 'sysadmins',
}
```
```
# puppet apply bob.pp
```

**CLASSES**

É possível definir uma classe para configurar vários recursos de uma vez. Exemplo é possível agrupar a configuração do package, file, services, user em uma unidade e dar nome de webserver.<br>
Esta classe pode ser definida à um node.<br>
Para que esta classe "funcione" a mesma deve ser declarada junto ao módulo.<br>

Exemplo de definição de uma classe:<br>
```
class sysadmins {

    group { 'sysadmins':
        ensure => present,
    }

    user { 'bob':
        ensure => present,
        uid => '9999',
        groups => 'sysadmins',
    }

}
```

**MODULES**

As classes devem estar junto aos módulos.<br>
Os módulos também provisionam arquivos e extensões.<br>
Há um layout de estrutura de pastas e arquivos que devem estar abaixo do modulo (modulepath).<br>

Comando para listar os módulos:<br>
```
# puppet module list
```

Para descobri o modulepath, é necessário executar o comando abaixo:<br>
```
# puppet config print modulepath
# puppet config print (geral)
```

Neste exemplo o deverá constar uma pasta "webserver" em: <br>
modulepath = /etc/puppetlabs/code/environments/production/modules<br>
<br />
Dentro da pasta criamos uma outra chamada "manifests" então inserimos o arquivo .pp que consta a classe.<br>
Todo módulo possuí uma classe "base" que deve ser chamada de "init.pp"<br>

Recapitulando:<br>

O arquivo init.pp que contém a classe webserver:<br>
```
class sysadmins {

    group { 'sysadmins':
        ensure => present,
    }

    user { 'bob':
        ensure => present,
        uid => '9999',
        groups => 'sysadmins',
    }

    user { 'susan':
        ensure => present,
        uid => '9998',
        groups => 'sysadmins',
    }

    user { 'peter':
        ensure => present,
        uid => '9997',
        groups => 'sysadmins',
    }

}
```

Após criada a estrutura de pasta sysadmins e adicionado o init.pp com a classe, deve ser realizado o comando com o parâmetro que indica qual módulo/classe base deseja executar:<br>

```
# puppet apply -e 'include sysadmins'
# puppet module list
```

<kbd>
    <img src="https://github.com/fabiokerber/Puppet/blob/main/img/120620222059.png">
</kbd>
<br />
<br />

**SERVER & AGENT**

Por default o puppet agent se comunica com o server de 30 em 30 minutos.<br>
O agente se conecta com o servidor via conexão SSL, baixa as configurações e as aplica.<br>
Puppet server é um CA.<br>
Quando o puppet agent executa pela primeira vez, é gerado uma chave SSL e solicita o certificado ao servidor.<br>

Puppet agent:<br>
```
# puppet agent -t (executa o sincronismo com o servidor e cria o certificado na primeira execução)
# puppet agent --noop (compara o catálogo local com o do servidor, mas não aplica nada)
```

Puppet server:<br>
```
# /opt/puppetlabs/server/bin/puppetserver ca list (lista certificados pendentes para assinatura)
# /opt/puppetlabs/server/bin/puppetserver ca sign --certname agent.localdomain (autoriza o certificado para agent.localdomain)
# /opt/puppetlabs/server/bin/puppetserver ca list --all (lista todos os certificados)
```

Puppet agent:<br>
```
# puppet agent -t (realiza a comunicação com o server)
```

**COMMON SSL ISSUES**

Servidor e agentes devem estar com o horário sincronizado.<br>
Certificados válidos entre servidor e agentes (caso o agente seja reinstalado, necessário verificar certificados).<br>
Se necessário os certificados devem ser removidos no servidor e agente.<br>

**REMOVE AGENT & SERVER CERTIFICATES**

Puppet server:<br>
```
# /opt/puppetlabs/server/bin/puppetserver ca clean --certname agent.localdomain (remove o certificado agent.localdomain)
```

Puppet agent:<br>
```
# puppet config print ssldir (exibe o path dos certificados ssl)
# rm -rf /etc/puppetlabs/puppet/ssl/* (remove todos os certificados)
# puppet agent -t (será gerado um novo certificado SSL)
```

Puppet server:<br>
```
# /opt/puppetlabs/server/bin/puppetserver ca list (lista certificados pendentes para assinatura)
# /opt/puppetlabs/server/bin/puppetserver ca sign --certname agent.localdomain (autoriza o certificado para agent.localdomain)
```

Puppet agent:<br>
```
# puppet agent -t (realiza a comunicação com o server)
```

**PUPPET RUN**
<kbd>
    <img src="https://github.com/fabiokerber/Puppet/blob/main/img/130620221013.png">
</kbd>
<br />
<br />

**FACTER**

Coleta uma variedade grande de informações sobre o node e os envia a cada execução do agente.

Puppet agent:<br>
```
# facter (exibe todos os dados que são enviados ao puppet server)
# facter operatingsystem (exibe informações sobre o sistema operacional)
```

**CATALOG**

Contém todos os recursos gerenciados e o estado que eles devem estar.

**CLASSIFICATION**

Quando há a necessidade de aplicar uma classe à um node, chama-se classificação.

**NODE DEFINITION**

Inclusão de classes a um determinado node.<br>

Exemplos:<br>
```
node "agent.localdomain" {
    include webserver
    include database
}

node /*.localdomain/ {
    include webserver
    include database
}
```

Exemplo caso não entre em nenhum case acima:
```
node default {
    include webserver
    include database
}
```

**APPLY CONFIG TO NODE**

Execute o comando abaixo para listar os módulos já configurados.<br>
Deve listar o sysadmins.<br>
Puppet server:<br>
```
# puppet module list
```

Para incluir os módulos aos nodes deve-se configurar os manifestos no path "manifests".
Execute o comando abaixo para listar o path:<br>
Puppet server:<br>
```
# puppet config print manifest
```

Crie o arquivo site.pp no path informado conforme abaixo:<br>
```
node "agent.localdomain" {
    include sysadmins
}
```

Recapitulando:<br>

- Criado o módulo "sysadmins" (init.pp)<br>
- Criado o manifesto "node agent.localdomain" (site.pp)<br>
<kbd>
    <img src="https://github.com/fabiokerber/Puppet/blob/main/img/130620221043.png">
</kbd>
<br />
<br />

**MORE RESOURCES**

install.sh
```
mkdir -p /opt/myapp
```

test.pp:<br>
```
exec { 'install application':
    path => '/usr/local/bin:/usr/bin:/bin', (procura o install.sh nessas pastas)
    command => 'install.sh',
    unless => 'test -d /opt/myapp', (só ira executar caso o retorno seja 0, ou seja, não exista a pasta)  
}
```

Outro exemplo, podendo variar os providers.<br>
mysql.pp:<br>
```
package { 'mysql':
    ensure => installed,
    provider => 'gem',
}

package { 'rpm mysql':
    ensure => installed,
    name => 'mysql',
}
```

**FILE RESOURCE TYPE**
Gerenciar arquivos, links simbolicos ou diretórios.
Manter modelo no servidor e toda vez que o puppet for executado, e por algum motivo o arquivo alterado o puppet já atualiza conforme o modelo do servidor.
Exemplo:<br>
```
file { '/etc/httpd/httpd.conf':
    ensure => file,
    owner => 'root',
    group => 'root',
    mode => '0644',
}

file { '/etc/motd':
    ensure => file,
    content => 'Welcome to my system',
    owner => 'root',
    group => 'root',
    mode => '0644',
}

file { '/etc/httpd/httpd.conf':
    ensure => file,
    source => 'puppet:///modules/apache/httpd.conf',
    owner => 'root',
    group => 'root',
    mode => '0644',
    replace => false, (com este replace, o puppet verifica se o arquivo existe, se não existir faz a cópia e se existir e estiver diferente do source, o puppet não altera nada do conteúdo e verifica apenas o owner, group e mode)
}

file { '/root/settings':
    ensure => directory,
}

URI: puppet://<hostname>/<mountpoint>/<path>
Obs: com três barras assume-se que o arquivo está no hostname <puppet_server> utilizado no momento da execução.
```

**RELATIONSHIPS**
<kbd>
    <img src="https://github.com/fabiokerber/Puppet/blob/main/img/150620221018.png">
</kbd>
<br />
<br />

**RESOURCE ORDERING**
Configurar dependência entre "resources".
Exemplo:<br>
```
package { 'httpd':
    ensure => installed,
}

service { 'httpd':
    ensure => running,
    require => Package['httpd'],
}

=

package { 'httpd':
    ensure => installed,
    before => Service['httpd'], (executa a instalação antes de assegurar que esteja em running)
}

service { 'httpd':
    ensure => running,
}
```

**TRIGGERING EVENTS**
Executa o "notify" quando há alteração no status do serviço.<br>
```
service { 'tinpot':
    ensure => running,
    enable => true,
    notify => Exec['clean tinpot cache'],
}

exec { 'clean tinpot cache':
    path => '/opt/tinpot/bin',
    command => 'tinpot --cleancache',
    refreshonly => true,
}
```

**RESOURCE CHAINING**
"Fixa" (->) uma sequência de execução de recurso após recurso.<br>
```
package { 'httpd':
    ensure => installed,
} ->
file { '/etc/httpd/httpd.conf':
    ensure => file,
    source => 'puppet:///modules/apache/httpd.conf'
}
```