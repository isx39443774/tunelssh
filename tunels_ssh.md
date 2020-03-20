# TÚNELS SSH

## Índex

- [Exercici 15 ldap remot i phpldapadmin local](#15-ldap-remot-i-phpldapadmin-local)

- [Exercici 16 ldap local i phpldapadmin remot](#16-ldap-local-i-phpldapadmin-remot)


### 15 ldap remot i phpldapadmin local


Configurem el nostre security-groups d'Amazon amb l'entorn gràfic perquè només estigui
obert el port 22.

Accedim a la màquina AMI d'Amazon.
```
ssh -i ~sergi/.ssh/AWS_casa.pem fedora@52.56.66.147
```

Engeguem el container ldap remot.
```
docker run --rm --name ldap.edt.org -h ldap.edt.org -d edtasixm06/ldapserver
```

Introduïm el nostre server ldap al /etc/hosts de la màquina AMI.
```
172.17.0.2 ldap.edt.org
```

En aquest punt encara no es poden realitzar consultes ldap, ja que no tenim el port 389
obert i túnel ssh no està creat.

Engeguem el nostre container phpldapadmin local en mode interactiu per poder editar-ne el fitxer de configuració.
```
docker run --rm --name phpldapadmin -h phpldapadmin --net mynet -it edtasixm06 phpldapadmin:19 /bin/bash
```

Editem el fitxer de configuració config.php, i el modifiquem perquè quedin les següents línies d'aquesta manera.
```
$servers->setValue('server','host','172.18.0.1');
$servers->setValue('server','port',50000);
```

Creem túnel directe ssh.
```
ssh -f -L 172.18.0.1:50000:ldap.edt.org:389 -i ~sergi/.ssh/AWS_casa.pem fedora@52.56.66.147 sleep 10000
```

Per acabar, engeguem el servei php.
```
bash startup.sh
```

Per realitzar-ne la pertinent comprovació, podem anar a qualsevol navegador i introduïm el següent per poder accedir al phpldapadmin.
```
localhost:8080/phpldapadmin
```



### 16 ldap local i phpldapadmin remot

Engeguem el container ldap local.
```
docker run --rm --name ldap.edt.org -h ldap.edt.org --net mynet -d edtasixm06/ldapserver19
```

Comprovem que es poden realitzar consultes localment al servidor ldap.
```
ldapsearch -x -LLL -b'dc=edt,dc=org' -h 172.18.0.2
```

Abans d'engegar el servidor phpldapadmin a Amazon, hem de comprovar que l'AMI permet fer bind a diferents interfícies. Per defecte només permet fer-ho localment. Per realitzar aquesta comprovació editarem el fitxer /etc/ssh/sshd_config i modifiquem les següents línies perquè quedin de la següent manera.
```
GatewayPorts yes
```

Un cop editat fem un restart al servei sshd perquè s'apliquin els canvis realitzats.
```
systemctl restart sshd
```

Engeguem el nostre container phpldapadmin local en mode interactiu per poder editar-ne el fitxer de configuració.
```
docker run --rm --name phpldapadmin -h phpldapadmin --net mynet -it edtasixm06 phpldapadmin:19 /bin/bash
```

Editem el fitxer de configuració config.php, i el modifiquem perquè quedin les següents línies d'aquesta manera.
```
$servers->setValue('server','host','172.18.0.1');
$servers->setValue('server','port',50000);
```

Creem el túnel reverse ssh des de el ldap local al phpldapadmin remot.
```
ssh -R 172.18.0.1:50000:172.18.0.2:389 -i ~sergi/.ssh/AWS_casa.pem fedora@52.56.66.147
```

Edtiem el fitxer /etc/hosts del host d'Amazon i afegim la següent línia.
```
172.18.0.1 phpldapadmin
```

Per acabar aquesta meravellosa pràctica, creem un túnel directe ssh des de el host local al host-remot phpldapadmin passant pel host-destí (AMI).
```
ssh -L 8080:phpldapadmin:80 -i ~sergi/.ssh/AWS_casa.pem fedora@52.56.66.147
```

Per realitzar-ne la pertinent comprovació, podem anar a qualsevol navegador i introduïm el següent per poder accedir al phpldapadmin.
```
localhost:8080/phpldapadmin
```
