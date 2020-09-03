![ULAKBIM](../img/ulakbim.jpg)
#Rsyslog Kurulumu
-----------------

[TOC]


####Ansible Playbookları ile Rsyslog Kurulumu
**NOT:**Kurulacak sistem, SIEM yapısına dahil edilmek isteniyorsa, kurulum sonrasında Siber Olay, Açıklık, Risk İzleme ve Yönetim Sistemi Kurulumu sayfasında bulunan 
MYS Clientlarında Ossec Agent Dağıtımı başlığı incelenmelidir.
* "roles/base/vars” klasörü altında rsyslog değişkenlerinin barındıran “rsyslog.yml” dosyası içerisine "base_ossimcik_servers" fonksiyonu altında bulunan “server1” ve “server2” satırları altına ossimcik $
Ossimcik makinelerine log gonderilmesi istenilen clientların "client" içerisinde FQDN bilgileri girilir.
```
$ cd roles/base/vars/
$ sudo vi rsyslog.yml
# Log sunucu ayarlarini iceren dosyadir.
# Yorum satiri ile gosterilen sablon doldurularak istenilen kadar log sunucusu eklenebilir.
rsyslog:
    conf:
        source: "rsyslog.conf.j2"
        destination: "/etc/rsyslog.conf"
        owner: "root"
        group: "root"
        mode: "0644"
    service:
        name: "rsyslog"
        state: "started"
        enabled: "yes"
    ActionQueueMaxDiskSpace: "2g"
    ActionQueueSaveOnShutdown: "on"
    ActionQueueType: "LinkedList"
    ActionResumeRetryCount: "-1"
    WorkDirectory: "/var/spool/rsyslog"
    IncludeConfig: "/etc/rsyslog.d/*"

base_ossimcik_servers:
    server1:
        fqdn: "ossimcik.gdys.local"
        port: "514"
        severity: "*"
        facility: "*"
        clients:
            client01:
                fqdn: "ansible_fqdn"
            client02:
                fqdn: "gitlab_fqdn"
#    serverX:
#        fqdn: ""
#        port: ""
#        severity: "*"
#        facility: "*"
#        clients:
#            client01:
#                fqdn:
```
**NOT:** Log gönderici client makinelerine rsyslog icin gerekli anahtarlar konulmalıdır.

**NOT:** Anahtar oluşturulması için [CA Kurulumu ve Anahtar Yönetimi](ca-kurulum.md) dökümanındaki SSL Anahtar Oluşturma başlığı incelenmelidir. 
Oluşturulan anahtarlar client makineler içerisinde aşağıdaki dizinlere konulmalıdır. “client_fqdn” yerine client makinenin FQDN bilgisi girilmelidir.

```
/etc/ssl/certs/rootCA.pem
/etc/ssl/certs/client_fqdn.crt
/etc/ssl/private/client_fqdn.key
```
* “Ansible Playbookları” dokümanında detaylı anlatımı bulunan, sunucu üzerinde gerekli sıkılaştırma işlemleri ve rsyslog kurulumu yapacak olan “rsyslog.yml” playbook’u çalıştırılır.
```
$ ansible-playbook /etc/ansible/playbooks/rsyslog.yml
```
