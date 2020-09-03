![ULAKBIM](../img/ulakbim.jpg)
#Güvenlik Duvarı Yönetim Sistemi Kurulumu
------------------------------------------

[TOC]



 |	İsterler		|   İşletim Sistemi   |
 |	----------		|  -----------------  |
 |Firewall Builder Sunucusu	|   Pardus-Ahtapot17  |
 |Firewall Sunucusu		|   Pardus-Ahtapot17  |
 |Test Firewall Sunucusu	|   Pardus-Ahtapot17  |


####Ansible Playbook ile FirewallBuilder Kurulumu

* FWB sunucusu VCS ve MYS sunucuları ile ssh bağlantısın kullanacağı için oluşturulan anahtarlar FWB sunucusuna aktarılır ve ilgili yerlere yerleştirilir. 

```
$ mkdir -p /home/ahtapotops/.ssh && chmod 700 ~/.ssh
$ touch /home/ahtapotops/.ssh/authorized_keys
$ cat ahtapotops.pub fw_kullanici.pub >> ~/.ssh/authorized_keys
$ cp ahtapotops ~/.ssh/id_rsa && cp ahtapotops.pub ~/.ssh/id_rsa.pub && cp ahtapotops-cert.pub ~/.ssh/id_rsa-cert.pub && chmod 600 ~/.ssh/*
```
```
# mkdir -p /root/.ssh &&
# cp git /root/.ssh/id_rsa && cp git.pub /root/.ssh/id_rsa.pub && cp git.pub /root/authorized_keys
```


*  **/etc/ansible/roles/firewallbuilder/vars/git.yml** dosyası üzerinde **yerel_gitlab_adresi** bölümüne kurulan Git sunucusunun adresi girilmelidir.

```
$ nano /etc/ansible/roles/firewallbuilder/vars/git.yml
# Git repolarini iceren dosyadir.
gitrepos:
    repo01:
        repo: "ssh://git@yerel_gitlab_adresi/ahtapotops/gdys.git"
        accept_hostkey: "yes"
        destination: "/etc/fw/gdys"
        key_file: "/home/ahtapotops/.ssh/id_rsa"
#    repoXX:
#        repo: "ssh://git@gitlab.ahtapot.org.tr/ahtapotops/gdys.git"
#        accept_hostkey: ""
#        destination: ""
#        key_file: ""
```
* **/etc/ansible/roles/firewallbuilder/vars/fwbuilder.yml** dosyasında **firewallbuilder** sunucusunun bilgileri girilir.

```
nano /etc/ansible/roles/firewallbuilder/vars/fwbuilder.yml
---
# Guvenlik Duvari Kurucusunun degiskenlerini iceren dosyadir.
firewallbuilder:
    fix:
        source: "reset_iptables"
        destination: "/usr/share/fwbuilder-5.1.0.3599/configlets/linux24/reset_iptables"
        group: "root"
        owner: "root"
        mode: "0644"
        force: "yes"
    bash:
        conf:
            source: "fwbuilder-ahtapot.sh.j2"
            destination: "/etc/profile.d/fwbuilder-ahtapot.sh"
            owner: "root"
            group: "root"
            mode: "0755"

fwb_editable_objects:
#örnek olarak
#fwb_editable_objects:
#   fwb.ahtapot.org.tr:
#       - objects:
#            - fwb.ahtapot.org.tr
#            - ikinci

```
* Sunucu üzerinde gerekli sıkılaştırma işlemleri ve FirewallBuilder kurulumu yapacak olan “**firewallbuilder.yml**” playbook’u Ansible makinesinden çalıştırılır.

```
$ ansible-playbook /etc/ansible/playbooks/firewallbuilder.yml
```
* FWB sunucusu VCS sunucusu ile root kullanıcısı ve git anahtarı iletişim sağlamaktadır. Bağlantının sağlandığını kontrol etmek için aşağıdaki adımları izleyiniz. Bağlantı sorunsuzsa ssh bağlantısını bitiriniz.

```
FWB makinesine ssh ile bağlanınız.
$ ssh fwbuilder.fqdn_bilgisi 
$ sudo su -
# cd /etc/fw/gdys/
# git status
# git pull
```
* GDYS için yapılan son değişiklikler için repo indirilir ve değişiklikler uygulanır.

```
$ git clone -b development https://github.com/Pardus-Ahtapot/GDYS.git
$ cp -r GDYS/ahtapot-gdys-gui/var/opt/gdysgui/*  /var/opt/gdysgui/
$ sudo chown -R ahtapotops:ahtapotops /var/opt/gdysgui/*
$ nano /var/opt/gdysgui/gitlab.py
```
* Gitlab sunucusunun FWB ile iletişiminde SSL kullanmak istemiyorsanız **def __init__(self, host, token="", oauth_token="", verify_ssl=True):** satırı **False** yapılır.

```
$ nano /var/opt/gdysgui/gitlab.py
```

####Ansible Playbook ile Test Firewall Kurulumu

* MYS sunucuları ile ssh bağlantısın kullanacağı için oluşturulan anahtarlar TFW sunucusuna aktarılır ve ilgili yerlere yerleştirilir. 

```
$ ssh-copy-id tfw.ahtapot.org.tr
```

* Sunucu üzerinde gerekli sıkılaştırma işlemleri ve Test Firewall kurulumu yapacak olan “**testfirewall.yml**” playbook’u çalıştırılır.

```
$ ansible-playbook /etc/ansible/playbooks/testfirewall.yml
```


####Ansible Playbook ile Firewall Kurulumu

* MYS makinasından FW makinasına paroalsız ssh bağlantısı sağlanmalıdır.
```
$ ssh-copy-id fw.ahtapot.org.tr
```

* “roles/firewall/vars” klasörü altında iptables değişkenlerini barındıran “iptables.yml” dosyası üzerinde “deploy” fonksiyonu altındaki “dest_port" bölümüne yerine 
Merkezi Yönetim Sistemi kapsamında kurulacak Git sunucusunun ssh portu girilmelidir.

```
$ cd roles/firewall/vars/
$ sudo vi iptables.yml
# Iptables yapilandirmasini iceren dosyadir.
iptables:
    service:
        v4conf: "/etc/iptables/rules.v4"
        v6conf: "/etc/iptables/rules.v6"
    deploy:
        repopath: "/etc/fw/gdys"
        filepath: "/etc/fw/gdys/files"
        rsync_opts: "--force"
        dest_port: "ssh_port"
        recursive: "yes"
```

* “**Ansible Playbookları**” dokümanında detaylı anlatımı bulunan, sunucu üzerinde gerekli sıkılaştırma işlemleri ve Firewall kurulumu yapacak olan “**firewall.yml**” playbook’u çalıştırılır.

```
$ ansible-playbook /etc/ansible/playbooks/firewall.yml --skip-tags=deploy
```

* Dokümanda yapılması istenilen değişiklikler terminal üzerinden yapıldığında  playbook oynatılmadan önce yapılan değişiklikler git’e push edilmelidir.

```
$ cd /etc/ansible
git status komutu ile yapılan değişiklikler gözlemlenir.
$ git status  
$ git add --all
$ git commit -m "yapılan değişiklik commiti yazılır"
$ git push origin master
```

####Güvenlik Duvarı Yönetim Sistemi Entegrasyon Adımları

* FWB makinasına bağlanılır ve arayüz yapılandırması yapılır. 

```
$ ssh ahtapotops@firewallbuilder -p ssh_port -i ahtapotops_kullanici_anahtari
```

* GitLab sunucusunu SSL sertifikası ile kullanmak istemiyorsanız, FWB makinasında ***/var/opt/gdysgui/** dizininde  **def __init__(self, host, token="", oauth_token="", verify_ssl=False):** satırı **False** yapınız ve sonraki başlığa arayüz yapılandırmaasına geçiniz.

```
$ nano /var/opt/gdysgui/gitlab.py
```

* FWB makinesinden GDYS Kontrol Paneli ile GitLab sunucusuna oluşturulan SSL sertifikaları ile erişmek için GitLab sunucusunun SSL sertifikası yüklenir. Oluşturmadı iseniz bu adımları atlayınız.
* SSL sertifika oluşturma için [SSL Anahtar Oluşturma](ca-kurulum.md) incelenmelidir.

```
$ sudo su -
```
* Gitlab üzerinde https bağlantısını kullanıldığından, Gitlab tarafında oluşturulmuş sertifika firewallbuilder makinesine tanıtılır. Bu işlem için Firewall Builder makinesinde “**/usr/share/ca-certificates**” klasörü altına oluşturulan sertifika dosyası kopyalanır.
* Sertifika yüklemek için kullanılacak ncurs menünün açılması için environment ayarlarından “**DEBIAN_FRONTEND**” seçeneği kaldırılır.

```
# unset DEBIAN_FRONTEND
```

* Sertifika yüklemek için aşağıdaki komut çalıştırılır.

```
# dpkg-reconfigure ca-certificates
```

* Açılan ncurs menüden “**Trust new certificates from certificate authorities ?**” seçeneğine “**yes**” cevabı verilir.

![Guvenlik-Duvari](../img/entegrasyon3.jpg)

* Bir sonraki ekranda sertifika dosyası seçilerek “**Ok**” butonuna basılır. Ve böylelikle sertifika yükleme işlemi tamamlanmış olur.

![Guvenlik-Duvari](../img/entegrasyon4.png)


####Güvenlik Duvarı Yönetim Sistemi Arayüz Yapılandırması

* FWB makinesine erişmek için fw_kullanici anahtarı kullanılacaktır. Oluşturduğunuz fw_kullanici anahtarını masaüstü ortamı olan bilgisayarıza aktarınız. 

```
$ ssh -X ahtapotops@FirewallBuilder_IP -i fw_kullanici
$ gdys-gui
```
* Erişim başarılı bir şekilde sağlandığında, Güvenlik Duvarı Yönetim Sistemi Kontrol Paneli uygulaması otomatik olarak açılacaktır.

![Guvenlik-Duvari](../img/entegrasyon5.jpg)

* “**Düzenle**” butonu altından “**Yapılandırma Ayarları**” seçilerek ilk kullanım öncesinde yapılması gereken yapılandırma ayarları yapılır. Yapılandırma ayarları ekranında bulunan “**Onay Mekanizması**” seçeneği ile Güvenlik Duvarı Yönetimin Sisteminde yapılacak değişikliklerin onay mekanizmasına dahil olup olmayacağı belirlenir. İlk açılışta “**Kapalı**” olarak gelen seçim onay mekanizmasını devreye almak için “**Açık**” konumuna getirilmelidir. 

![Guvenlik-Duvari](../img/entegrasyon6.jpg)

* Açıldığında kilitli olarak gelen “**Yapılandırma Ayarları**” üzerinde değişiklik yapmak için “**Kilidi Aç**” butonuna basılır ve açılan ekrana FirewallBuilder Makinesinın “**root**” kullanıcısına ait şifre girilir.
* Değiştirebilir duruma gelen “**Dizin Yapılandırma**” tabında, gerekli bilgi girişleri sağlanır ve “**Kaydet**” butonuna basılır.
    * “**Yapılandırma Dosya Adı**” satırında seçim otomatik olarak gelmektedir. Bu dizin, FirewallBuilder uygulamasına ait yapılandırmayı sakladığı XML dosyası olup, ilk çalıştırıldığında, Yerel GitLab’a konumlandırılmış olan ve içerisinde herhangi bir yapılandırma bulunmayan “**/etc/fw/gdys**” dizininde bulunan “**gdys.fwb**” dosyasından açılmaktadır. Onay mekanizmasının çalışabilmesi için belirtilen dizin altından bu dosyanın seçilmesi zaruridir.
    * “**Test Betik Dizini**” satırında, söz dizimi bakımından kontrol edilmek üzere test Makinesina gönderilmeden önce betiklerin konumlandırılacağı dizindir. Bu satıra “**/home/ahtapotops/testfw/**” yazılması zaruridir.
    * “**Hata Bildirim Dizini**” alanına, test betiklerinde hata alınması durumda ilgili hata ve logunun yazılması için oluşturulmuş ve tüm Ahtapot projesi kapsamında log yapısı için kullanılan “**/var/log/ahtapot/**” dizini girilir. Yapının bütünlüğünü korumak adına belirtilen dizinin girilmesi zaruridir.
    * “**Test Makinesi IP Adresi**” satırına AHTAPOT Test Güvenlik Duvarı Kurulum dokümanı takip edilerek kurulan test güvenlik duvarı makinesinın ip adresi yazılır.
    * “**Test Makinesi Kullanıcı Adı**” bilgisi olarak “**ahtapot**” kullanıcısı girilmesi zaruridir.
    * “**Test Makinesi Kopya Dizin**” alanı test makinesine gönderilen betiklerin konumlandırılacağı dizini belirtmekte olup, ahtapotops kullanıcısının ana dizini olan “**/home/ahtapotops/**” olması zaruridir.
    * “**Test Makinesi Port Numarası**” alanı test makinesine bağlantı sağlanırken kullanılacak ssh portunun belirtildiği alandır.
* “**Gitlab Yapılandırma**” tabına geçiş yapılarak, onay mekanizması için AHTAPOT GitLab Kurulum dokümanı takip edilerek yerele kurulmuş olan GitLab sunucusunun bilgileri girilirek “**Kaydet**” butonuna basılır.
    * “**GitLab Bağlantı Adresi**” satırına yerele kurulmuş olan GitLab sunucunun FQDN bilgisi “**https://Gitlabsunucu FQDN**” şeklinde yazılır.
    * “**GitLab Kullanıcı Adı**” satırına “**gdysapi**” kullanıcısının girilmesi zaruridir
    * “**GitLab Kullanıcı Parolası**” satırına “**gdysapi**” kullanıcısı için belirlenmiş olan parolanın girilmesi zaruridir.
    * “**GitLab Onay Dalı**” satırında “**onay**” dalının belirtilmesi zaruridir.
    * “**GitLab Ana Dal**” bilgisi  olarak “**master**” dalının belirtilmesi zaruridir.
    * “**GitLab Proje Adı**” bilgisi olarak “**gdys**” proje adının belirtilmesi zaruridir.

**NOT :** GitLab yapılandırma ekranında girilen bilgilerin  AHTAPOT GitLab Kurulum dokümanı Yapılandırma İşlemleri bölümünde onay mekanizması için oluşturulan proje bilgilerini içermesi zaruridir. Maddelerde belirtilmiş zaruri bilgiler GitLab Kurulum dokümanı takip edilerek yapılan kurulum sonrasında oluşan bilgilerdir. Kurulum aşamasında bu bilgilerden bir ve ya birden fazlasında değişiklik yapılır ise bu adımda değişikliği içeren bilgiler girilmelidir.


![Guvenlik-Duvari](../img/entegrasyon7.jpg)

* Yapılandırma işlemlerinin tamamlanmasının ardında “**Kilitle**” butonuna basılarak bu alanlarda değişiklik yapılması engellenir. “**x**” işaretine basılarak “**Yapılandırma Ayarları**” ekranından çıkılır.
* Yapılandırma işlemleri tamamlandıktan sonra, “**Çalıştır**” butonları aktif hale gelir. Firewall Builder uygulamasında ilk kullanım ayarlarını yapmak üzere “**Onaylanmış Ayalar ile Çalıştır**” seçeneği seçilirek Firewall Builder arayüzü açılır.

![Guvenlik-Duvari](../img/entegrasyon8.jpg)

* Açılan Firewall Builder uygulamasına onay mekanizması entegrasyonunda kullanılan betikleri dahil etmek için “**Edit**” seçeneğine basılarak “**Preferences...**” ekranı açılır. 

![Guvenlik-Duvari](../img/entegrasyon9.jpg)

*  Ayarlar ekranında “**Working directory:**” bölüme GitLab üzerinde bulunan “**gdys**” deposunda çalışmak için “**/etc/fw/gdys**” dizini girilmesi zaruridir. İlgili bilgi girildikten sonra “**Installer**” tabına geçilir.

![Guvenlik-Duvari](../img/entegrasyon10.jpg)

* “**Installer**” sekmesinde “**A full path to the Secure Shell utility**” bölüme Güvenlik Duvarı Yönetim Sistemi Kontrol Paneli uygulaması kapsamında geliştirilen installer.py betiğinin adresi olan “**/var/opt/gdysgui/installer.py**” satırının; “**A full path to the SCP utility**” bölümüne ise preinstaller.py betiğinin adresi olan “**/var/opt/gdysgui/preinstaller.py**” satırının girilmesi zaruridir. Seçimlerin yapılmasının ardından “**OK**” butonuna basılarak; ana ekrana dönüş yapılır.

![Guvenlik-Duvari](../img/entegrasyon11.jpg)

* Bu adımlar tamalandıktan sonra GDYS entegrasyon işlemi tamamlanmış, olup kullanıma hazır hale gelmiş olacaktır.



####Kurulum Sonrası Yapılacak Kontroller

* Kurulum işlemleri tamamlandığında, “**Base**” playbookunda var olan “**ssh_port**” değişkeni değiştirildi ise, sunuculara bağlantı yapılacak ssh port değiştiğinden GitLab arayüzünden “**ahtapotops/mys**” projesi açılarak “**Files**” dizini altındaki “**ansible.cfg**” dosyasında bulunan “**remote_port**” parametresi mevcut ssh port bilgisi ile değiştirilmelidir.

![MYS](../img/merkezi37.jpg)

* “**/etc/fw/gdys**” ile “**/etc/ansible**” altındaki ve “**/home/ahtapotops**” dizininde bulunan gizli klasör olan “**.git**” klasörünün altındaki tüm dosyaların sahibinin ahtapotops kullanıcısı olmasına dikkat edilmelidir. Bunun için aşağıdaki komutları kullanılabilir.
```
# sudo chown ahtapotops:ahtapotops -R /etc/fw/gdys/
# sudo chown ahtapotops:ahtapotops -R /etc/ansible/
# sudo chown ahtapotops:ahtapotops -R /home/ahtapotops/.ssh/
```
* SSH Port değiştiği durumlarda daha önce kayıt girilmiş, “**known_host**” dosyası güncelliğini yitireceğinden, yeniden ekleme yapmak için “**root**” ve “**ahtapotops**” kullanıcıları ile karşılıklı ssh bağlantısı sağlanması gerekmektedir. Bunun için aşağıdaki komutlar her sunucudan diğer bir sunucuya doğru çalıştırılmalı ve sunucu anahtarlarının kabul edilmesi sorulduğunda “**yes**” yazılmalıdır.
```
$ ssh FQDN_SUNUCU_ADI -p SSH_PORT
$ sudo su -
# ssh ahtapotops@FQDN_SUNUCU_ADI -p SSH_PORT -i /home/ahtapotops/.ssh/id_rsa
```

**NOT :** Sisteme ISO üzerinden kurulmuş ve ilk ansible ile kurulumları yapılacak makinelerde ansible playbookların çalıştırılabilmesi için, “**/etc/ansible**” altında bulunan “**ansible.cfg**” dosyasındaki “**remote_port**” parametresi öncelikle “**22**” yapılmalı ve playbook çalıştıktan sonra bu dosyadaki “**remote_port**” parametresine belirlenen “**ssh_port**” girilmelidir.


**Sayfanın PDF versiyonuna erişmek için [buraya](merkezi-yonetim-sistemi-ile-ahtapot-kurulumlari-yapilmasi.pdf) tıklayınız.**
