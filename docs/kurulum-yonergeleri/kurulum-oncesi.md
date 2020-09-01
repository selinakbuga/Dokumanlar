# Kurulumlara Başlamadan Önce Yapılması Gereken Hazırılıklar

### Pardus 17 Ahtapot ISO'sunun indirilmesi

**Pardus-Ahtapot 17 ISO** 'su indirmek için; [http://ahtapot.org.tr/indirmeler.html](http://ahtapot.org.tr/indirmeler.html)

Kurulan minimal işletim sisteminin ön tanımlı kullanıcısı **ahtapotops** ön tanımlı parolası **LA123** İlk giriş sırasında yeni parola belirleyiniz.

### Ahtapot BSGS Repo Adresleri

```
deb http://depo.ahtapot.org.tr/ahtapot stable main
deb http://depo.ahtapot.org.tr/ahtapot testing main
deb http://depo.ahtapot.org.tr/ahtapot siem main
deb http://depo.ahtapot.org.tr/ahtapot nac main
deb http://depo.ahtapot.org.tr/ahtapot eps main
```
### Pardus Repo Adresleri
```
deb http://depo.pardus.org.tr/pardus onyedi main contrib non-free
deb http://depo.pardus.org.tr/guvenlik onyedi main contrib non-free
```

Pardus Ahtapot ISOları, Pardus Sunucu ISO 'sunun sıkılaştırma ve Ahtapot Projesi için paketlerin sadeleştirilmesi ile oluşturulur. 

### Sertifika Otoritesi Sunucusu

Kuracağınız sistemde sertifika (CA) sunucusunun ayrı bir sunucu olarak yer alabileceği gibi Merkezi Yönetim Sistemi sunucusunda sertifikalarınızı oluşturabilirsiniz. 
