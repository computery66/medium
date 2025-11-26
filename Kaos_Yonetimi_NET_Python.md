# ⚙️ Kaos Yönetimi: .NET ve Python’da Dayanıklı Sistem Tasarımı  
### (Chaos Management in .NET and Python)

![Chaos Management Overview](https://files.catbox.moe/2n4pl5.png)

> **“Kaos yönetimi, yazılımın gerçek dünyadaki beklenmedik durumlara nasıl tepki verdiğini anlamanın sanatıdır.”**

Modern yazılım sistemleri artık sadece koddan ibaret değil; mikro servisler, bulut tabanlı bileşenler, üçüncü parti API’ler ve mesaj kuyrukları arasında yaşayan karmaşık bir ekosistem.  
Bu yapıların “çökmeden” çalışabilmesi, yani **dayanıklı (resilient)** olması ise doğrudan “kaos yönetimi” yaklaşımlarına bağlıdır.

---

## 🔍 Kaos Yönetimi Nedir?
Kaos Yönetimi (Chaos Management), sistemlerin hata koşullarında nasıl davrandığını anlamak, hata toleransını artırmak ve geri dönüş stratejilerini test etmek için kullanılan bir mühendislik yaklaşımıdır.  
Amaç, sistemleri **bilinçli olarak stres altına almak** ve onların nasıl tepki verdiğini gözlemlemektir.

### 💡 Örnek Durum:
Bir .NET API’si düşünün — dış bir servise bağlanıp veri çekiyor.  
Bu servis aniden yanıt vermezse ne olur?  
İşte kaos yönetimi burada devreye girer.

---

## 🧱 Temel Prensipler

1. **Hata Tolere Etme (Fault Tolerance)**  
   Her sistem bileşeni, diğerinin çökmesini tolere edebilmelidir.

2. **Gözlemlenebilirlik (Observability)**  
   Loglama, metrikler ve tracing araçları (OpenTelemetry, Prometheus, Application Insights vb.) sistemin nabzını tutar.

3. **Kendini İyileştirme (Self-Healing)**  
   Hata sonrası otomatik olarak yeniden deneme (retry), yönlendirme veya devre kesici mekanizmaları devreye girer.

---

## 🧩 .NET Dünyasında Kaos Yönetimi

### 🔁 Retry ve Timeout Pattern

![Retry and Timeout Flow](https://files.catbox.moe/4btqiw.png)

.NET’te en popüler kütüphanelerden biri **Polly**’dir.  
Polly, retry, timeout, circuit breaker gibi dayanıklılık stratejilerini uygulamak için kullanılır.

#### 🧠 Örnek: Polly ile Retry Politikası
```csharp
var policy = Policy
    .Handle<HttpRequestException>()
    .WaitAndRetryAsync(3, attempt => 
        TimeSpan.FromSeconds(Math.Pow(2, attempt)), 
        (exception, timeSpan, retryCount, context) =>
        {
            Console.WriteLine($"Deneme {retryCount} başarısız. {timeSpan.TotalSeconds} saniye sonra yeniden deneniyor...");
        });

await policy.ExecuteAsync(async () =>
{
    var response = await httpClient.GetAsync("https://api.service.com/data");
    response.EnsureSuccessStatusCode();
});
```

Bu örnekte:  
- **3 kez yeniden deneme** yapılıyor.  
- Her denemede **üstel artan bekleme süresi** uygulanıyor.  
- Başarısız her istekte log kaydı tutuluyor.

---

## 🧠 Circuit Breaker (Devre Kesici) Pattern

![Circuit Breaker Diagram](https://files.catbox.moe/1d2u3q.png)

Circuit Breaker, sistemin sürekli hatalı servislere istek göndermesini önler.  
Yani “bile bile kötü servise gitme” prensibine dayanır.

#### 🧩 Polly ile Circuit Breaker:
```csharp
var circuitBreakerPolicy = Policy
    .Handle<HttpRequestException>()
    .CircuitBreakerAsync(
        exceptionsAllowedBeforeBreaking: 2,
        durationOfBreak: TimeSpan.FromSeconds(30),
        onBreak: (ex, breakDelay) =>
        {
            Console.WriteLine($"Circuit açık! {breakDelay.TotalSeconds} saniye bekleniyor...");
        },
        onReset: () => Console.WriteLine("Circuit sıfırlandı."),
        onHalfOpen: () => Console.WriteLine("Circuit yarı açık durumda, test isteği gönderiliyor.")
    );

await circuitBreakerPolicy.ExecuteAsync(async () =>
{
    await httpClient.GetAsync("https://api.service.com/data");
});
```

---

## 🐍 Python Dünyasında Kaos Yönetimi

Python tarafında en yaygın kullanılan kütüphane **Tenacity**’dir.  
Tenacity, basit ama güçlü bir yeniden deneme (retry) mekanizması sağlar.

#### 🧠 Tenacity ile Retry:
```python
from tenacity import retry, wait_exponential, stop_after_attempt

@retry(wait=wait_exponential(multiplier=1, min=2, max=10), stop=stop_after_attempt(5))
def fetch_data():
    print("Veri çekiliyor...")
    response = requests.get("https://api.service.com/data")
    response.raise_for_status()
    return response.json()

fetch_data()
```

Bu örnekte:  
- **5 kez yeniden deneme** yapılır.  
- Her hata sonrası bekleme süresi üstel olarak artar.  
- Maksimum 10 saniyeye kadar beklenir.

---

## 🧰 İzleme (Monitoring) ve Telemetri

![Monitoring and Telemetry](https://files.catbox.moe/8z6c3n.png)

Kaos yönetiminin kalbinde **izleme (monitoring)** vardır.  
Sistemin ne durumda olduğunu bilmeden dayanıklılığını ölçemezsiniz.

### .NET için:
- **Application Insights**
- **Serilog + Seq**
- **OpenTelemetry Collector**

### Python için:
- **Prometheus Client**
- **Sentry**
- **Grafana Agent**

---

## 🔄 Kaos Testleri (Chaos Experiments)

### 💥 Örnek: Servis Kesintisi Testi
Bir test ortamında rastgele mikro servisler devre dışı bırakılarak, sistemin nasıl tepki verdiği ölçülür.

#### Bash örneği:
```bash
kubectl delete pod my-service-pod-1
```

Ardından dashboard’dan hata oranı, latency ve recovery süresi gözlemlenir.

---

## 🧭 Sonuç

Kaos yönetimi, sadece hata senaryolarını ele almak değil, **sistemi hata yapmaya teşvik etmek** ve bu hatalardan **öğrenmek** felsefesidir.  
.NET ve Python ekosistemleri, bu konuda güçlü kütüphaneler ve gözlemlenebilirlik araçları sunar.

> “Dayanıklı sistemler, hiç hata yapmayanlar değil; hata yaptığında ayağa kalkabilenlerdir.”

---

## 📚 Kaynakça

- Microsoft Docs – [Polly Resilience Library](https://github.com/App-vNext/Polly)
- Tenacity – [Python Retry Library](https://tenacity.readthedocs.io/)
- Netflix Chaos Monkey
- OpenTelemetry & Prometheus Docs

---

**Yazar:** Fatma  
*Full-Stack Mobile Developer | Kaos Yönetimi & Dayanıklılık Meraklısı*

![Footer Graphic](https://files.catbox.moe/2n4pl5.png)
