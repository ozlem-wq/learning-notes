# Rust Ekosistem Rehberi - Notlar

**Tarih**: 2026-02-04
**Kaynak**: Ayaz'ın hazırladığı Rust Ekosistem Rehberi
**Amaç**: Uzun vadede Rust ekosistemine aşina olmak, micro app'lerde performans-kritik bileşenlerde kullanmak

---

## 1. Why Rust? - Neden Herkes Geçiyor?

### Performans & Bellek

| Metrik | Rust | Go | Node.js | Java | Python |
|--------|------|-----|---------|------|--------|
| HTTP req/s | **900K+** | 450K | 80K | 350K | 15K |
| Bellek (idle) | **2-5 MB** | 8-15 MB | 30-50 MB | 100-200 MB | 25-40 MB |
| Cold start | **1-5 ms** | 5-15 ms | 50-200 ms | 500-3000 ms | 100-500 ms |
| Container image | **5-15 MB** | 10-25 MB | 150-300 MB | 200-400 MB | 100-250 MB |

**Neden bu kadar hızlı?**
- **GC yok** → Ownership sistemi compile-time'da bellek yönetir
- **Zero-cost abstractions** → Yüksek seviye kod yaz, C/C++ performansı al
- **Discord örneği**: Go'dan Rust'a geçti, GC pause sorunu tamamen çözüldü

### Güvenlik

- Microsoft: CVE'lerin %70'i memory safety hatalarından
- `use-after-free`, `double-free`, `data race` → **İMKANSIZ**
- `NullPointerException` → **YOK** (Option<T> ile zorunlu handle)
- NSA, CISA, White House resmi olarak Rust'ı öneriyor
- Linux Kernel'e Rust desteği eklendi (2022)

### Kim Kullanıyor?

| Şirket | Nereden → Nereye | Sonuç |
|--------|------------------|-------|
| **Discord** | Go → Rust | Tail latency sıfır, 10x kullanıcı |
| **Cloudflare** | C (Nginx) → Rust (Pingora) | Trilyonlarca req/gün, bug sıfır |
| **AWS** | C → Rust | Firecracker: Lambda & Fargate |
| **Figma** | TypeScript → Rust+WASM | Native performans tarayıcıda |
| **Dropbox** | Go → Rust | Milyarlarca ops, RAM dramatik düştü |
| **1Password** | Çoklu dil → Rust | Tek core → 6 platform |
| **Shopify** | Ruby/JS → Rust+WASM | 10x hızlı rendering |

**Ortak tema**: Hepsi önce başka dilde yazdı, scale olunca Rust'a geçti.

### Gelecek & Vizyon

- **Rust Foundation**: Mozilla, AWS, Google, Microsoft, Huawei kurucular
- **Linux Kernel**: 2022'den beri resmi kernel dili → 30+ yıl garanti
- **WASM**: Rust, WASM'a compile eden en iyi dil
- **AI/ML inference**: Python eğitir, Rust serve eder
- **Büyüme**: GitHub'da yıllık %40, Stack Overflow 9 yıl #1 sevilen dil

### Dezavantajlar & Çözümler

| Problem | Çözüm |
|---------|-------|
| Öğrenme eğrisi dik | `rustlings → Rust Book → Exercism → CLI tool` |
| Compile süresi uzun | `cargo-watch`, `sccache`, `mold linker` |
| Frontend immatür | **Rust backend + React frontend** + `ts-rs` |
| ML ekosistemi küçük | Training: Python → Inference: Rust (ONNX) |
| MVP yavaş | **MVP: Python/Node → Scale: Rust** |
| Developer bulmak zor | Kritik servisler Rust, geri kalan Go/Node |

### Ne Zaman Rust?

| Senaryo | Karar |
|---------|-------|
| Yüksek performanslı API | ✅ Rust (Axum + SQLx) |
| CLI araçları | ✅ Rust (tek binary) |
| WebAssembly | ✅ Rust (en iyi WASM dili) |
| K8s operator | ✅ Rust (kube-rs) |
| Hızlı MVP | ⚠️ Hibrit (Python/Node → sonra Rust) |
| ML training | ❌ Python (PyTorch/JAX) |
| Script/otomasyon | ❌ Python |

---

## 2. Web Frameworks & Backend Araçları

### Web Frameworks

| Framework | Stars | Özellik | Ne Zaman? |
|-----------|-------|---------|-----------|
| **Axum** | 20k+ | En Popüler. Tokio ekibi. | API, microservice, real-time |
| **Actix Web** | 23k+ | En Hızlı. Actor model. | Yüksek trafik |
| **Rocket** | 24k+ | Stabil. Rails/Django benzeri. | Hızlı prototip, CRUD |
| **Loco** | 5k+ | Rails'in Rust versiyonu. | Full-stack, admin, hızlı MVP |
| **Poem** | 4k+ | Minimal. OpenAPI otomatik. | API-first projeler |

**Tavsiye**: Yeni başlayanlar için **Axum**

### Backend Araçları

| Araç | Ne İşe Yarar? | Kritik Not |
|------|---------------|------------|
| **Tokio** | Async runtime temeli | Hepsi bunun üzerine kurulu |
| **SQLx** | Async SQL | **Sorgu hatası compile'da yakalanır!** |
| **SeaORM** | Async ORM | Migration, relation, pagination |
| **Diesel** | En olgun ORM | Production-ready |
| **Serde** | JSON/YAML serialize | Ekosistem omurgası |
| **Reqwest** | HTTP client | API çağrıları, webhook |
| **jsonwebtoken + argon2** | JWT + password hash | Auth temeli |

### Supabase + Rust Notu

- Supabase PostgreSQL → Rust tarafında **SQLx** ile bağlan
- Compile-time query checking = runtime hata riski sıfır

### n8n + Rust Notu

- n8n workflow → Rust API endpoint'e webhook
- Rust Axum sunucu → Yüksek performanslı işlemler
- Sonuç → n8n'e geri webhook

---

## 3. Database'ler & AI/ML

### Rust ile Yazılmış Database'ler

| Database | Stars | Özellik | Use Case |
|----------|-------|---------|----------|
| **SurrealDB** | 28k+ | Document + Graph DB | Real-time, multi-model |
| **TiKV** | 15k+ | Distributed KV | Scale |
| **Qdrant** | 22k+ | Vector DB | RAG, AI |
| **Meilisearch** | 48k+ | Full-text search | Arama, autocomplete |
| **Neon** | 15k+ | Serverless PostgreSQL | Serverless PG |

### SurrealDB Detay

```
✅ Document DB (MongoDB gibi)
✅ Graph DB (Neo4j gibi)
✅ Relational (PostgreSQL gibi)
✅ SQL + GraphQL + REST + WebSocket
✅ ACID transactions
✅ Full-text search
✅ Embed edilebilir (SQLite gibi)
✅ TiKV ile 100+ TB scale
```

**Kombinasyon fikri**: Supabase (auth, storage, ana DB) + SurrealDB (graph queries)

### AI/ML Araçları

| Araç | Ne Yapar? | Latency |
|------|-----------|---------|
| **ort (ONNX Runtime)** | ONNX model inference | 0.5-5 ms |
| **candle (HuggingFace)** | LLM inference, embedding | HF production'da kullanıyor |
| **burn** | Rust-native deep learning | Training + inference |
| **rust-bert** | BERT, GPT-2, T5 | NLP tasks |

### Önerilen AI/ML Pipeline

```
1. Training     → Python (PyTorch/JAX)
2. Export       → ONNX / SafeTensors
3. Inference    → Rust (Axum + ort) [0.5-5ms]
4. Vector       → Qdrant (Rust)
5. Search       → Meilisearch (Rust)
6. Orchestrate  → n8n + Webhooks
7. Edge         → Rust → WASM
```

### n8n + Supabase + Rust AI Örneği

```
n8n trigger → Supabase'den veri çek →
Rust API'ye gönder (Axum + ort) →
ONNX model inference →
Qdrant'ta similarity search →
Sonuç n8n'e dön → Supabase'e yaz
```

Latency: Python'da 100ms+ → Rust'ta 5ms

---

## 4. Stack Kombinasyonları

### Stack 1: Pure Rust Full-Stack

```
Frontend  → Leptos (SSR + WASM)
Framework → Axum
ORM       → SeaORM / SQLx
Database  → SurrealDB
Auth      → jwt + argon2
Search    → Meilisearch
Deploy    → Docker 5MB
```

**İdeal**: Performance-critical, tek dev

### Stack 2: Rust + React (En Pratik!) ⭐

```
Frontend  → React / Next.js
API       → Axum + Tonic (REST + gRPC)
ORM       → SQLx / SeaORM
Database  → PostgreSQL (Neon/Supabase)
Search    → Meilisearch
Cache     → Redis
Types     → ts-rs / specta (Rust→TS otomatik!)
```

**İdeal**: Ekip, SaaS, startup - **Benim için en uygun stack**

### Stack 3: Tauri Desktop

```
Shell     → Tauri v2
Frontend  → React/Vue/Svelte
Database  → SQLite (embedded)
```

**Electron 200MB → Tauri 15MB**

### Stack 4: Event-Driven Microservices

```
Gateway   → Axum + Tower
Service   → Tonic (gRPC)
Events    → Kafka / NATS
Database  → SurrealDB + TiKV
K8s       → kube-rs
Container → distroless 5MB
```

**İdeal**: Büyük ölçek, IoT

### Stack 5: AI/ML + Rust

```
API       → Axum
Vector DB → Qdrant
Search    → Meilisearch
ML        → ort (ONNX)
Embedding → candle
Orch      → n8n + webhooks
```

**İdeal**: RAG, real-time AI, n8n AI

### Stack 6: Rust JS Tooling

```
Bundler    → Rspack / Turbopack (Webpack 10x)
Transpiler → SWC (Babel 20x)
Linter     → oxlint (ESLint 100x)
Formatter  → Biome
```

**JS projelerini hızlandır, Rust öğrenmeden bile Rust kullan**

---

## 5. Benim Senaryolarım İçin Öneriler

| Senaryo | Önerilen Stack |
|---------|----------------|
| Teable/NocoDB + Supabase + Metabase | **Stack 2** (Rust+React) + Meilisearch |
| Atomic CRM | **Stack 2** + SurrealDB (graph relations) |
| AI destekli micro app | **Stack 5** (AI/ML) |
| Desktop dashboard | **Stack 3** (Tauri) |
| n8n ağırlıklı otomasyon | **Stack 2** + Axum webhook endpoints |

---

## 6. Öğrenme Yolu

```
1. rustlings (interaktif alıştırmalar)
2. The Rust Book (resmi kitap)
3. Exercism (pratik)
4. İlk proje: CLI tool (clap ile)
5. İkinci proje: Axum API
6. Üçüncü proje: SQLx + PostgreSQL
```

---

## 7. Kurulu Araçlar (Fedora'da muhtemelen var)

Terminal araçları (zaten Rust ile yazılmış):
- `ripgrep (rg)` - grep 10x hızlı
- `fd` - find alternatifi
- `bat` - cat + syntax highlighting
- `eza` - ls + renk + ikon
- `starship` - cross-shell prompt
- `zoxide` - akıllı cd
- `delta` - git diff güzel
- `bottom (btm)` - htop grafikli

---

## Kaynaklar

- [Rust Ekosistem Rehberi HTML](/home/ozlem/Masaüstü/rust-ecosystem-guide.html)
- [Ayaz'ın NotebookLM](https://notebooklm.google.com/notebook/778ec87d-8c5e-4393-9c1f-3feb233db14c)
- [Supabase Postgres Best Practices](https://www.notion.so/Supabase-in-Postgres-En-yi-Uygulamalar-2f96c2d5f40780c8865dc05825acd707)
- [Superpowers Plugin](https://github.com/obra/superpowers)
- [Anthropic Skills](https://github.com/anthropics/skills)
- [Skill Seekers](https://github.com/yusufkaraaslan/Skill_Seekers)

---

*Bu notlar Ayaz ile Discord'da yapılan konuşma ve Rust Ekosistem Rehberi baz alınarak oluşturulmuştur.*
