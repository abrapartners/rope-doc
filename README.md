<!-- This file is part of Abra ROPE. -->

<!-- Copyright (c) 2020, Oguzhan Karahan. -->
<!-- All rights reserved (see LICENSE). -->

Bu dosya, `abra_rope` API'sini açıklar.

İçindekiler:
- [Input format](#input)
- [Output format](#output)

**Notlar**:

- tüm koordinat dizileri için beklenen sıra `[lon, lat]` şeklindedir.
- tüm zamanlamalar saniye cinsindendir.
- tüm mesafeler metre cinsindendir.
- bir `time_window` nesnesi, `[start, end]` şeklinde ifade edilir.
- kullanımdan kaldırılan anahtarların üstü çizilidir.
- çıktıdaki `cost` değerleri, optimizasyon hedefinde kullanılan değerlerdir. (şu anda `duration`a eşittir)
- bir `task` ya bir `job`, bir `delivery` ya da bir `pickup` olarak eklenebilir.

# Input

Hata açıklamaları, standart girdiden veya bir dosyadan okunur (`-i` kullanılarak) ve şablondaki gibi geçerli bir `json` olmalıdır.

| Anahtar   | Açıklama  |
|-----------|-----------|
| [`jobs`](#jobs) | ziyaret edilecek yerleri açıklayan bir dizi `job` nesnesi |
| [`shipments`](#shipments) | toplama ve teslimat görevlerini açıklayan `shipment` nesneleri dizisi |
| [`vehicles`](#vehicles) | mevcut araçları açıklayan bir dizi `vehicle` nesnesi |
| [[`matrix`](#matrix)] | özel bir matrisi tanımlayan isteğe bağlı iki boyutlu bir dizi |

## Jobs

Bir `job` nesnesi aşağıdaki özelliklere sahiptir:

| Anahtar     | Açıklama    |
| ----------- | ----------- |
| `id` | unique olarak kullanılan bir tam sayı |
| [`description`] | bu işi açıklayan bir metin |
| [`location`] | koordinat dizisi |
| [`location_index`] | özel matriste ilgili satır ve sütunun dizini |
| [`service`] | job hizmet süresi (varsayılanı 0) |
| ~~[`amount`]~~ | ~~çok boyutlu nicelikleri tanımlayan bir tamsayı dizisi~~ |
| [`delivery`] | teslimat için çok boyutlu miktarları tanımlayan bir tamsayı dizisi |
| [`pickup`] | teslim alma için çok boyutlu miktarları açıklayan bir tamsayı dizisi |
| [`skills`] | zorunlu skill'leri tanımlayan bir dizi tamsayı |
| [`priority`] | öncelik düzeyini açıklayan `[0, 100]` aralığında bir tam sayı (varsayılanı 0'dır) |
| [`time_windows`] | job hizmetinin başlatılması için geçerli parametreleri tanımlayan bir dizi `time_window` nesnesi |

## Shipments

Bir `shipment` nesnesi aşağıdaki özelliklere sahiptir:

| Anahtar     | Açıklama    |
| ----------- | ----------- |
| `pickup` | teslim almayı açıklayan bir `shipment_step` nesnesi |
| `delivery` | teslimatı açıklayan bir `shipment_step` nesnesi |
| [`amount`] | çok boyutlu nicelikleri tanımlayan bir tamsayı dizisi |
| [`skills`] | zorunlu skill'leri tanımlayan bir dizi tamsayı |
| [`priority`] | öncelik düzeyini açıklayan `[0, 100]` aralığında bir tam sayı (varsayılanı 0'dır) |

Bir `shipment_step`, bir `job` nesnesine benzer (`shipment`ta zaten mevcut olan, paylaşılan anahtarlardır):

| Anahtar     | Açıklama    |
| ----------- | ----------- |
| `id` | unique olarak kullanılan bir tam sayı |
| [`description`] | bu step'i açıklayan metin |
| [`location`] | koordinat dizisi |
| [`location_index`] | özel matriste ilgili satır ve sütunun dizini |
| [`service`] | job hizmet süresi (varsayılanı 0'dır) |
| [`time_windows`] | job hizmetinin başlatılması için geçerli parametreleri açıklayan bir dizi `time_window` nesnesi |

## Vehicles

Bir `vehicle` nesnesi aşağıdaki özelliklere sahiptir:

| Anahtar     | Açıklama    |
| ----------- | ----------- |
| `id` | unique olarak kullanılan bir tam sayı |
| [`profile`] | yönlendirme profili (varsayılanı `car`dır) |
| [`description`] | bu aracı tanımlayan bir metin |
| [`start`] | koordinat dizisi |
| [`start_index`] | özel matriste ilgili satır ve sütunun dizini |
| [`end`] | koordinat dizisi |
| [`end_index`] | özel matriste ilgili satır ve sütunun dizini |
| [`capacity`] | çok boyutlu nicelikleri tanımlayan bir tamsayı dizisi |
| [`skills`] | skill'leri tanımlayan bir dizi tamsayı |
| [`time_window`] | çalışma saatlerini tanımlayan bir `time_window` nesnesi |
| [`breaks`] | bir dizi `break` nesnesi |

Bir `break` nesnesi aşağıdaki özelliklere sahiptir:

| Anahtar     | Açıklama    |
| ----------- | ----------- |
| `id` | unique olarak kullanılan bir tam sayı |
| `time_windows` | mola başlangıcı için geçerli parametreleri tanımlayan bir dizi `time_window` nesnesi |
| [`service`] | mola süresi (varsayılanı 0'dır) |
| [`description`] | bu molayı açıklayan bir metin |

## Notlar

### Task lokasyonları

Özel bir matris sağlanmışsa `job`, `pickup` ve `delivery` nesneleri için:

- `location_index` zorunludur.
- `location` isteğe bağlıdır ancak response'ta koordinatları alacak şekilde ayarlanabilir.

Özel matris sağlanmazsa:

- route engine'e bir `table` sorgusu gönderilmesi gerekir.
- `location` zorunludur.
- `location_index` zorunlu değildir.

### `vehicle` lokasyonları

- `start` ve `end` anahtarlarından en az biri olmalıdır. Her ikisini de eklemek bir `vehicle` için isteğe bağlıdır.
- `end` atlanırsa, response'ta ortaya çıkan rota, seçimi optimizasyon süreci tarafından belirlenen son ziyaret edilen görevde duracaktır.
- `start` atlanırsa, response'taki rota, seçimi optimizasyon süreci tarafından belirlenen ilk ziyaret edilen görevde başlar.
- gidiş-dönüş talep etmek için, aynı koordinatlarla hem `start` hem de `end`i belirtmeniz yeterlidir.
- özel bir matris sağlanıp sağlanmadığına bağlı olarak, gerekli alanlar `job` anahtarı `location` ve `location_index` ile aynı mantığı izler.

### Kapasite kısıtlamaları

Kapasite kısıtlamalarıyla ilgili bir sorunu tanımlamak için (araçlar için `capacity`, işler için `delivery` ve `pickup`, gönderiler için `amount`) kullanılır. Bu diziler, aynı anda birkaç ölçümün özel kısıtlamalarını modellemek için kullanılabilir (ör. öğe sayısı, ağırlık, hacim vb.). Bir aracın yalnızca her rota adımında ortaya çıkan yükün her bir metrik için `capacity`deki eşleşen değerden düşük olması durumunda, bir dizi görevi yerine getirmesine izin verilecektir. Miktarlar için birden fazla bileşen kullanırken, en önemli/sınırlayıcı metriklerin ilk sıraya konulması önerilir. `priority` değerinin nasıl kullanıldığını inceleyebilirsiniz.

Job'ların teslimatıyla ilgili tüm adetlerin araç başlangıcında yüklendiği, tüm teslim almayla ilgili adetlerin araç sonunda geri getirildiği varsayılır.

### Skills

Tüm görevlerin tüm araçlar tarafından gerçekleştirilemediği bir sorunu tanımlamak için `skills`i kullanabilirsiniz. Job skill'leri zorunludur, yani bir işe yalnızca **tüm** gerekli skill'lere sahip bir araç hizmet verebilir. Başka bir deyişle: `j` işi, `v` aracına uygunsa, `j.skills`, `v.skills` içinde yer almalıdır. Yani vehicle skills, job skills'i kapsamalıdır.

Herhangi bir skill gerektirmeyen modelleme problemlerini kolaylaştırmak için, herhangi bir `skills` anahtarı sağlanmadığında hiçbir kısıtlama olmadığı varsayılır.

### Görev öncelikleri

Görevleri önceliklendirmek ve sıralamak için kullanışlıdır. Bazı görevler için yüksek bir `priority` değeri belirlemek, onları daha düşük öncelikli görevleri yapmak yerine, çözüme mümkün olduğunca dahil etme eğiliminde olacaktır.

### Zaman pencereleri

Zaman pencerelerinin nasıl tanımlanacağına karar vermek kullanıcılara kalmıştır:

- **relative values**, ör. Planlama başlangıcındaki 4 saatlik bir zaman penceresi için `[0, 14400]`. Bu durumda, response'ta `arrival` anahtarıyla gösterilen tüm zamanlar, planlama başlangıcına göredir;
- **absolute values**, "real" zaman parametreleridir. Bu durumda, çıktıda `arrival` alanıyla belirtilen tüm zamanlar, zaman aralıkları olarak yorumlanabilir.

Request'te bir zaman penceresinin olmaması, hiçbir zamanlama kısıtlamasının uygulanmadığı anlamına gelir. Özellikle, `time_window` anahtarı olmayan bir araç, herhangi bir sayıda göreve hizmet edebilecektir ve `time_windows` anahtarı olmayan bir görev, skill'ler gibi diğer kısıtlamaların izin verdiği ölçüde (kapasiteye ve diğer araçlar/görev zaman pencerelerine de bakarak), herhangi bir zamanda herhangi bir rotaya dahil edilebilir.

## Matrix

Bir `matrix` nesnesi, route engine tarafından hesaplanan seyahat süresi matrisine alternatif olarak özel bir seyahat süresi matrisinin satırlarını tanımlayan bir işaretsiz tamsayı dizisi dizisidir. Bu nedenle, özel bir matris sağlanırsa, `location`, `start` ve `end` özellikleri isteğe bağlı hale gelir. Optimizasyon sırasında koordinatlar yerine `*_index` parametreleri ile sağlanan satır ve sütun gösterimleri kullanılır.

# Output

Hesaplanan çözüm, standart çıktıda `json` olarak veya aşağıdaki gibi biçimlendirilmiş bir dosyada (`-o` kullanılarak) yazılır.

| Anahtar     | Açıklama    |
| ----------- | ----------- |
| `code` | durum kodu |
| `error` | hata mesajı (mevcut `code`, `0`dan farklıysa) |
| [`summary`](#özet) | response'u özetleyen göstergelerin nesnesi |
| `unassigned` | `id` ve `location` (varsa) ile atanmamış görevleri açıklayan nesne dizisi |
| [`routes`](#routes) | `route` nesneleri dizisi |

## Code

Durum kodu için olası değerler şunlardır:

| Değer         | Durum       |
| ------------- | ----------- |
| `0` | no error raised       |
| `1` | internal error        |
| `2` | input error           |
| `3` | routing error         |

## Summary

`summary` nesnesi aşağıdaki özelliklere sahiptir:

| Anahtar     | Açıklama    |
| ----------- | ----------- |
| `cost` | tüm güzergahlar için toplam maliyet |
| `unassigned` | yerine getirilemeyen görev sayısı ve nesneleri |
| `service` | tüm güzergahlar için toplam hizmet süresi |
| `duration` | tüm güzergahlar için toplam seyahat süresi |
| `waiting_time` | tüm güzergahlar için toplam bekleme süresi |
| `priority` | atanan tüm görevler için öncelik toplamı|
| ~~[`amount`]~~ | ~~tüm rotalar için toplam adet~~ |
| [`delivery`] | tüm güzergahlar için toplam teslimat adedi |
| [`pickup`] | tüm güzergahlar için toplam alım adedi |
| [`distance`]* | tüm güzergahlar için toplam mesafe |

*: `-g` flag'i kullanılırken sağlanır.

## Routes

Bir `route` nesnesi aşağıdaki özelliklere sahiptir:

| Anahtar     | Açıklama    |
| ----------- | ----------- |
| `vehicle` | bu rotaya atanan aracın id'si |
| [`steps`](#steps) | `step` nesneleri dizisi |
| `cost` | bu rota için maliyet |
| `service` | bu rota için toplam hizmet süresi |
| `duration` | bu rota için toplam seyahat süresi |
| `waiting_time` | bu rota için toplam bekleme süresi |
| `priority` | bu rotadaki görevler için öncelik toplamı |
| ~~[`amount`]~~ | ~~bu rotadaki görevler için toplam adet~~ |
| [`delivery`] | bu rotadaki görevler için toplam teslimat adedi |
| [`pickup`] | bu rotadaki görevler için toplam alım adedi |
| [`description`] | araç açıklaması (girişte belirtilmişse) |
| [`geometry`]* | rota geometrisi |
| [`distance`]* | toplam rota mesafesi |

*: `-g` flag'i kullanılırken sağlanır.

### Steps

Bir `step` nesnesi aşağıdaki özelliklere sahiptir:

| Anahtar     | Açıklama    |
| ----------- | ----------- |
| `type` | bir metin verisi (`start`, `job`, `pickup`, `delivery`, `break` veya `end` olarak girilebilir) |
| `arrival` | mevcut step için tahmini varış zamanı |
| `duration` | mevcut step için varışta kümülatif seyahat süresi |
| [`description`] | mevcut step açıklaması (girişte belirtilmişse) |
| [`location`] | mevcut step için koordinatlar dizisi (girişte belirtilmişse) |
| [`id`] | Mevcut step'te gerçekleştirilen görevin id'si (yalnızca `type` değeri `job`, `pickup`, `delivery` veya `break` ise sağlanır) |
| ~~[`job`]~~ | ~~mevcut step için gerçekleştirilen işin id'si (yalnızca `type` değeri `job` ise sağlanır)~~ |
| [`load`] | bu step tamamlandıktan sonraki araç yükü (kapasite kısıtlamaları ile birlikte sağlanır) |
| [`service`] | bu step'teki hizmet süresi (`start` ve `end` için servis süresi yoktur) |
| [`waiting_time`] | bu step'e varış için bekleme süresi (`start` ve `end` için bekleme süresi yoktur) |
| [`distance`]* | bu step'e varış için kat edilen mesafe |

*: `-g` flag'i kullanılırken sağlanır.
