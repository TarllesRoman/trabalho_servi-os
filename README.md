---
description: >-
  Trabalho realizado pelo aluno Tarlles Roman Sfredo, para a disciplina de
  Gerência e Configuração de Serviços de Internet, do curso de Tecnologia em
  Sistemas para Internet no IFSEMG-Barbacena 2019
---

# Introdução

## Objetivo

Configurar os servidores ProFTPD e VsFTPD para realizar autenticação de usuários em uma base LDAP previamente configurada pelo professor. Utilizando para tal os pacotes OpenLDAP, ProFTPD e VSFTPD rodando sobre um servidor CentOS 7

## Sobre o LDAP

Um exemplo de usuário da base LDAP que será usada nas proximas configurações é mostrado abaixo.

```text
dn: uid=diego,ou=Usuarios,dc=domith,dc=labredes,dc=info
uid: diego
objectClass: posixAccount
objectClass: shadowAccount
objectClass: inetOrgPerson
cn: diego
sn: ferreira
uidNumber: 10003
gidNumber: 10003
homeDirectory: /home/diego
loginShell: /bin/bash
gecos: Diego Ferreira
userPassword: {crypt}$6$fP80J3obdEYxWITS$c5e/UkMKLbejx3Iabe3MG0J5x74/Xr6MWuWhraK1lxbjwoldk9PkExBvWLyHKfq0EwCNdV0pGqf4t4FueDtpY1
shadowLastChange: 18177
shadowMax: 99999
shadowWarning: 7
```

Veja esse relatório em [https://tarllesrs.gitbook.io/trabalho-servicos/](https://tarllesrs.gitbook.io/trabalho-servicos/)

