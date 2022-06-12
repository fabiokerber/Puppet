# Puppet

Resource Types:<br>
- Package
- User
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


