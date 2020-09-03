![ULAKBIM](../img/ulakbim.jpg)
#Güvenlik Duvarı Yönetim Sistemi Kurulumu
------

[TOC]



####Ansible Playbookları ile GKTS Kurulumu
* “roles/gkts/vars/” klasörü altında değişkenleri barındıran “gkts.yml” dosyası üzerinde “hook” fonksiyonu altında bulunan “server” değişkenine 
Merkezi Yönetim Sisteminde bulunan ansible makinasının FQDN bilgisi, “port” değişkenine ansible makinesine ssh bağlantısı için kullanılcak ssh port bilgisi yazılır.
```
$ cd roles/gkts/vars/
$ sudo vi gkts.yml
# GKTS'in degiskenlerini iceren dosyadir
gkts:
# gkts playbooku ile kurulacak paketleri belirtmektedir.
    hook:
        conf:
            source: gktshook.sh.j2
            destination: /var/opt/ahtapot-gkts/gktshook.sh
            owner: ahtapotops
            group: ahtapotops
            mode: 755
        server: ansible01.gdys.local
        port: 22
```
* "roles/gkts/vars/" dizini altında bulunan "nginx.yml" dosyası içerisine “nginx” fonksiyonunun alt fonksinyonu olan “admin” altında bulunan “server_name” değişkenine 
admin arayüzü için ayarlanması istenen url adres bilgisi yazılır (Örn: admin.gkts.local). Yönetici arayüzüne erişim için internet tarayıcısında bu adres kullanılacaktır.
“nginx” fonksiyonunun alt fonksinyonu olan “developer” altında bulunan “server_name” değişkenine kullanıcı arayüzü için ayarlanması istenen domain adres bilgisi yazılır(Örn: kullanici.gkts.local).
Kullanıcı arayüzüne erişim için internet tarayıcısında bu adres kullanılacaktır.
```
$ cd roles/gkts/vars/
$ sudo vi nginx.yml
# Nginx'in degiskenlerini iceren dosyadir
nginx:
    conf:
        source: "gkts.conf.j2"
        destination: "/etc/nginx/conf.d/gkts.conf"
        owner: "root"
        group: "root"
        mode: "0644"
    admin:
        listen: "443"
        server_name: "admin_url_adresi"
        access_log: "/var/log/nginx/gkts-admin-access.log"
        error_log: "/var/log/nginx/gkts-admin-error.log"
    developer:
        listen: "443"
        server_name: "kullanici_url_adresi"
        access_log: "/var/log/nginx/gkts-developer-access.log"
        error_log: "/var/log/nginx/gkts-developer-error.log"
    service:
        name: "nginx"
        state: "started"
        enabled: "yes"
    default:
        path: "/etc/nginx/sites-available/default"
        state: "absent"
    certificate:
        source: "gkts.crt.j2"
        destination: "/etc/nginx/ssl/gkts.crt"
        owner: "root"
        group: "root"
        mode: "0644"
    key:
        source: "gkts.key.j2"
        destination: "/etc/nginx/ssl/gkts.key"
        owner: "root"
        group: "root"
        mode: "0644"
    ssldir:
        path: "/etc/nginx/ssl"
        owner: "root"
        group: "root"
        mode: "755"
        state: "directory"
```
