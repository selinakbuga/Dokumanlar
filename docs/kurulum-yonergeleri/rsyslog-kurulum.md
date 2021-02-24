![ULAKBIM](../img/ulakbim.jpg)
# Log Sistemi Kurulumu
-----------------

[TOC]


 |   İsterler	|   İşletim Sistemi   |
 |  ----------	|  -----------------  |
 |     RSYSLOG	|   Pardus-Ahtapot17  |
 |     OSSİMCİK	|   Pardus-Ahtapot17  |


### RSYSLOG KURULUMU

MYS ve RSYSLOG makinalarının ssh bağlantısı sağlanır. 

Kurulacak sistem, SIEM yapısına dahil edilmek isteniyorsa, kurulum sonrasında Siber Olay, Açıklık, Risk İzleme ve Yönetim Sistemi Kurulumu sayfasında bulunan 
MYS Clientlarında Ossec Agent Dağıtımı başlığı incelenmelidir.

**/etc/ansible/roles/base/vars/rsyslog.yml** içerisinde mys sisteminde bulunan makinelerin log göndereceği ossimcik belirlenir. **rsyslog** altında bulunan **permittedpeer** satırına ossimcik makinesinin bilgisi girilir. **ossimciks** altında **server1** içerisine **ossimcik_fqdn** bilgisi **clients** altında **client01** içerisinde ossimcik'e log göndericek **client makinelerin fqdn** bilgisi girilir.
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

**/etc/ansible/roles/rsyslog/vars/rsyslog.yml** dosyası içerinde ossim, ossim korelasyon ve ossec makinalarının ip adresleri girilir. **permittedpeer:** bilgisine genelden logların imzalanacağı **rsyslog** makinasının bilgisi girilir.

```
nano /etc/ansible/roles/rsyslog/vars/rsyslog.yml
---
# Rsyslog'un degiskenlerini iceren dosyadir
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
      myprivkey: "/etc/ssl/private/{{ ansible_fqdn }}.key"
      authmode: "name"
      permittedpeer: "*.ahtapot.org.tr"
   port: "20514"
   port_udp: "514"
   imthreads: "5"
   MainQueueSize: "100000000"
   MainQueueWorkerThreads: "5"
   asyncWriting: "on"
   ioBufferSize: "1024k"
   WorkDirectory: "/var/spool/rsyslog"
   IncludeConfig: "/etc/rsyslog.d/*"

sources:
   src01:
      addr: "OSSIM_AV_Alerts"
      alertname: "OSSIM_AV_Alerts"
      alertfile: "{{ logrotate['Directory'] }}/ossim/ossimalerts.log"
      rawname: "OSSIM_All"
      rawfile: "{{ logrotate['Directory'] }}/ossim/ossim_raw.log"
   src02:
      addr: "OSSIMCIK_IP"
      alertname: "OSSIMCIK_AV_Alerts"
      alertfile: "{{ logrotate['Directory'] }}/ossimcik/ossimcikalerts.log"
      rawname: "OSSIMCIK_All"
      rawfile: "{{ logrotate['Directory'] }}/ossimcik/ossimcik_raw.log"
   src03:
      addr: "OSSIMCORR_IP"
      alertname: "OSSIMCORR_AV_Alerts"
      alertfile: "{{ logrotate['Directory'] }}/ossimcorr/ossimcorr_alerts.log"
      rawname: "OSSIMCORR_All"
      rawfile: "{{ logrotate['Directory'] }}/ossimcorr/ossimcorr_raw.log"

```

Rsyslog makinası ve log yollayacak makinaların rsyslog düzenlemesini yapacak olan **rsyslog.yml** playbook’u çalıştırılır.
```
$ ansible-playbook /etc/ansible/playbooks/rsyslog.yml
```

### OSSIMCIK KURULUMU

MYS ve OSSİMCİK makinalarının ssh bağlantısı sağlanır. 

**/etc/ansible/roles/ossimcik/vars/rsyslog.yml** dosyası içerisinde ossimcik makinesinin logları göndermesi istenilen rsyslog ve ossim makinelerinin fqdn bilgileri girilir. **permittedpeer:** bilgisine genelden log toplayabilmesi için ** *.ahtapot.org.tr** veya log yollayacak makinaların **FQDN** bilgileri yazılır. (mys.ahtapot.org.tr,vcs.ahtapot.org.tr) yazılır. 
```
$ nano /etc/ansible/roles/ossimcik/vars/rsyslog.yml
# Rsyslog degiskenlerinin tutuldugu dosyadir.
ossimcik_rsyslog:
   service:
      name: "rsyslog"
      state: "started"
      enabled: "yes"
   conf:
      source: "rsyslog.conf.j2"
      destination: "/etc/rsyslog.conf"
      owner: "root"
      group: "root"
      mode: "0644"
   tls:
      state: "on"
      cacert: "/etc/ssl/certs/rootCA.pem"
      mycert: "/etc/ssl/certs/{{ ansible_fqdn }}.crt"
      myprivkey: "/etc/ssl/private/{{ ansible_fqdn }}.key"
      authmode: "name"
      permittedpeer: "CERTIFICATE COMMON NAME"
   rsyslog_server: "RSYSLOG_FQDN"
   ossim_server: "OSSIM_FQDN"
   port: "20514"
   port_udp: "514"
   imthreads: "4"
   MainQueueSize: "100000000"
   MainQueueWorkerThreads: "4"
   RsyslogQueueWorkerThreads: "4"
   asyncWriting: "on"
   ioBufferSize: "1024k"
   ActionResumeRetryCount: "-1"
   ActionQueueFileName01: "fwd_ClientToRsyslog"
   ActionQueueFileName02: "fwd_AlertsToRsyslog"
   ActionQueueFileName03: "fwd_AlertsToOSSIM"
   ActionQueueFileName04: "fwd_WinToRsyslog"
   ActionQueueSaveOnShutdown: "on"
   ActionQueueType: "LinkedList"
   ActionQueueMaxFileSize: "100m"
   ActionQueueSize: "1000000"
   Mode: "inotify"
   WorkDirectory: "/var/spool/rsyslog"
   IncludeConfig: "/etc/rsyslog.d/*"
   LogLocationUSB: "/var/log/usb.log"
   LogLocationALL: "/var/log/client.log"

#    permittedpeer: "mys.ahtapot.org.tr"
#    permittedpeer: '"mys.ahtapot.org.tr", "vcs.ahtapot.org.tr", "ipds.ahtapot.org.tr"'
#    rsyslog_server: "rsyslog.ahtapot.org"
#    ossim_server: "ossim01.ahtapot.org"
```

**ossimcik.yml** playbooku çalıştırılır.
```
$ ansible-playbook /etc/ansible/playbooks/ossimcik.yml
```
Ossimcik playbookunun oynatılmasının ardından Rsyslog ve Nxlog **SSL** ile haberleşmeleri için sertifikalar yerleştirilmelidir.

Log gönderici client makinelerine rsyslog icin gerekli anahtarlar konulmalıdır. Anahtar oluşturulması için [CA Kurulumu ve Anahtar Yönetimi](ca-kurulum.md) dökümanındaki SSL Anahtar Oluşturma başlığı incelenmelidir. Oluşturulan anahtarlar client makineler içerisinde aşağıdaki dizinlere konulmalıdır. “client_fqdn” yerine client makinenin FQDN bilgisi girilmelidir.

Rsyslog için rootCA sertifikası **/etc/ssl/private** dizini altına kopyalanır.
```
/etc/ssl/certs/rootCA.pem
```
Ossimcik sertifikaları ossimcik **FQDN** ismi ile dosya oluşturularak aşağıdaki dizinlere kopyalanır.
```
/etc/ssl/certs/client_fqdn.crt
/etc/ssl/private/client_fqdn.key
```
Sertifikaların yerleştirilmesiyle Rsyslog servisi yeniden başlatılır.
```
systemctl restart rsyslog.service
```
Nxlog sertifikaları için rsyslog ile aynı sertifikalar kullanılır. Nxlog sertifikaları okuyabilmesi için yeni bir dizin oluşturulur.
```
mkdir /etc/ssl/nxlog
```
Oluşturulan dizin içerisine aşağıdaki gibi **rootCA** isimli ve makinenin **FQDN** isimili dosyalar oluşturularak keyler kopyalanır. 
```
/etc/ssl/nxlog/rootCA.pem
/etc/ssl/nxlog/client_fqdn.crt
/etc/ssl/nxlog/client_fqdn.key
```
Sertifikalar yerlestirilmesiyle Nxlog servisi yeniden başlatılır.
```
systemctl restart nxlog.service
```
