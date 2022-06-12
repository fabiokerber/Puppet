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


