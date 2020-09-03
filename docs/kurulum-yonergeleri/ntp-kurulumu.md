![ULAKBIM](../img/ulakbim.jpg)
#Güvenlik Duvarı Yönetim Sistemi Kurulumu
------

[TOC]


####Ansible Playbookları ile NTP Kurulumu
* “roles/base/vars” klasörü altında ntp değişkenlerinin barındıran “ntp.yml” dosyası içerisine "base_ntp_servers" fonksiyonu altında bulunan "server1" ve "server2" satırları altına 
NTP sunucularının FQDN bilgileri girilmelidir. Sistemde bir NTP sunucusu olduğu durumda "server2" satırları silinebilir yada istenildiği kadar NTP sunucusu eklenebilir.

```
$ cd roles/base/vars/
$ sudo vi ntp.yml
# Zaman sunucusu ayarlarini iceren dosyadir.
# Yorum satiri ile gosterilen sablon doldurularak istenilen kadar zaman sunucusu eklenebilir.
ntp:
    conf:
        source: "ntp.conf.j2"
        destination: "/etc/ntp.conf"
        owner: "root"
        group: "root"
        mode: "0644"
   service:
        name: "ntp"
        state: "started"
        enabled: "yes"

base_ntp_servers:
    server1:
        fqdn: "0.tr.pool.ntp.org"
    server2:
        fqdn: "1.tr.pool.ntp.org"
#    serverX:
#        fqdn: ""
```

```
$ ansible-playbook /etc/ansible/playbooks/ntp.yml
```
