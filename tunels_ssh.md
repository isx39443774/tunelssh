# Índex

- [Exercici 15 ldap remot i phpldapadmin local](#15-ldap-remot-i-phpldapadmin-local)

- [Exercici 16 ldap local i phpldapadmin remot](#16-ldap-local-i-phpldapadmin-remot)


# 15 ldap remot i phpldapadmin local


Configurem el nostre security-groups de amazon amb l'entorn gràfic perquè només estigui
obert el port 22

Accedim a la màquina AMI de Amazon

```
ssh -i ~sergi/.ssh/AWS_casa.pem fedora@52.56.66.147
```

Engegem el servidor ldap

```
docker run --rm --name ldap.edt.org -h ldap.edt.org -d edtasixm06/ldapserver
```

Introduïm el nostre server ldap al /etc/hosts de la màquina AMI

```
172.17.0.2 ldap.edt.org
```
En aquest punt encara no es poden realitzar consultes ldap, ja que no tenim el port 389
obert i tunel ssh no esta creat.

Engegem el nostre container phpldapadmin
```
docker run --rm --name phpldapadmin -h phpldapadmin --net mynet -d edtasixm06 phpldapadmin:19
```

Creem túnel ssh
```
ssh -f -L 172.18.0.1:50000:ldap.edt.org:389 -i ~sergi/.ssh/AWS_casa.pem fedora@52.56.66.147 sleep 10000
```

Verifiquem que podem realitzar consultes ldap, especifican el port
```
ldapsearch -x -LLL -p 50000 -b'dc=edt,dc=org' -h localhost
```

Editem el fitxer de configuració config.php, i el modifiquem perquè quedin les següents linies d'aquesta manera.
```
$servers->setValue('server','host','172.18.0.1');
$servers->setValue('server','port',50000);
```



# 16 ldap local i phpldapadmin remot
