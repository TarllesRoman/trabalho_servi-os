---
description: >-
  Relatório apresenta a configuração do servidor VsFTPd para realizar
  autenticações em uma base LDAP remota.
---

# VsFTPD + LDAP

O relatório a seguir apresenta a configuração de um servidor FTP para autenticação em uma base LDAP já existente.Para o servidor FTP utilizaremos o VsFTPD e a base ldap foi configurada com a utilização do OpenLDAP. Nesse relatório não será abordado a criação da base LDAP que será utilizada mas suas peculiaridades serão informadas conforme o autor achar necessário. Primeiro vamos configurar um cliente para o banco LDAP, para isso instale os pacotes:

```dart
# yum install epel-release
# yum update
# yum install openldap-clients ldapvi nss-pam-ldapd
```

Caso o seu servidor DNS não responde pela base LDAP que está utilizando, insira seu endereço no arquivo /etc/hosts e assim utilizaremos apenas nome ldap1 nos comandos subsequentes.

```dart
# echo “0.0.0.0 ldap1 myldap.server” >> /etc/hosts
```

Após isso já é possível testar as consultas para a base que contém os usuários de nosso serviço FTP.

```dart
# ldapsearch -LLL -D cn=usuario,dc=base,dc=com -H ldap://ldap1 -b ou=Usuarios,dc=base,dc=com -W
```

Ao usar o comando acima será perguntada a senha do usuário que realizou a busca e se for um usuário apto a realizar tal busca a mesma retornará todos os dados da “tabela” Usuários. Se o passo estiver ok, vamos passar para a configuração de nosso servidor.

Antes de instalar o servidor VsFTPD, primeiro desative o firewall e o selinux para evitar erros estranhos e desnecessários

```dart
# systemctl stop firewalld.service
# systemctl disable firewalld.service
```

{% code title="@ /etc/selinux/config" %}
```dart
Neste mude o valor da chave SELINUX para disabled. É necessária a reiniciar o sistema para essa configuração ter efeito.
```
{% endcode %}

Agora instale o pacote vsftpd através do yum

```dart
# yum install vsftpd
```

Crie um usuário de testes e baixe o cliente FTP FileZilla para realização de testes.

```dart
# useradd teste
# passwd teste
```

Iniciando serviço:

```dart
# systemctl start vsftpd
# systemctl enable vsftpd
```

O processo de autenticação do VsFTPD utiliza o pam, portanto será necessário uma pequena configuração no arquivo /etc/pam.d/vsftpd

{% code title="\# vim /etc/pam.d/vsftpd" %}
```dart
#%PAM-1.0
##session    optional   pam_keyinit.so    force revoke
##auth       required   pam_listfile.so item=user sense=deny file=/etc/vsftpd/ftpusers onerr=succeed
##auth       required   pam_shells.so
##auth       include    password-auth
##account    include    password-auth
##session    required   pam_loginuid.so
##session    include    password-auth
auth         required   pam_ldap.so
account      required   pam_ldap.so
```
{% endcode %}

Agora entre no aruivo /etc/nsswitch e acrescente a opção 'ldap' nas configurações abaixo.

{% code title="\# vim /etc/nsswitch" %}
```dart
passwd:     files sss ldap
shadow:     files sss ldap
group:      files sss ldap
...
services:   files sss ldap
netgroup:   nisplus sss ldap
...
automount:  files nisplus sss ldap
```
{% endcode %}

Agora realize algumas configurações no arquivo /etc/nslcd. Deixe o restante do aquivo comentado

{% code title="\# vim /etc/nslcd" %}
```dart
uri ldap://0.0.0.0/         ##your ldap address
ldap_version 3
base dc=base,dc=com

ssl no

binddn cn=usuario,dc=base,dc=com
bindpw senha

base shadow ou=Usuarios,dc=base,dc=com
```
{% endcode %}

```dart
# systemctl stop nslcd
# systemctl start nslcd
```

Por ultimo realize as configurações de bind da sua base ldap em: /etc/openldap/ldap.conf

{% code title="\# vim /etc/openldap/ldap.conf" %}
```dart
host 192.168.11.161

BASE    ou=Usuarios,dc=base,dc=com
URI     ldap://ldap1

binddn cn=user,dc=base,dc=com
bindpw password

TLS_CACERTDIR   /etc/openldap/certs

SASL_NOCANON    on
```
{% endcode %}

Agora basta configurar o servidor VSFTPD corretamente para que a autenticação comece a funcionar.

{% code title="\# vim /etc/vsftpd/vsftpd.conf" %}
```dart
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
anon_upload_enable=YES
anon_mkdir_write_enable=YES
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES

xferlog_file=/var/log/xferlog
xferlog_std_format=YES

chroot_local_user=YES
allow_writeable_chroot=YES

listen=NO
listen_ipv6=YES

pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
guest_enable=YES
guest_username=ftp
local_root=/home/ftp
```
{% endcode %}

{% hint style="info" %}
O diretório cujo o caminho está especificado em _`local_root`_ dever ser criado manualmente.
{% endhint %}

Se desejar que seu sevidor trabalhe com criptografia, acrescente também as configurações abaixo:

{% code title="\# vim /etc/vsftpd/vsftpd.conf" %}
```dart
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
rsa_cert_file=/etc/vsftpd/ssl/cert-vsftpd.tarlles.pem
rsa_private_key_file=/etc/vsftpd/ssl/priv-vsftpd.tarlles.pem
```
{% endcode %}

```dart
# systemctl stop vsftpd
# systemctl start vsftpd
```

### Referencias

* [https://translate.google.com/translate?hl=pt-BR&sl=zh-CN&tl=pt&u=https%3A%2F%2Fjuejin.im%2Fpost%2F5d6cd8845188255a9f1925e](https://translate.google.com/translate?hl=pt-BR&sl=zh-CN&tl=pt&u=https%3A%2F%2Fjuejin.im%2Fpost%2F5d6cd8845188255a9f1925ef) - acesso em 24/11/2019
* [https://translate.google.com/translate?hl=pt-BR&sl=zh-CN&u=https://www.bbsmax.com/A/A7zgYqY54n/&prev=search](https://translate.google.com/translate?hl=pt-BR&sl=zh-CN&u=https://www.bbsmax.com/A/A7zgYqY54n/&prev=search) - acesso em 24/11/2019

