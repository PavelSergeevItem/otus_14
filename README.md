**PAM**

**1. Описание задания** 

Запретить всем пользователям, кроме группы admin, логин в выходные (суббота и воскресенье), без учета праздников.

 **2. Выполенение задания.**

 1. Создал виртуальную машину при помощи вагрант и вагрантфайл из методички.
 2. Подключился к машине, перешел в режим рут.
 3. Созадл пользователей. 
`sudo useradd otusadm && sudo useradd otus`
4. Задал этим пользователям пароли.
`echo "Otus2022!" | sudo passwd --stdin otusadm && echo "Otus2022!" | sudo passwd --stdin otus`
5.  Создал группу admin `sudo groupadd -f admin`.
6.  Добавил пользователя vagrant, root и otusadm в группу admin `usermod otusadm -a -G admin && usermod root -a -G admin && usermod vagrant -a -G admin`.
7.  Проверил, что два недавно созданных пользователей могу подключится к вирутальной машние через ssh/
```
ssh otus@192.168.57.10
The authenticity of host '192.168.57.10 (192.168.57.10)' can't be established.
ED25519 key fingerprint is SHA256:3uALhhntr6lJjDJsqKlmRlqudwzBKX+tL/PbGjXA9OQ.
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.57.10' (ED25519) to the list of known hosts.
otus@192.168.57.10's password: 
[otus@pam ~]$ 
[otus@pam ~]$ ^C
[otus@pam ~]$ logout
Connection to 192.168.57.10 closed.
```
```
spa@stnd:~/u$ ssh otusadm@192.168.57.10
otusadm@192.168.57.10's password: 
[otusadm@pam ~]$ logout
```
8. Проверил, что пользователи root, vagrant и otusadm есть в группе admin:
```
[root@pam ~]# cat /etc/group | grep admin
printadmin:x:994:
admin:x:1003:otusadm,root,vagrant
[root@pam ~]# 
```
9. Создал скрипт `nano /usr/local/bin/login.sh`, добавил права на исполнения `chmod +x /usr/local/bin/login.sh`
10.  Указал в файле /etc/pam.d/sshd модуль pam_exec и скрипт:
```
#%PAM-1.0
auth	   substack     password-auth
auth	   include	postlogin
account    required     pam_sepermit.so
account    required     pam_nologin.so
account    include	password-auth
password   include	password-auth
#pam_selinux.so close should be the first session rule
session    required     pam_selinux.so close
session    required     pam_loginuid.so
# pam_selinux.so open should only be followed by sessions to be executed in the user context
session    required     pam_selinux.so open env_params
session    required     pam_namespace.so
session    optional     pam_keyinit.so force revoke
session    optional     pam_motd.so
session    include	password-auth
session    include	postlogin
auth	   required     pam_exec.so /usr/local/bin/login.sh
```
11. Перевел дату на виртуальной машины, чтобы текущая дата была суббота: `sudo date 082712302022.00`.
12. Проверил работу. Попробовал подключится через ssh пользователями otus и otusadm. Подключился только otusadm, настройка выполненна правильно.
```
spa@stnd:~/u$ ssh otusadm@192.168.57.10
otusadm@192.168.57.10's password: 
Last failed login: Thu Jun 22 13:21:46 UTC 2023 from 192.168.57.1 on ssh:notty
There were 4 failed login attempts since the last successful login.
Last login: Sat Aug 27 13:10:56 2022 from 192.168.57.1
[otusadm@pam ~]$ 
```
