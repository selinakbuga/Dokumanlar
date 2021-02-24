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


### OSSIMCIK Kurulumu

* **NOT:** Dökümanda yapılması istenilen değişiklikler gitlab arayüzü yerine terminal üzerinden yapılması durumunda playbook oynatılmadan önce yapılan değişiklikler git'e push edilmelidir.

```
$ cd /etc/ansible
git status komutu ile yapılan değişiklikler gözlemlenir.
$ git status  
$ git add --all
$ git commit -m "yapılan değişiklik commiti yazılır"
$ git push origin master
```

* Ahtapot Temel ISO ile kurulumu sağlanmış olan sunucunun Merkezi Yönetim Sistemi ile bağlanıtısı sağlandıktan sonra OSSIMCIK rolünü yüklemesi için Ansible Playbook oynatılır.
* GitLab arayüzünde MYS reposunda bulunan **hosts** dosyasına "**ossimcik**" rolü altına ilgili makinanın fqdn bilgileri yazılır.
```
[ossimcik]
ossimcik01.fqdn_bilgisi
```

* GitLab arayüzünde MYS reposunda "**roles/base/vars/hosts.yml**" dosyasına ossimcik makinasının bilgileri eklenir.
```
    server22:
        ip: "x.x.x.x" 
        fqdn: "ossimcik01.gdys.local"
        hostname: "ossimcik01"
```
* **/etc/ansible/roles/ossimcik/vars/rsyslog.yml** dosyası içerisinde ossimcik makinesinin logları göndermesi istenilen rsyslog ve ossim makinelerinin fqdn bilgileri girilir. **permittedpeer** satırına ossimcik makinasının fqdn bilgisi girilir.
```
vi/etc/ansible/roles/ossimcik/vars/rsyslog.yml
    permittedpeer: ["ossimcik.ahtapot.org.tr"]
    rsyslog_server: "rsyslog.ahtapot.org"
    ossim_server: "ossim01.ahtapot.org"
```
* **/etc/ansible/roles/base/vars/rsyslog.yml** içerisinde mys sisteminde bulunan makinelerin loglarını göndereceği ossimcikler belirlenir ve **ossimciks** altında **server1** içerisine **ossimcik_fqdn** bilgisi **clients** altında **client01** içerisinde ossimcik'e log göndericek **client makinelerin fqdn** bilgisi girilir.
```
vi /etc/ansible/roles/base/vars/rsyslog.yml 
---
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
      permittedpeer: "ossimcik.ahtapot.org.tr"
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
      fqdn: "ossimcik.ahtapot.org.tr"
      port: "20514"
      clients:
        - "mys.ahtapot.org.tr"
        - "git.ahtapot.org.tr"
#       - "*.ahtapot.org.tr"
```
* "**/etc/ansible/roles/rsyslog/vars/rsyslog.yml**" dosyası içerinde ossim, ossim korelasyon ve ossec makinalarının ip adresleri girilir. **permittedpeer:** bilgisine genelden log toplayabilmesi için ** *.ahtapot.org.tr** yazılır.
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
* ISO kurulumu tamamlanmmış ve OSSIMCIK rolü yüklenecek makina üzerinde ansible playbooku çalıştırmak için, Ansible makinasına **ahtapotops** kullanıcısı ile SSH bağlantısı yapılarak, "**ossimcik.yml**" playbooku çalıştırılır.
```
$ cd /etc/ansible
$ ansible-playbook playbooks/ossimcik.yml
```
* Ossimcik playbookunun oynatılmasının ardından Rsyslog ve Nxlog **SSL** ile haberleşmeleri için sertifikalar yerleştirilmelidir.
**NOT:** Anahtar oluşturulması için CA Kurulumu ve Anahtar Yönetimi dökümanındaki [Log Yönetimi Anahtar Oluşturma](ca-kurulum.md) başlığı incelenmelidir.

* Rsyslog için rootCA sertifikası **/etc/ssl/private** dizini altına kopyalanır.
```
vi /etc/ssl/certs/rootCA.pem
```
* Ossimcik sertifikaları ossimcik **FQDN** ismi ile dosya oluşturularak aşağıdaki dizinlere kopyalanır.
```
vi /etc/ssl/certs/ossimcik01.gdys.local.crt
vi /etc/ssl/private/ossimcik01.gdys.local.key
```
* Sertifikaların yerleştirilmesiyle Rsyslog servisi yeniden başlatılır.
```
systemctl restart rsyslog.service
```
* Nxlog sertifikaları için rsyslog ile aynı sertifikalar kullanılır. Nxlog sertifikaları okuyabilmesi için yeni bir dizin oluşturulur.
```
mkdir /etc/ssl/nxlog
```
* Oluşturulan dizin içerisine aşağıdaki gibi **rootCA** isimli ve makinenin **FQDN** isimili dosyalar oluşturularak keyler kopyalanır. 
```
vi /etc/ssl/nxlog/rootCA.pem
vi /etc/ssl/nxlog/ossimcik01.gdys.local.crt
vi /etc/ssl/nxlog/ossimcik01.gdys.local.key
```
* Sertifikalar yerlestirilmesiyle Nxlog servisi yeniden başlatılır.
```
systemctl restart nxlog.service
```
