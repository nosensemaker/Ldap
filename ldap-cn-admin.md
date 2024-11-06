# Como colocar o CN = ADMIN no Ldap

1.0 Crie um arquivo .ldif com as configurações do admin

dn: cn=admin,dc=meusite,dc=com
objectClass: simpleSecurityObject
objectClass: organizationalRole
cn: admin
description: LDAP administrator
userPassword: secret

2.0 Roda o comando para inserior no banco de dados.
    $ sudo ldapadd -x -D cn=admin,dc=meusite,dc=com -W -f base.ldif