![ULAKBIM](../img/ulakbim.jpg)
#Güvenlik Duvarı Yönetim Sistemi Kurulumu
------------------------------------------

[TOC]



####Merkezi Yönetim Sistemi Entegrasyon Adımları
* Playbooklar üzerinde değişiklik yapıldıktan sonra sonra, yapılan değişikliklerin git reposunda güncellenmesi için aşağıdaki komutlar çalıştırılır.

```
# cd /etc/ansible/
# git pull
```
* Tüm playbookları Ansible makinesinden ilgili sistemlerde oynatarak güncel hallerinin çalışması sağlanır.

```
# ansible-playbook /etc/ansible/playbooks/ansible.yml
# ansible-playbook /etc/ansible/playbooks/gitlab.yml
# ansible-playbook /etc/ansible/playbooks/firewallbuilder.yml
# ansible-playbook /etc/ansible/playbooks/testbuilder.yml
# ansible-playbook /etc/ansible/playbooks/firewall.yml --skip-tags=deploy
# ansible-playbook /etc/ansible/playbooks/rsyslog.yml
```

* Kurulan tüm ana bileşen ve güvenlik duvarı durum kontrolü için, Ansible sunucusu üzerinde crontab’ a “**crontab -e**” komutu kullanılarak aşağıdaki komutlar eklenir. 
Böylelikle her beş dakikada bir yeni güvenlik duvarı kuralları sisteme otomatik gönderilerek, durum kontrölü sağlanır. 
Ayrıca girişi yapılmış ve onaylanmış her yeni kural en geç beş dakika içerisinde sistemlerde aktif hale gelir.
Her otuz dakikada bir ana bileşenlerin hepsi kontrol edilerek, kontrol dışı yapılan değişiklikler kaldırılır. 
Her bir crontab işinin başına eklenen “**MAILTO=**” parametresi ile crontab işi her çalıştığında mail göndermesi engellenir.

```
MAILTO=“”
*/30 * * * * /usr/bin/ansible-playbook /etc/ansible/playbooks/state.yml -vvvv
MAILTO=“”
59 00 * * * /usr/bin/ansible-playbook /etc/ansible/playbooks/maintenance.yml -vvvv
```

