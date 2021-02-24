![ULAKBIM](../img/ulakbim.jpg)
# Log Sistemi ve Rsyslog Kurulumu
-----------------

[TOC]


####Ansible Playbookları ile Rsyslog Kurulumu
**NOT:**Kurulacak sistem, SIEM yapısına dahil edilmek isteniyorsa, kurulum sonrasında Siber Olay, Açıklık, Risk İzleme ve Yönetim Sistemi Kurulumu sayfasında bulunan 
MYS Clientlarında Ossec Agent Dağıtımı başlığı incelenmelidir.
* "roles/base/vars” klasörü altında rsyslog değişkenlerinin barındıran “rsyslog.yml” dosyası içerisine "base_ossimcik_servers" fonksiyonu altında bulunan “server1” altına ossimcik $
Ossimcik makinelerine log gonderilmesi istenilen clientların "client" içerisinde FQDN bilgileri girilir. **permittedpeer** satırına ossimcik makinesinin bilgisi girilir.
```
$ nano /etc/ansible/roles/base/vars/rsyslog.yml

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
   tls:
      state: "on"
      cacert: "/etc/ssl/certs/rootCA.pem"
      mycert: "/etc/ssl/certs/{{ ansible_fqdn }}.crt"
      mykey: "/etc/ssl/private/{{ ansible_fqdn }}.key"   
      authmode: "name"
      permittedpeer: ""
   MainQueueSize: "100000"
   MainQueueWorkerThreads: "2"
   ActionResumeRetryCount: "-1"
   QueueType: "LinkedList"
   QueueFileNameANS: "srvfrwd_ans"
   QueueFileNameIPT: "srvfrwd_iptables"
   QueueFileNameSYS: "srvfrwd_syslog"
   QueueFileNameSURICATA: "srvfrwd_suricata"
   QueueSaveOnShutdown: "on"
   QueueMaxFileSize: "100m"
   QueueSize: "250000"
   asyncWriting: "on" 
   ioBufferSize: "256k" 
   Mode: "inotify"
   WorkDirectory: "/var/spool/rsyslog" 
   IncludeConfig: "/etc/rsyslog.d/*" 

ossimciks:
   server01:
      fqdn: "OSSIMCIK_FQDN"
      port: "20514"
      clients:
        - "LOG_KAYNAGI_FQDN"
        - "LOG_KAYNAGI_FQDN"
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
