<h1 align="center" style="color:red;">
  Hafta 2
</h1>

<h2 style="text-align:left; color:yellow;">
  1. Gün: AI Digital Twin Projesine Giriş ve Hafıza Problemi
</h2>

Bu hafta, "AWS üzerinde AI Platform Mühendisliği" konusuna odaklanılarak, "Digital Twin" adında yeni bir proje geliştirilmeye başlandı. Gün, önceki haftadan kalan AWS kaynaklarının temizlenmesi, yeni bulut mimarilerinin teorik anlatımı ve projenin yerel ortamda temel bir versiyonunun oluşturulması ile geçti.

### Haftaya Giriş ve AWS Temizliği

1.  **Maliyet Kontrolü:** AWS Console'da "Billing and Cost Management" servisi üzerinden geçen haftaki `consultation-app` uygulamasının maliyetleri incelendi.
2.  **Kaynakların Silinmesi:**
    *   **App Runner:** Aktif olarak çalışan `consultation-app-service`, maliyet oluşturmaması için "Actions -> Delete" seçeneği ile kalıcı olarak silindi.
    *   **ECR (Elastic Container Registry):** `consultation-app` Docker imajını barındıran depo da temizlendi.

### Teorik Altyapı ve Yeni Proje Mimarisi

Eğitmen, bu hafta kullanılacak AWS servislerini ve genel bulut dağıtım mimarilerini tanıttı.

*   **Bulut Dağıtım Arketipleri:**
    *   **IaaS (EC2):** Sunucuyu kiralayıp her şeyi kendin kurduğun temel model.
    *   **PaaS (Vercel, Beanstalk):** Sadece kodu yüklediğin, altyapının platform tarafından yönetildiği model.
    *   **CaaS (App Runner):** Uygulamayı bir konteyner içinde verdiğin, geri kalanını servisin hallettiği model.
    *   **Kubernetes (ECS/EKS):** Kendi konteyner filonu yönettiğin ve ölçeklendirdiğin gelişmiş model.
    *   **FaaS (Lambda):** Tek tek küçük fonksiyonlar yükleyip, istek başına ödeme yaptığın sunucusuz model.

*   **Bu Hafta Kullanılacak AWS Servisleri ("Amazon Biology"):**
    *   **Backend:** Lambda (İş Mantığı), S3 (Konuşma Geçmişi için Hafıza), API Gateway (API Rotaları).
    *   **Frontend:** S3 (Statik Site Dosyaları), CloudFront (CDN ile Hızlı Dağıtım).
    *   **AI Model:** Bedrock (LLM'lere erişim için yönetilen servis).

### Bölüm 1: Proje Kurulumu ve Yapılandırma

"Digital Twin" projesi için sıfırdan bir çalışma ortamı hazırlandı.

1.  **Proje Klasörünün Oluşturulması:**
    *   Cursor'da `twin` adında yeni ve boş bir proje klasörü oluşturuldu.
    *   Bu klasörün içine `backend` ve `memory` adında iki yeni alt klasör eklendi.

2.  **Frontend'in Başlatılması (Next.js App Router):**
    *   Terminalde, `twin` ana dizinindeyken, aşağıdaki komut çalıştırılarak `frontend` adında yeni bir Next.js projesi oluşturuldu:
        ```bash
        npx create-next-app@latest frontend --typescript --tailwind --app --no-src-dir
        ```
    *   **Önemli Fark:** Bu hafta, daha modern ve performanslı olan **App Router** mimarisi kullanıldı (1. Haftadaki Pages Router yerine).

3.  **Paket Yöneticisi (UV):**
    *   Hızlı ve modern bir Python paket yöneticisi olan `uv`, talimatlar takip edilerek sisteme kuruldu.

### Bölüm 2: Geliştirme Ortamı ve Backend'in Oluşturulması

1.  **Backend Dosyalarının Oluşturulması:**
    *   `backend/requirements.txt`: Gerekli Python kütüphaneleri (`fastapi`, `uvicorn`, `openai` vb.) listelendi.
    *   `backend/.env`: `OPENAI_API_KEY` ve `CORS_ORIGINS` gibi ortam değişkenleri tanımlandı.
    *   `backend/me.txt`: Digital Twin'in kişiliğini, rolünü ve hedeflerini tanımlayan bir metin dosyası oluşturuldu. Bu dosya, yapay zekanın "system prompt"u olarak kullanılacaktır.
    *   `backend/server.py`: İlk versiyon FastAPI sunucusu yazıldı. Bu sunucu, `/chat` adresine gelen POST isteklerini alır, `me.txt` dosyasındaki kişilikle birlikte kullanıcı mesajını OpenAI'ye gönderir ve yanıtı geri döndürür. **Bu ilk versiyonda herhangi bir hafıza mekanizması bulunmamaktadır.**

### Bölüm 3: Frontend Arayüzünün Geliştirilmesi

1.  **Chat Bileşeninin Oluşturulması (`twin.tsx`):**
    *   `frontend/components` klasörü içinde, sohbet arayüzünün tüm mantığını ve görsel yapısını içeren `twin.tsx` adında bir React bileşeni oluşturuldu.
    *   Bu bileşen, mesajları, kullanıcı girdisini ve yüklenme durumunu `useState` ile yönetir.

2.  **Gerekli Kütüphanenin Kurulması:**
    *   Arayüzde kullanılan ikonlar için `lucide-react` kütüphanesi kuruldu:
        ```bash
        cd frontend
        npm install lucide-react
        cd ..
        ```
3.  **Ana Sayfa ve Stil Güncellemeleri:**
    *   `frontend/app/page.tsx`: Ana sayfa, oluşturulan `Twin` bileşenini gösterecek şekilde güncellendi.
    *   `frontend/postcss.config.mjs` ve `frontend/app/globals.css`: Yeni Tailwind CSS v4 versiyonu ile uyumlu olacak şekilde stil yapılandırma dosyaları güncellendi.

### Bölüm 4: Hafızasız Versiyonun Test Edilmesi ve Problem Tespiti

1.  **Sunucuların Başlatılması:**
    *   **Backend:** Bir terminalde `backend` dizinine gidilerek `uv` ile sanal ortam oluşturuldu ve FastAPI sunucusu başlatıldı:
        ```bash
        cd backend
        uv init --bare
        uv python pin 3.12
        uv add -r requirements.txt
        uv run uvicorn server:app --reload
        ```
    *   **Frontend:** Ayrı bir terminalde `frontend` dizinine gidilerek Next.js geliştirme sunucusu başlatıldı:
        ```bash
        cd frontend
        npm run dev
        ```

2.  **Hafıza Probleminin Tespiti:**
    *   Tarayıcıda `http://localhost:3000` adresi açıldı.
    *   Sohbet robotuyla yapılan test konuşmasında ("Benim adım Alex." -> "Benim adım ne?"), robotun önceki mesajları hatırlamadığı net bir şekilde görüldü. Her istek, bağlamdan kopuk ve bağımsız olarak işleniyordu.

### Bölüm 5: Dosya Tabanlı Hafıza Entegrasyonu

Tespit edilen hafıza problemini çözmek için backend'e basit ama etkili bir dosya tabanlı hafıza sistemi eklendi.

1.  **Backend Mantığının Güncellenmesi (`server.py`):**
    *   `server.py` dosyasının içeriği, hafıza yönetimi yetenekleri eklenmiş yeni versiyonuyla tamamen değiştirildi.
    *   **`load_conversation` ve `save_conversation` Fonksiyonları:**
        *   Bu iki fonksiyon, her bir konuşma oturumu (`session_id`) için `memory/` klasörü altında bir JSON dosyası oluşturur.
        *   Yeni bir mesaj geldiğinde, ilgili oturumun geçmişi bu dosyadan okunur (`load`), yeni mesajlar eklenerek OpenAI'ye gönderilir ve konuşmanın güncel hali tekrar dosyaya yazılır (`save`).
2.  **Yeniden Test:**
    *   Backend sunucusu yeniden başlatıldıktan sonra, arayüzde yapılan yeni testte sohbet robotunun artık önceki mesajları hatırladığı ve bağlamı koruduğu doğrulandı. `memory/` klasörünün içinde konuşma geçmişini içeren JSON dosyalarının oluştuğu gözlemlendi.

### Not: 
Eğer mevcut bilgisayarda 3000 portunda baska bir uygulama calisiyor ve bu port mesgul ise 'Digital Twin' uygulamasinin hata vermemesi icin su degisiklikler yapilmalidir: 

### 1. Frontend (Next.js) Port Değişikliği

Next.js geliştirme sunucusunun hangi portta çalışacağını belirtmek için:

*   **Dosya:** `frontend/package.json`
*   **Değişiklik:** `scripts` bölümündeki `dev` komutunu bulun ve sonuna `-p 3001` ekleyin (veya dilediginiz baska bir port, örnegin -p 3002).

**Eski Hali:**
```json
"scripts": {
  "dev": "next dev",
  ...
}
```

**Yeni Hali:**
```json
"scripts": {
  "dev": "next dev -p 3001",
  ...
}
```

### 2. Backend (FastAPI) CORS Ayarı

Frontend'in yeni port adresinden gelen isteklere backend'in izin vermesi için CORS ayarını güncellemelisiniz.

*   **Dosya:** `backend/.env`
*   **Değişiklik:** `CORS_ORIGINS` değişkenindeki portu `3001` olarak değiştirin.

**Eski Hali:**
```
CORS_ORIGINS=http://localhost:3000
```

**Yeni Hali:**
```
CORS_ORIGINS=http://localhost:3001
```

Bu iki değişikliği yaptıktan sonra hem backend (`uv run uvicorn server:app --reload`) hem de frontend (`npm run dev`) sunucularını yeniden başlatın. Artık uygulamanız `http://localhost:3001` adresinde sorunsuz çalışacaktır.


<h2 style="text-align:left; color:yellow;">
  2. Gün: Digital Twin Projesinin AWS Üzerine Dağıtımı (Lambda, S3, API Gateway, CloudFront)
</h2>

Bu günün ana hedefi, dün yerel ortamda geliştirilen Digital Twin uygulamasını kişisel verilerle zenginleştirmek ve ardından tamamen sunucusuz (serverless) bir mimari kullanarak AWS üzerinde canlıya almaktı.

### Bölüm 1: Digital Twin'in Kişisel Verilerle Zenginleştirilmesi

Uygulamanın yapay zeka modeline daha zengin ve kişisel bir bağlam sunmak için backend'e yeni veri kaynakları ve modüller eklendi.

1.  **Veri Dosyalarının Oluşturulması:**
    *   `backend/data` adında yeni bir klasör oluşturuldu.
    *   Bu klasörün içine, dijital ikizin temsil ettiği kişi hakkında yapılandırılmış bilgiler içeren şu dosyalar eklendi:
        *   `facts.json`: İsim, rol, konum gibi temel bilgiler.
        *   `summary.txt`: Kişisel ve profesyonel bir özet metni (`me.txt` dosyasının ismi değiştirilerek kullanıldı).
        *   `style.txt`: İletişim tarzını belirten notlar.
        *   `linkedin.pdf`: LinkedIn profilinin PDF olarak kaydedilmiş hali.

2.  **Veri İşleme Modüllerinin Yazılması:**
    *   `backend/resources.py`: Oluşturulan veri dosyalarını (PDF, JSON, TXT) okuyup Python içinde kullanılabilir değişkenlere atayan bir modül yazıldı.
    *   `backend/context.py`: `resources.py` modülünden gelen verileri, yapay zeka modeline verilecek olan ana system prompt'u oluşturmak için bir araya getiren bir modül yazıldı. Bu prompt, modele rolünü, bağlamı, iletişim tarzını ve uyması gereken kuralları detaylı bir şekilde anlatır.

3.  **Sunucu ve Bağımlılıkların Güncellenmesi:**
    *   `backend/requirements.txt`: AWS ile etkileşim için `boto3`, PDF okumak için `pypdf` ve Lambda uyumluluğu için `mangum` kütüphaneleri eklendi.
    *   `backend/server.py`: Sunucu kodu, AWS'de çalışacak şekilde yeniden düzenlendi. Artık ortam değişkenlerine (`USE_S3`) bağlı olarak konuşma geçmişini ya yerel `memory` klasörüne ya da AWS S3'ye kaydedebilecek esnek bir yapıya kavuştu.
    *   `backend/lambda_handler.py`: FastAPI uygulamasını AWS Lambda ortamında çalıştırılabilir hale getiren `mangum` kütüphanesini kullanarak bir sarmalayıcı (handler) dosyası oluşturuldu.

4.  **Yerel Test:**
    *   Yapılan tüm bu geliştirmelerden sonra, backend ve frontend sunucuları yeniden başlatılarak zenginleştirilmiş Digital Twin'in yerel ortamda sorunsuz çalıştığı teyit edildi.

### Bölüm 2: AWS Ortamının Hazırlanması ve Lambda Paketinin Oluşturulması

1.  **IAM Yetkilendirmesi:**
    *   `aiengineer` kullanıcısının yeni AWS servislerini kullanabilmesi için, IAM üzerinde `TwinAccess` adında yeni bir kullanıcı grubu oluşturuldu.
    *   Bu gruba `Lambda`, `S3`, `API Gateway`, `CloudFront` ve `IAM` servisleri için gerekli olan tam erişim veya okuma yetkileri verildi ve `aiengineer` kullanıcısı bu gruba eklendi.

2.  **Lambda Dağıtım Paketinin Hazırlanması:**
    *   `backend/deploy.py`: Lambda'ya yüklenecek olan `.zip` paketini otomatikleştiren bir Python betiği yazıldı. Bu betik, Docker kullanarak bağımlılıkları Lambda'nın çalışma ortamıyla tam uyumlu (`linux/amd64`) bir şekilde indirir, ardından uygulama dosyalarıyla birleştirerek `lambda-deployment.zip` dosyasını oluşturur.
    *   `uv run deploy.py` komutu çalıştırılarak bu dağıtım paketi başarıyla oluşturuldu.

### Bölüm 3: Backend'in AWS Lambda ve S3'e Dağıtımı

1.  **Lambda Fonksiyonunun Oluşturulması:**
    *   AWS Lambda servisinde `twin-api` adında, Python 3.12 ve x86_64 mimarisini kullanan yeni bir fonksiyon oluşturuldu.
    *   Oluşturulan `lambda-deployment.zip` paketi bu fonksiyona yüklendi.
    *   **Handler:** `lambda_handler.handler` olarak ayarlandı.
    *   **Timeout:** 30 saniyeye çıkarıldı.
    *   **Ortam Değişkenleri:** `OPENAI_API_KEY`, `CORS_ORIGINS`, `USE_S3=true` ve `S3_BUCKET` (henüz oluşturulmadı) tanımlandı.
    *   Fonksiyonun sağlıklı çalışıp çalışmadığı, `/health` yolunu test eden bir test olayı (event) ile doğrulandı.

2.  **S3 Bucket'larının Oluşturulması:**
    *   **Hafıza Bucket'ı:** `twin-memory-[hesap-no]` adıyla, konuşma geçmişi JSON dosyalarını saklamak için özel (private) bir S3 bucket'ı oluşturuldu.
    *   **Frontend Bucket'ı:** `twin-frontend-[hesap-no]` adıyla, statik web sitesi dosyalarını barındırmak için halka açık (public) bir S3 bucket'ı oluşturuldu. Bu bucket için "Static website hosting" özelliği etkinleştirildi ve herkesin `s3:GetObject` iznine sahip olmasını sağlayan bir bucket policy eklendi.

3.  **Lambda S3 İzinlerinin Yapılandırılması:**
    *   Lambda fonksiyonunun `twin-memory` bucket'ına yazıp okuyabilmesi için, Lambda'nın execution rolüne `AmazonS3FullAccess` politikası eklendi.

### Bölüm 4: API Gateway ve CloudFront ile Uygulamanın Canlıya Alınması

1.  **API Gateway Kurulumu:**
    *   Gelen HTTP isteklerini Lambda fonksiyonuna yönlendirmek için `twin-api-gateway` adında bir HTTP API oluşturuldu.
    *   `/`, `/health`, `/chat` ve `/{proxy+}` gibi yollar (routes) için Lambda entegrasyonu tanımlandı.
    *   Farklı kaynaklardan (origin) gelen isteklerin kabul edilmesi için CORS (Cross-Origin Resource Sharing) ayarları `*` (herkese izin ver) olarak yapılandırıldı.
    *   Oluşturulan API'nin `Invoke URL`'i test edildi ve Lambda'dan başarılı yanıt alındığı görüldü.

2.  **Frontend'in Dağıtımı:**
    *   `frontend/components/twin.tsx` dosyasındaki `fetch` adresi, localhost yerine yeni oluşturulan API Gateway `Invoke URL`'i ile güncellendi.
    *   `npm run build` komutu ile frontend projesinin statik `out` klasörü oluşturuldu.
    *   `aws s3 sync` komutu kullanılarak `out` klasörünün içeriği, `twin-frontend` S3 bucket'ına yüklendi.

3.  **CloudFront Dağıtımının Yapılandırılması:**
    *   **Amaç:** Uygulamaya global bir CDN (Content Delivery Network) üzerinden hızlı ve HTTPS ile güvenli erişim sağlamak.
    *   Yeni bir CloudFront dağıtımı (`twin-distribution`) oluşturuldu.
    *   **Origin (Kaynak):** `twin-frontend` S3 bucket'ının "static website endpoint" URL'si kaynak olarak belirtildi. **Önemli:** Origin protocol policy olarak `HTTP only` seçildi.
    *   **Viewer (İzleyici):** İzleyici isteklerinin `HTTP`'den `HTTPS`'e yönlendirilmesi ayarlandı.
    *   Dağıtım tamamlandıktan sonra, CloudFront tarafından sağlanan `Distribution domain name` (örn: `d204k92dr41083.cloudfront.net`) kopyalandı.

4.  **Son Güvenlik Ayarı ve Test:**
    *   Güvenliği artırmak için Lambda fonksiyonunun `CORS_ORIGINS` ortam değişkeni, `*` yerine CloudFront'un alan adıyla güncellendi. Bu, API'nin yalnızca kendi CloudFront dağıtımından gelen istekleri kabul etmesini sağlar.
    *   Son olarak, CloudFront alan adı tarayıcıda açılarak uygulamanın canlı ortamda, HTTPS ile ve S3'e kaydedilen hafıza özelliğiyle tam fonksiyonel çalıştığı başarıyla test edildi.


**NOT:** 

Uygulamanin artik yerelde calismamasinin nedeni, AWS Lambda'daki `CORS_ORIGINS` ayarını sadece CloudFront adresine izin verecek şekilde değiştirmiş olmamızdır. Şu anda AWS, `localhost:3001`'den gelen istekleri reddediyor.

**Yereldeki çalistirmak için:**

1.  `frontend/components/twin.tsx` dosyasını açın.
2.  `fetch` satırını tekrar `localhost:8000` olarak değiştirin.

    *   **Eski Hali (AWS URL'si):**
        ```javascript
        const response = await fetch('https://qpmobb4cg8.execute-api.eu-central-1.amazonaws.com/chat', {
        ```
    *   **Yeni Hali (Yerel URL):**
        ```javascript
        const response = await fetch('http://localhost:8000/chat', {
        ```

3.  Değişikliği kaydedin ve backend sunucunuzun (`uv run uvicorn server:app --reload`) çalıştığından emin olun.

Artık yerel ortamınız da eskisi gibi çalışacaktır.


<h2 style="text-align:left; color:yellow;">
  3. Gün: OpenAI'den AWS Bedrock'a Geçiş
</h2>

Bu günün ana odak noktası, Digital Twin uygulamasının yapay zeka altyapısını harici bir servis olan OpenAI'den, AWS ekosistemi içinde yer alan yönetilen bir servis olan **Amazon Bedrock**'a taşımaktı. Bu değişiklik, daha düşük gecikme süresi, potansiyel maliyet avantajları ve diğer AWS servisleriyle daha derin entegrasyon gibi kurumsal faydalar sağlamaktadır.

### Bölüm 1: AWS İzinlerinin ve Bedrock Model Erişiminin Yapılandırılması

1.  **IAM İzinlerinin Genişletilmesi:**
    *   AWS Console'a **root kullanıcı** ile giriş yapıldı.
    *   IAM servisinde, `aiengineer` kullanıcısının dahil olduğu `TwinAccess` kullanıcı grubuna gidildi.
    *   Grubun yetkilerine (permissions), Bedrock servisine ve gelişmiş izleme için CloudWatch'a tam erişim sağlayan iki yeni politika eklendi:
        *   `AmazonBedrockFullAccess`
        *   `CloudWatchFullAccess`
    *   Bu adımdan sonra root hesabından çıkış yapılıp, `aiengineer` IAM kullanıcısı ile devam edildi.

2.  **Bedrock Model Erişimi:**
    *   Amazon Bedrock servisine gidildi.
    *   Sol menüdeki "Model access" bölümünden, AWS'in kendi geliştirdiği **Nova** model ailesine erişim talebinde bulunuldu.
    *   Erişim, aşağıdaki üç model için talep edildi ve anında onaylandı ("Access granted"):
        *   **Nova Micro:** En hızlı ve en uygun maliyetli model.
        *   **Nova Lite:** Dengeli performans sunan genel amaçlı model.
        *   **Nova Pro:** En yetenekli ancak daha yüksek maliyetli model.

### Bölüm 2: Uygulama Kodunun Bedrock için Güncellenmesi

Uygulamanın backend'i, OpenAI yerine Bedrock ile iletişim kuracak şekilde yeniden düzenlendi.

1.  **Bağımlılıkların Düzenlenmesi (`requirements.txt`):**
    *   Artık kullanılmayacağı için `openai` kütüphanesi `requirements.txt` dosyasından kaldırıldı.

2.  **Sunucu Kodunun Yeniden Yazılması (`server.py`):**
    *   `server.py` dosyasının içeriği, Bedrock entegrasyonunu içerecek şekilde tamamen güncellendi.
    *   **Önemli Değişiklikler:**
        *   OpenAI istemcisi yerine, AWS SDK'sı (`boto3`) kullanılarak bir **Bedrock istemcisi** (`bedrock-runtime`) başlatıldı.
        *   Kullanılacak modelin ID'si (`BEDROCK_MODEL_ID`), bir ortam değişkeni aracılığıyla dinamik olarak ayarlandı. Varsayılan olarak `amazon.nova-lite-v1:0` seçildi.
        *   OpenAI API çağrısı yapan mantık, Bedrock'un `converse` API'sini çağıran yeni bir `call_bedrock` fonksiyonu ile değiştirildi. Bu fonksiyon, Bedrock'un beklediği mesaj formatına uygun bir yapı oluşturur.
        *   Bedrock'a özgü olası hataları (örn: `AccessDeniedException`) yakalamak için daha detaylı bir hata yönetimi (error handling) eklendi.

### Bölüm 3: Güncellenmiş Uygulamanın AWS Lambda'ya Dağıtımı

1.  **Lambda Ortam Değişkenlerinin Güncellenmesi:**
    *   `twin-api` Lambda fonksiyonunun yapılandırmasına gidildi.
    *   `OPENAI_API_KEY` ortam değişkeni silindi.
    *   Yerine, kullanılacak Bedrock modelini belirten `BEDROCK_MODEL_ID` ve Lambda'nın hangi AWS bölgesinde çalıştığını belirten `DEFAULT_AWS_REGION` eklendi.

2.  **Lambda İzinlerinin Güncellenmesi:**
    *   Lambda fonksiyonunun Bedrock servisini çağırabilmesi için, fonksiyonun execution rolüne `AmazonBedrockFullAccess` politikası eklendi.

3.  **Dağıtım Paketinin Yeniden Oluşturulması ve Yüklenmesi:**
    *   `requirements.txt` dosyasındaki değişiklikler nedeniyle, `uv run deploy.py` komutu tekrar çalıştırılarak `lambda-deployment.zip` paketi yeniden oluşturuldu.
    *   Güncellenmiş bu paket, `.zip file` yükleme seçeneği ile Lambda fonksiyonuna yüklendi ve kaydedildi.
    *   Yükleme sonrası, Lambda'nın test özelliği kullanılarak `/health` endpoint'i test edildi ve yanıtın artık Bedrock model bilgisini içerdiği doğrulandı.

### Bölüm 4: Test, İzleme ve Canlı Ortam Doğrulaması

Bu bölümde, AWS Bedrock entegrasyonu tamamlanmış uygulamanın canlı ortamda doğru çalıştığı doğrulandı ve performansını takip etmek için AWS'in izleme servisi olan CloudWatch üzerinde detaylı incelemeler ve yapılandırmalar yapıldı.

#### Adım 1: Canlı Uygulamanın Test Edilmesi

Uygulamanın son kullanıcıya sunulan versiyonunun beklendiği gibi çalışıp çalışmadığını doğrulamak için aşağıdaki adımlar izlendi:

1.  **CloudFront URL'sinin Alınması:**
    *   AWS Management Console'a `aiengineer` kullanıcısı ile giriş yapıldı.
    *   Arama çubuğuna **"CloudFront"** yazılarak servise gidildi.
    *   Listelenen dağıtımlar (distributions) arasından bir önceki gün oluşturulan `twin-distribution` bulundu ve üzerine tıklandı.
    *   Açılan genel bakış sayfasında, **"Distribution domain name"** başlığı altındaki URL (örneğin: `https://d204k92dr41083.cloudfront.net`) kopyalandı. Bu, uygulamanın internetteki herkese açık adresidir.

2.  **Fonksiyonel Test:**
    *   Kopyalanan URL, bir web tarayıcısının adres çubuğuna yapıştırıldı ve sayfa açıldı. Digital Twin'in sohbet arayüzünün başarıyla yüklendiği görüldü.
    *   Sohbet kutusuna birkaç mesaj yazılarak bir diyalog başlatıldı. Örneğin:
        *   "Merhaba, kendini tanıtır mısın?"
        *   "Hangi yapay zeka modelini kullanıyorsun?"
    *   Digital Twin'in bu mesajlara, artık AWS Bedrock (Nova modeli) üzerinden mantıklı ve tutarlı yanıtlar verdiği gözlemlendi. Bu test, **CloudFront -> API Gateway -> Lambda -> Bedrock** zincirinin tamamının başarılı bir şekilde çalıştığını doğruladı.

#### Adım 2: CloudWatch Metriklerini İnceleme

Uygulamanın performansını ve kullanımını sayısal verilerle analiz etmek için CloudWatch metrikleri incelendi:

1.  **CloudWatch Servisine Erişim:**
    *   AWS Management Console arama çubuğuna **"CloudWatch"** yazılarak servise gidildi.

2.  **Lambda Metriklerinin İncelenmesi:**
    *   Sol menüden **Metrikler (Metrics)** -> **Tüm metrikler (All metrics)** seçeneğine tıklandı.
    *   Açılan panelde, AWS servislerinin listelendiği bir bölüm bulunur. Buradan **Lambda** kutucuğuna tıklandı.
    *   Bir sonraki ekranda **By Function Name** seçeneğine tıklandı.
    *   Listelenen fonksiyonlar arasından `twin-api` fonksiyonunun yanındaki kutucuk işaretlendi.
    *   İşaretleme yapıldığında, ekranın üst kısmında bir grafik belirdi. Grafiğin altındaki "Metrikler" listesinden, uygulamanın sağlığını anlamak için şu metrikler seçilerek grafiğe eklendi:
        *   **Invocations (Çağrı Sayısı):** Lambda fonksiyonunun ne kadar sık tetiklendiğini gösterir. `Statistic` olarak `Sum` seçildi.
        *   **Duration (Çalışma Süresi):** Fonksiyonun bir isteği işlemesinin ortalama ne kadar sürdüğünü milisaniye cinsinden gösterir. `Statistic` olarak `Average` seçildi.
        *   **Errors (Hatalar):** Fonksiyonun ne kadar sıklıkla hata verdiğini gösterir.
        *   **Throttles (Kısıtlamalar):** Eş zamanlı istek limitine ulaşıldığı için ne kadar isteğin reddedildiğini gösterir.

3.  **Bedrock Metriklerinin İncelenmesi:**
    *   "Tüm metrikler" ekranına geri dönüldü.
    *   Servis listesinden bu kez **Bedrock** seçildi.
    *   **By Model Id** seçeneğine tıklandı.
    *   Kullanılan modelin (`amazon.nova-lite-v1:0`) yanındaki kutucuk işaretlendi.
    *   Grafiğe şu önemli metrikler eklendi:
        *   **InvocationLatency (Çağrı Gecikmesi):** Bedrock'a gönderilen bir isteğin yanıtının ne kadar sürede geri döndüğünü gösterir.
        *   **InputTokenCount / OutputTokenCount:** Modele gönderilen ve modelden alınan "token" sayısını gösterir. Bu, maliyetleri doğrudan etkilediği için en kritik metriklerden biridir.
    *   Grafik üzerinde fare hareket ettirilerek, belirli zaman aralıklarındaki kullanım detayları görüntülendi.

#### Adım 3: CloudWatch Loglarını İnceleme (Hata Ayıklama için)

Uygulamada bir sorun olduğunda veya çalışma anındaki detayları görmek için Lambda logları incelendi:

1.  CloudWatch ana sayfasında, sol menüden **Log grupları (Log groups)** seçeneğine tıklandı.
2.  Arama çubuğuna `/aws/lambda/twin-api` yazılarak veya listeden bulunarak ilgili log grubuna tıklandı.
3.  Bu grup, `twin-api` Lambda fonksiyonunun ürettiği tüm logları içerir. İçerisinde, zamana göre sıralanmış **Log akışları (Log streams)** listesi bulunur.
4.  En son log akışına tıklanarak detaylar açıldı.
5.  Burada, her bir Lambda çağrısı için `START`, `END` ve `REPORT` satırları görüldü. `REPORT` satırı, o çağrının ne kadar sürdüğü ve ne kadar bellek kullandığı gibi özet bilgiler içerir. Ayrıca, `server.py` kodundaki `print()` ifadelerinin çıktıları da bu ekranda görünür, bu da hata ayıklama için çok değerlidir.

#### Adım 4: CloudWatch Panosu (Dashboard) Oluşturma

Tüm bu önemli metrikleri her seferinde ayrı ayrı aramak yerine, tek bir ekranda toplamak için özel bir pano (dashboard) oluşturuldu:

1.  CloudWatch ana sayfasında, sol menüden **Panolar (Dashboards)** seçeneğine tıklandı.
2.  **Pano oluştur (Create dashboard)** butonuna basıldı.
3.  Panoya **`twin-dashboard`** adı verildi ve "Pano oluştur" butonuna tıklandı.
4.  Boş bir pano ekranı açıldı. İlk "widget"ı (görsel bileşen) eklemek için bir pencere belirdi.
5.  **Widget 1: Lambda Hataları (Sayısal Gösterge):**
    *   Widget türü olarak **Sayı (Number)** seçildi.
    *   Veri kaynağı olarak **Metrikler (Metrics)** seçildi ve "Yapılandır" butonuna tıklandı.
    *   Yukarıdaki Adım 2'de olduğu gibi `Lambda` -> `By Function Name` -> `twin-api` yolu izlendi ve **Errors** metriği seçildi.
    *   İstatistik (Statistic) olarak `Sum`, Periyot (Period) olarak `1 hour` seçildi ve **Widget oluştur (Create widget)** butonuna tıklandı.
6.  **Widget 2: Lambda Çağrıları ve Süresi (Çizgi Grafiği):**
    *   Panonun sağ üst köşesindeki **+** ikonuna tıklanarak yeni bir widget eklendi.
    *   Widget türü olarak **Çizgi (Line)** seçildi.
    *   `Lambda` -> `By Function Name` -> `twin-api` yolu izlendi. Bu kez hem **Invocations** hem de **Duration** metrikleri seçildi.
    *   Grafik yapılandırmasında, Invocations için istatistik `Sum`, Duration için `Average` olarak ayarlandı.
7.  **Widget 3: Bedrock Kullanımı (Çizgi Grafiği):**
    *   Yeni bir çizgi grafiği widget'ı daha eklendi.
    *   Bu kez `Bedrock` -> `By Model Id` yolu izlendi ve kullanılan Nova modelinin **Invocations** metriği seçildi.
8.  Tüm widget'lar eklendikten sonra, sağ üstteki **Panoyu kaydet (Save dashboard)** butonuna tıklanarak pano kalıcı hale getirildi. Artık bu panoya bakarak uygulamanın genel sağlığı ve performansı tek bir ekrandan anlık olarak izlenebilir durumdaydı.

**Hata Ayıklama Notu: CORS ve Önbellekleme Sorunu**

*   **Problem:** Doğrudan yapılan Lambda testlerinin başarılı olması ve backend'in çalıştığını kanıtlamasına rağmen, CloudFront üzerindeki frontend sürekli olarak bir CORS hatası tarafından engelleniyordu.
*   **Kök Neden:** CloudFront, API Gateway'den gelen eski ve hatalı bir CORS politikasını önbelleğe alıyordu.
*   **Çözüm:**
    1.  Hem API Gateway'de hem de Lambda'da `CORS_ORIGINS` ayarı, tam CloudFront alan adı URL'si olarak doğru bir şekilde ayarlandı.
    2.  Yeni ve doğru politikayı çekmeye zorlamak için tam bir CloudFront önbellek geçersizleştirme (`/*`) işlemi oluşturuldu.

### Özet Tablo

| Dosya/Klasör Adı         | Oluşturan      | Amaç                                                                    |
| ------------------------ | -------------- | ----------------------------------------------------------------------- |
| `twin/backend/`          | **SİZ**        | Python backend kodlarını barındırır.                                    |
| `twin/memory/`           | **SİZ**        | Yerel hafıza dosyalarını saklar.                                        |
| `twin/frontend/`         | **OTOMATİK**   | `create-next-app` komutuyla oluşturulan frontend projesi.               |
| `frontend/components/twin.tsx` | **SİZ** | Sohbet arayüzünün ana React bileşeni.                                   |
| `backend/data/*`         | **SİZ**        | AI'nin bilgi kaynakları (JSON, PDF, TXT).                               |
| `backend/*.py`           | **SİZ**        | Backend mantığı, Lambda handler, dağıtım betiği vb.                     |
| `backend/requirements.txt` | **SİZ**        | Python bağımlılıkları.                                                  |
| `frontend/app/*`         | **OTOMATİK**   | Next.js'in temel sayfa ve yapılandırma dosyaları.                       |
| `frontend/node_modules/` | **OTOMATİK**   | İndirilen JavaScript kütüphaneleri.                                     |
| `frontend/out/`          | **OTOMATİK**   | `npm run build` ile oluşturulan, S3'e yüklenen statik dosyalar.         |
| `backend/__pycache__/`   | **OTOMATİK**   | Python'un derlenmiş kod önbelleği.                                      |
| `backend/lambda-package/`| **OTOMATİK**   | `deploy.py` tarafından oluşturulan geçici paketleme klasörü.          |
| `backend/uv.lock`        | **OTOMATİK**   | `uv`'nin kilitlediği Python bağımlılık versiyonları.                    |


<h2 style="text-align:left; color:yellow;">
  4. Gün: Terraform ile Altyapının Kod Olarak Yönetimi (Infrastructure as Code)
</h2>

Bu günün ana teması, bulut altyapısı yönetiminde devrim niteliğinde bir yaklaşım olan **Infrastructure as Code (IaC)** ve bu yaklaşımın en popüler aracı olan **Terraform** oldu. Önceki günlerde AWS Console üzerinden manuel olarak yapılan tüm işlemler (Lambda, S3, API Gateway vb. oluşturma), artık kod dosyaları aracılığıyla otomatik, tekrarlanabilir ve versiyon kontrollü bir şekilde yönetilmeye başlandı.

### Bölüm 1: Manuel Kaynakların Temizlenmesi

Terraform ile otomasyona geçmeden önce, bir önceki gün AWS Console üzerinden manuel olarak oluşturulan tüm kaynaklar dikkatli bir şekilde silindi. Bu işlem, Terraform'un "temiz bir sayfa" üzerinde çalışmasını sağlamak ve yöneteceği kaynakların tam kontrolüne sahip olmasını garantilemek için yapıldı. Silinen servisler:
1.  **Lambda Fonksiyonu:** `twin-api`
2.  **API Gateway:** `twin-api-gateway`
3.  **S3 Bucket'ları:** `twin-memory` ve `twin-frontend` bucket'ları önce boşaltıldı, sonra silindi.
4.  **CloudFront Dağıtımı:** İlgili dağıtım önce devre dışı bırakıldı (disable), ardından silindi.

### Bölüm 2: Terraform Kurulumu ve Proje Yapılandırması

1.  **Terraform Kurulumu:**
    *   İşletim sistemine (macOS) uygun olarak, `brew` paket yöneticisi kullanılarak Terraform'un en güncel versiyonu kuruldu.
    *   Kurulum, `terraform --version` komutu ile doğrulandı.

2.  **Proje Yapısının Hazırlanması:**
    *   Proje ana dizininde (`twin/`) `terraform` adında yeni bir klasör oluşturuldu.
    *   `.gitignore` dosyası, Terraform'un ürettiği durum dosyaları (`.tfstate`), geçici dosyalar (`.terraform/`) ve değişken dosyaları (`.tfvars`) gibi hassas veya gereksiz dosyaları içermeyecek şekilde güncellendi.

3.  **Terraform Yapılandırma Dosyalarının Oluşturulması:**
    `terraform` klasörü içine, altyapıyı tanımlayan `.tf` uzantılı dosyalar oluşturuldu:
    *   `versions.tf`: Terraform ve kullanılacak AWS sağlayıcısının (provider) versiyonlarını belirtir.
    *   `variables.tf`: Altyapıyı parametreleştirmek için kullanılan değişkenleri tanımlar (örn: `project_name`, `environment`, `bedrock_model_id`).
    *   `main.tf`: Altyapının ana mantığını içerir. Tüm AWS kaynakları (S3 bucket'ları, Lambda rolü ve fonksiyonu, API Gateway, CloudFront dağıtımı vb.) burada "resource" blokları olarak tanımlandı.
    *   `outputs.tf`: Dağıtım tamamlandıktan sonra ihtiyaç duyulacak bilgileri (örn: CloudFront URL'si, API Gateway endpoint'i) çıktı olarak verir.
    *   `terraform.tfvars`: `variables.tf` içinde tanımlanan değişkenlere varsayılan değerler atar. Bu dosya, `dev` ortamı için temel yapılandırmayı içerir.

4.  **Frontend'in Dinamik API URL'si için Güncellenmesi:**
    *   `frontend/components/twin.tsx` dosyasındaki API çağrısı, hard-coded bir URL yerine `process.env.NEXT_PUBLIC_API_URL` ortam değişkenini kullanacak şekilde güncellendi. Bu, aynı frontend kodunun hem yerel geliştirmede hem de farklı dağıtım ortamlarında (dev, test, prod) doğru API'ye bağlanmasını sağlar.

### Bölüm 3: Otomatik Dağıtım Betiklerinin (Scripts) Oluşturulması

Tüm dağıtım sürecini tek bir komutla çalıştırılabilir hale getirmek için `scripts` adında bir klasör oluşturuldu ve içine platforma özel betikler eklendi.

1.  **`deploy.sh` (macOS/Linux için):**
    *   Bu betik, `dev`, `test` veya `prod` gibi bir ortam parametresi alarak çalışır.
    *   **Adım 1:** `backend` dizinine gidip `uv run deploy.py` komutuyla Lambda dağıtım paketini (`.zip`) oluşturur.
    *   **Adım 2:** `terraform` dizinine gidip `terraform init` komutunu çalıştırır ve belirtilen ortam için bir Terraform "workspace" oluşturur veya seçer.
    *   **Adım 3:** `terraform apply` komutunu çalıştırarak tüm AWS altyapısını otomatik olarak oluşturur veya günceller.
    *   **Adım 4:** Terraform çıktılarından API URL'sini ve S3 bucket adını alır.
    *   **Adım 5:** `frontend` dizinine gidip, aldığı API URL'sini `.env.production` dosyasına yazar, `npm run build` ile statik dosyaları oluşturur ve `aws s3 sync` ile bu dosyaları ilgili S3 bucket'ına yükler.
    *   **Adım 6:** Son olarak, kullanıcıya erişebileceği CloudFront ve API Gateway URL'lerini gösterir.

2.  **`destroy.sh` (macOS/Linux için):**
    *   Bu betik, `deploy.sh` ile oluşturulan bir ortamın tüm kaynaklarını güvenli bir şekilde yok etmek için kullanılır.
    *   Terraform'un S3 bucket'larını silebilmesi için önce bucket'ların içini `aws s3 rm --recursive` komutuyla boşaltır.
    *   Ardından `terraform destroy` komutunu çalıştırarak ilgili workspace'e ait tüm kaynakları siler.

### Bölüm 4: Farklı Ortamların Dağıtımı ve Test Edilmesi

1.  **Geliştirme (dev) Ortamının Dağıtımı:**
    *   Terminalde, proje ana dizinindeyken `./scripts/deploy.sh dev` komutu çalıştırıldı.
    *   Betik, yukarıda açıklanan tüm adımları başarıyla tamamladı. AWS Console'da kontrol edildiğinde, tüm kaynakların (`twin-dev-api`, `twin-dev-memory-bucket` vb.) Terraform tarafından otomatik olarak oluşturulduğu görüldü.
    *   Betik çıktısındaki CloudFront URL'si tarayıcıda açılarak uygulamanın canlıda sorunsuz çalıştığı test edildi.

2.  **Test (test) Ortamının Dağıtımı:**
    *   Aynı şekilde, `./scripts/deploy.sh test` komutu çalıştırıldı.
    *   Terraform, `test` adında yeni bir workspace oluşturarak, `dev` ortamından tamamen izole, yeni bir kaynak seti (`twin-test-api` vb.) daha oluşturdu.
    *   Bu, farklı ortamların birbirini etkilemeden aynı kod tabanı üzerinden nasıl yönetilebileceğinin güçlü bir örneği oldu.

### Önemli Not:
Eğitmen Bedrock Nova modelini kullanırken, siz bu çalışmada kendi tercihinize göre OpenAI modelini kullanmaya devam ettiniz. Bunun için `requirements.txt`, `main.tf`, `variables.tf` ve `terraform.tfvars` dosyalarında gerekli düzenlemeleri yaparak Terraform yapılandırmasını kendi modelinize başarıyla uyarladınız. Bu, Terraform'un esnekliğini ve yapılandırmanın ne kadar kolay değiştirilebildiğini gösteren harika bir pratik oldu.

### İsteğe Bağlı Adım: Custom Domain
Talimatlarda, `prod` ortamı için AWS Route 53 üzerinden özel bir alan adı (custom domain) ve SSL sertifikası (ACM) yapılandırması da anlatılıyordu. Mevcut bir alan adınız olduğu için bu adımı atladınız, ancak bu, Terraform'un DNS kayıtlarını ve SSL sertifikalarını bile kod ile yönetebildiğini göstermesi açısından önemlidir.