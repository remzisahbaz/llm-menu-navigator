# LLM Menu Navigator

Kurumsal web uygulamaları için lokal LLM tabanlı akıllı menü yönlendirme asistanı.

## Problem

100+ menü öğesi, 4 seviye derinlik, kullanıcılar sürekli "X nerede?" diye soruyor. Çözüm: Kullanıcının yazdığı kelimeyi tüm menü ağacında arayıp, eşleşen menüleri JSON formatında döndüren bir AI asistanı.

## Mimari

```
Kullanıcı → Open WebUI → Ollama → Mistral Small 24B → JSON Response
                                        ↑
                                  System Prompt + Menü JSON
```

- **Ollama** — AI modellerini kendi donanımınızda çalıştıran açık kaynak araç
- **Open WebUI** — ChatGPT benzeri sohbet arayüzü
- **Mistral Small 24B** — Talimat takibinde güçlü, açık kaynak AI modeli

## Neden Lokal LLM?

- Şirket verileri dışarı çıkmaz
- API maliyeti yok
- İnternet bağımlılığı yok
- Anlık yanıt süresi

## Model Karşılaştırması

"Rapor" araması — menü ağacında 12 farklı yerde eşleşme var:

| Model | Boyut | Bulunan | Doğruluk | Kural Uyumu | Format |
|-------|-------|---------|----------|-------------|--------|
| Gemma 3 4B | 4B | 1/12 | %8 | Çok kötü | Bozuk |
| Llama 3.1 8B | 8B | 5/12 | %42 | Kötü | Bozuk |
| Gemma 3 12B | 12B | 6/12 | %50 | Orta | İyi |
| **Mistral Small 24B** | **24B** | **12/12** | **%100** | **Çok iyi** | **Mükemmel** |

### Neden Küçük Modeller Başarısız Oldu?

- **Gemma 3 4B**: Kuralları tamamen görmezden geldi. Menü yönlendirmesi yerine React kodu yazdı.
- **Llama 3.1 8B**: Prompt'taki ipucu metnini cevaba kopyaladı — kuralı anlamadı, ezberledi.
- **Gemma 3 12B**: Ana kategorideki menüleri buldu ama farklı bölümlerin alt menülerindeki eşleşmeleri kaçırdı.
- **Mistral Small 24B**: Recursive arama, kural takibi ve JSON çıktı formatında eksiksiz performans.

## Öğrenilen Dersler

### 1. RAG vs Prompt Embedding

Menü verisini RAG (bilgi bankası) ile yüklediğimizde sistem veriyi parçalara ayırırken üst menü-alt menü ilişkisini kaybetti. 12 eşleşmeden sadece 6'sını bulabildi. Veriyi doğrudan prompt'a gömdüğümüzde sonuç 6'dan 12'ye çıktı.

**Kural:** Hiyerarşik veri + tam tarama gerekiyorsa → RAG yerine prompt embedding kullan.

### 2. Hallucination Kontrolü

Kullanıcı "üye" yazdığında model "döviz kurları" sayfasını da döndürdü — çünkü anlamsal çağrışım yaptı. Çözüm: "Sadece label'da harfi harfine geçen kelimeleri eşleştir, tahmin yapma" kuralı.

### 3. Model Seçimi

Daha güçlü model her zaman daha iyi sonuç vermez. Göreve uygun model seçimi çok daha kritik. Bu görev için instruction-following kapasitesi, reasoning'den daha önemli.

## Örnek Menü JSON

Gerçek proje verisi yerine benzer yapıda örnek menü kullanılmıştır:

→ [sample-menu.json](./sample-menu.json)

## System Prompt Örneği

→ [system-prompt.md](./system-prompt.md)

## Örnek Çıktı

Kullanıcı girdisi: `rapor`

```json
{
  "Response1": {"path_description": "Sol menü → Raporlar → Satış Raporu", "url": "/reports/sales"},
  "Response2": {"path_description": "Sol menü → Raporlar → Satış Detay Raporu", "url": "/reports/sales/detail"},
  "Response3": {"path_description": "Sol menü → Raporlar → Performans Raporu", "url": "/reports/performance"},
  "Response4": {"path_description": "Sol menü → Raporlar → Genel Kurul Raporu", "url": "/reports/general-assembly"},
  "Response5": {"path_description": "Sol menü → Raporlar → İşlem Log Raporu", "url": "/reports/transaction-logs"},
  "Response6": {"path_description": "Sol menü → Raporlar → Kullanıcı Aktivite Raporu", "url": "/reports/user-activity"},
  "Response7": {"path_description": "Sol menü → Stok Yönetimi → Kritik Stok Raporu", "url": "/inventory/critical-stock-report"},
  "Response8": {"path_description": "Sol menü → Stok Yönetimi → Stok Raporları → Giriş Raporu", "url": "/inventory/reports/stock-in-report"},
  "Response9": {"path_description": "Sol menü → Stok Yönetimi → Stok Raporları → Çıkış Raporu", "url": "/inventory/reports/stock-out-report"},
  "Response10": {"path_description": "Sol menü → Stok Yönetimi → Stok Raporları → Transfer Raporu", "url": "/inventory/reports/transfer-report"},
  "Response11": {"path_description": "Sol menü → Sipariş İşlemleri → Garanti İşlemleri → Garanti Raporu", "url": "/orders/warranty/report"},
  "Response12": {"path_description": "Sol menü → Doküman Yönetimi → Doküman Raporu", "url": "/documents/report"}
}
```

## Kurulum

```bash
# 1. Ollama kur
curl -fsSL https://ollama.ai/install.sh | sh

# 2. Modeli indir
ollama pull mistral-small

# 3. Open WebUI kur (Docker)
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway -v open-webui:/app/backend/data --name open-webui --restart always ghcr.io/open-webui/open-webui:main

# 4. Open WebUI'da system prompt'u yapıştır ve test et
```

## Teknolojiler

- [Ollama](https://ollama.ai/) — Lokal LLM çalıştırma
- [Open WebUI](https://github.com/open-webui/open-webui) — Chat arayüzü
- [Mistral Small 24B](https://mistral.ai/) — AI modeli

