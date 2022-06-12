# Puppet

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
Esta "class" pode ser definida à um node.<br>
Para que esta "class" "funcione" a mesma deve ser declarada junto ao módulo.<br>

Exemplo de definição de um "class":<br>
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

As "class" devem estar junto aos módulos.<br>
Os módulos também provisionam arquivos e extensões.<br>
Há um layout de estrutura de pastas e arquivos que devem estar abaixo do modulo (modulepath).<br>
<br />
Para descobri o modulepath, é necessário executar o comando abaixo:<br>
```
# puppet config print
```
<br />
Neste exemplo o deverá constar uma pasta "webserver" em: <br>
modulepath = /etc/puppetlabs/code/environments/production/modules<br>
<br />
Dentro da pasta criamos uma outra chamada "manifests" então inserimos o arquivo .pp que consta a "class".<br>
