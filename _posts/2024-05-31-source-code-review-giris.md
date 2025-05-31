## **Secure Code Review Nedir?**

Secure Code Review, yazılım geliştirme sürecinde kaynak kodun güvenlik açısından incelenmesi işlemidir. SDLC'nin kritik bir aşaması olan bu süreç, özellikle **XSS**, **SQLi**, **SSRF** gibi yaygın güvenlik açıklarının tespit edilmesini hedefler. Bu tür açıkların erken tespiti, veri sızıntısı gibi büyük güvenlik ihlallerinin önlenmesine katkı sağlar ve kurumun hem maddi hem de itibari kayıplar yaşamasının önüne geçer.

## **Secure Code Review’un Amacı ve Katkısı**

- **Hassas bilgilerin korunması**,
    
- **Veri sızıntılarının önlenmesi**,
    
- **Hard-coded secrets ve API key’lerin ifşasının engellenmesi**,
    
- **İtibar ve yasal yükümlülüklerin korunması**,
    
- **Güvenlik açıklarının erken aşamada ve düşük maliyetle düzeltilmesi**.
    

## **Yaklaşım: Kritik Kontrol Başlıkları**

Kod inceleme sürecinde odaklanılması gereken temel başlıklar şunlardır:

### **1. Authentication**

- Uygulama bir kimlik doğrulama mekanizması kullanıyor mu?
    
- Hangi kimlik doğrulama yöntemleri (şifre, OTP, 2FA vb.) kullanılıyor?
    

### **2. Authorization**

- Farklı kullanıcı rolleri tanımlanmış mı?
    
- Her istek için yetkilendirme kontrolü yapılıyor mu?
    

### **3. Data Validation**

- Kullanıcıdan alınan veriler nasıl doğrulanıyor?
    
- Doğrulama işlemi veriler alındığı anda mı, yoksa kullanılmadan önce mi gerçekleşiyor?
    
- Whitelist mi yoksa blacklist mi kullanılıyor?
    

### **4. Session Management**

- Oturumlar güvenli bir şekilde yönetiliyor mu?
    
- Session ID’ler nasıl oluşturuluyor ve korunuyor?
    
- Uygulama, logout sonrası oturumu geçersiz kılabiliyor mu?
    

> Bu kontroller yalnızca bir örnektir. Daha fazla kontrol eklenebilir, çıkartılabilir veya kontroller değiştirilebilir.

## **Manuel ve Otomatik Test Karşılaştırması**

### **Manuel İnceleme**

- İş mantığına dair hataların tespitinde etkilidir.
    
- Ancak zaman alıcıdır ve uzmanlık gerektirir.
    

### **SAST**

- **Avantajları:**
    
    - Daha hızlı sonuç,
        
    - Daha az manuel çaba,
        
    - Tekrarlanabilirlik ve sürekli entegrasyon ile uyum.
        
- **Dezavantajları:**
    
    - İş mantığı hatalarını tespit edemez,
        
    - False-positive üretme olasılığı,
        
    - Kaynak kısıtlamaları (dil desteği, konfigürasyon zorlukları).
        

## **Kod İnceleme Sırasında Dikkat Edilecek Noktalar**

Küçük kod parçalarını incelerken aşağıdaki faktörler önemlidir:

- Nesne yönelimli programlama bilgisi,
    
- Uygulamanın konfigürasyonu, amacı ve iş mantığının anlaşılması,
    
- Veri akışlarının ve API’lerin analizi,
    
- Üçüncü parti kütüphanelerin ve bağımlılıkların güvenliği.
    

## **Kritik Kavramlar**

### **Source**

Kötü amaçlı verinin kullanıcıdan uygulamaya ilk giriş yaptığı yerdir.

### **Sink**

Verinin, güvenlik açığı oluşturabilecek şekilde işlendiği noktadır.

### **Taint**

Kaynağından itibaren güvensiz olduğu belirlenen veridir.

---

## **Yaygın Güvenlik Zafiyetleri**

### **1. Cross-Site Scripting (XSS)**

- Kullanıcıdan alınan veriler filtrelenmeden HTML içeriğine yansıtıldığında oluşur.
    
- Kullanıcıların tarayıcılarında kötü amaçlı JavaScript çalıştırılabilir.
    
- **Çözüm:** Input validation, output encoding, güvenli şablonlama.
    
- **Manual Test Yöntemi:** Kullanıcı girdisinin `innerHTML`, `document.write` veya _template engine_ gibi yerlere doğrudan yansıtılıp yansıtılmadığı kontrol edilir.
    

### **2. SQL Injection (SQLi)**

- Kullanıcı girdisinin doğrudan SQL sorgularında kullanılması sonucu gerçekleşir.
    
- Saldırganlar veri sızdırabilir, değiştirebilir ya da veri tabanını silebilir.
    
- **Çözüm:** Prepared statement kullanımı, ORM tercih edilmesi.
    
- **Manual Test Yöntemi:** Raw SQL sorgularının varlığı kontrol edilir. Hazır kütüphane kullanılıyorsa, kütüphanenin sorguları nasıl oluşturduğu analiz edilir.
    

### **3. Path Traversal**

- Dosya yollarında kullanıcı girdisinin kontrolsüz kullanılmasıyla dizin dışı dosyalara erişim sağlanır.
    
- Örnek saldırı: `../../../etc/passwd`
    
- **Çözüm:** Dosya yollarını sabit/izinli dizinlerle sınırlandırmak, path sanitizasyonu.
    
- **Manual Test Yöntemi:** Dosya çağıran fonksiyonlar kontrol edilir.
    

### **4. Command Injection**

- Kullanıcı girdisi sistem komutlarına entegre edilerek komut çalıştırılmasına yol açar.
    
- Saldırgan `;`, `&&`, `|` gibi karakterlerle yeni komutlar ekleyebilir.
    
- **Çözüm:** Komut satırına kullanıcı verisi göndermekten kaçınmak, güvenli API'ler kullanmak.
    
- **Manual Test Yöntemi:** Kullanıcıdan alınan verilerin işletim sisteminde komut çalıştırma sırasında kullanılıp kullanılmadığı kontrol edilir. `shell=False` gibi önlemler alındıysa, argument injection kontrol edilir.
    

### **5. Server-Side Request Forgery (SSRF)**

- Uygulama, kullanıcının belirlediği bir URL’ye sunucu tarafından istek gönderdiğinde ve bu istek filtrelenmeden gerçekleştirilirse oluşur.
    
- Saldırgan iç ağdaki servisleri tarayabilir, metadata servislerine erişebilir (örn: AWS metadata endpoint).
    
- **Çözüm:** Hedef URL’leri whitelist’le sınırlamak, IP doğrulaması yapmak, iç IP aralığını filtrelemek.
    
- **Manual Test Yöntemi:** Kullanıcıdan alınan verilerin uygulamanın HTTP isteği yollayacağı URL'de kullanılıp kullanılmadığı kontrol edilir.
    

### **6. Insecure Deserialization**

- Kullanıcıdan gelen verilerin doğrudan nesneye dönüştürülmesiyle saldırgan, uygulama üzerinde zararlı nesneler çalıştırabilir.
    
- Bu zafiyet uzaktan kod çalıştırma (RCE) gibi ciddi sonuçlara yol açabilir.
    
- **Çözüm:** Güvenli ve doğrulanmış serileştirme formatları kullanmak (JSON, protobuf), mümkünse deserialization’dan kaçınmak.
    
- **Manual Test Yöntemi:** Kullanıcıdan alınan verinin deserialize edilip edilmediği kontrol edilir.
    

### **7. Server-Side Template Injection (SSTI)**

- Şablon motorlarının kullanıcı verisini doğrudan yorumlaması sonucu oluşur.
    
- Örnek: `{{7*7}}` ifadesinin `49` olarak render edilmesi.
    
- Saldırgan bu mekanizmayla dosya okuma, kod yürütme gibi saldırılar gerçekleştirebilir.
    
- **Çözüm:** Kullanıcı verisini şablon motoruna doğrudan geçirmemek, güvenli şablonlama kullanmak.
    
- **Manual Test Yöntemi:** Kullanıcıdan alınan verinin doğrudan _template engine_'e aktarılıp aktarılmadığı kontrol edilir.
