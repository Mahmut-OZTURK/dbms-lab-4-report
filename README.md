# Deney Sonu Teslimatı

Sistem Programlama ve Veri Yapıları bakış açısıyla veri tabanlarındaki performansı öne çıkaran hususlar nelerdir?

Aşağıda kutucuk (checkbox) ile gösterilen maddelerden en az birini seçtiğiniz açık kaynak kodlu bir VT kaynak kodları üzerinde göstererek açıklayınız. Açıklama bölümüne kısaca metninizi yazıp, kod üzerinde gösterim videonuzun linkini en altta belirtilen kutucuğa yerleştiriniz.

- [X]  Seçtiğiniz konu/konuları bu şekilde işaretleyiniz. **!**
    
---

# 1. Sistem Perspektifi (Operating System, Disk, Input/Output)

### Disk Erişimi

- [ ]  **Blok bazlı disk erişimi** → block_id + offset
- [ ]  Rastgele erişim

### VT için Page (Sayfa) Anlamı

- [ ]  VT hangisini kullanır? **Satır/ Sayfa** okuması

---

### Buffer Pool

- [X]  Veritabanları, Sık kullanılan sayfaları bellekte (RAM) kopyalar mı (caching) ?

- [X]  LRU / CLOCK gibi algoritmaları
- [X]  Diske yapılan I/O nasıl minimize ederler?

# 2. Veri Yapıları Perspektifi

- [X]  B+ Tree Veri Yapıları VT' lerde nasıl kullanılır?
- [ ]  VT' lerde hangi veri yapıları hangi amaçlarla kullanılır?
- [ ]  Clustered vs Non-Clustered Index Kavramı
- [ ]  InnoDB satırı diskte nasıl durur?
- [ ]  LSM-tree (LevelDB, RocksDB) farkı
- [ ]  PostgreSQL heap + index ayrımı

DB diske yazarken:

- [ ]  WAL (Write Ahead Log) İlkesi
- [ ]  Log disk (fsync vs write) sistem çağrıları farkı

---

# Özet Tablo

| Kavram      | Bellek          | Disk / DB      |
| ----------- | --------------- | -------------- |
| Adresleme   | Pointer         | Page + Offset  |
| Hız         | O(1)            | Page IO        |
| PK          | Yok             | Index anahtarı |
| Veri yapısı | Array / Pointer | B+Tree         |
| Cache       | CPU cache       | Buffer Pool    |

---

# Video [Linki](https://www.youtube.com/watch?v=Nw1OvCtKPII&t=2635s) 
Ekran kaydı. 2-3 dk. açık kaynak V.T. kodu üzerinde konunun gösterimi. Video kendini tanıtma ile başlamalıdır (Numara, İsim, Soyisim, Teknik İlgi Alanları). 

---

# Açıklama (Ort. 600 kelime)

   Veritabanı Performansında Buffer Pool ve B+ Tree Mekanizmaları: SQLite Kaynak Kod İncelemesi
Modern veritabanı sistemlerinin performansı, altta yatan sistem programlama ve veri yapıları implementasyonlarına bağlıdır. Bu çalışmada, açık kaynak kodlu SQLite veritabanı yönetim sisteminin kaynak kodları üzerinden iki kritik performans bileşenini inceledim: Buffer Pool (bellek önbellekleme) mekanizması ve B+ Tree veri yapısı. Bu mekanizmaların veritabanı performansına doğrudan etkisini SQLite'ın C dilinde yazılmış kaynak kodları üzerinde somut örneklerle gösterdim.
  Buffer Pool: Disk I/O Optimizasyonu
Veritabanı sistemlerinde en büyük performans darboğazlarından biri disk erişim hızıdır.  Bu sorunu çözmek için SQLite, Buffer Pool adı verilen bir önbellekleme mekanizması kullanır.
SQLite'ın getPageNormal fonksiyonunda bu mekanizmayı inceledim. Fonksiyon, bir veritabanı sayfasını okumadan önce sqlite3PcacheFetch fonksiyonuyla Page Cache'e (pPCache) bakar. Eğer istenen sayfa cache'de mevcutsa - bu duruma "cache hit" denir - disk erişimi yapılmadan doğrudan RAM'den veri döndürülür. Bu durumda PAGER_STAT_HIT sayacı artırılır ve fonksiyon hızlıca sonlanır. Disk I/O tamamen atlanmış olur, bu da milisaniyeler kazandırır.
Cache'de sayfa bulunamadığında ise "cache miss" durumu oluşur. Bu durumda sqlite3PcacheFetchStress fonksiyonu devreye girer. Bu fonksiyon, LRU (Least Recently Used - En Az Yakın Zamanda Kullanılan) algoritmasını uygular. Cache dolu olduğunda, en az kullanılan sayfayı bellekten çıkararak yeni sayfa için yer açar. Ardından readDbPage fonksiyonuyla sayfa diskten okunur ve cache'e eklenir. PAGER_STAT_MISS sayacı artırılır. Önemli olan nokta şudur: Diskten okunan her sayfa otomatik olarak cache'e konur, böylece aynı sayfaya yapılacak sonraki erişimler cache hit olarak gerçekleşir.
Buffer Pool mekanizması sayesinde SQLite, sık erişilen sayfaları RAM'de tutarak disk I/O işlemlerini drastik şekilde azaltır. 
  B+ Tree: Hızlı Veri Erişimi
SQLite'da her tablo ve index, B+ Tree veri yapısı kullanılarak organize edilir. B+ Tree, dengeli bir ağaç yapısıdır ve verilere logaritmik zamanda erişim sağlar.
moveToRoot fonksiyonu, B+ Tree üzerinde arama işleminin başlangıç noktasıdır. Her tablonun ve index'in kendine ait bir kök sayfası vardır ve pgnoRoot değişkeni bu kök sayfanın disk üzerindeki numarasını tutar. getAndInitPage fonksiyonuyla bu kök sayfa belleğe yüklenir. Kök sayfa numarası tablo oluşturulduğunda belirlenir ve sabit kalır.
Arama işlemi sırasında moveToChild fonksiyonu kullanılarak ağaçta aşağı doğru ilerlenir. Bu fonksiyonda iPage değişkeni ağacın kaçıncı seviyesinde olunduğunu gösterir ve her child'a geçişte bir artar. newPgno parametresi, hangi alt düğüme gidileceğini belirtir. Bu sayfa numarası, arama algoritmasının anahtar karşılaştırması sonucunda belirlenir.
B+ Tree'nin gücü, arama karmaşıklığının O(log n) olmasıdır
SQLite kaynak kodlarında incelediğim Buffer Pool ve B+ Tree mekanizmaları, veritabanı performansının temelini oluşturur. Buffer Pool, disk erişimini minimize ederek latency'yi azaltırken, B+ Tree logaritmik arama karmaşıklığı sağlayarak büyük veri setlerinde bile hızlı erişim garantiler. Bu iki mekanizma birlikte çalışarak modern veritabanı sistemlerinin yüksek performansını mümkün kılar.

## VT Üzerinde Gösterilen Kaynak Kodları

Buffer Pool - getPageNormal fonksiyonu [Github](https://github.com/sqlite/sqlite/blob/master/src/pager.c#L5535) \
B+ Tree Root - moveToRoot fonksiyonu [Github](https://github.com/sqlite/sqlite/blob/master/src/btree.c#L5542) \
B+ Tree Child - moveToChild fonksiyonu [Github](https://github.com/sqlite/sqlite/blob/master/src/btree.c#L4567) \
... \
...
