# HASS-S3
Bu özel entegrasyon, S3 ile etkileşim kurmak, dosya yüklemek veya dosyaları bucket'lar içinde ve arasında kopyalamak için bir servis sağlar.

S3 bucket'ınızı AWS konsolu üzerinden oluşturun, bucket isimlerinin benzersiz olması gerektiğini unutmayın. Ben varsayılan erişim ayarlarıyla (tüm public erişimler KAPALI) bir bucket oluşturdum ve bucket ismini `my-bucket-random_number` formatında, `random_number` kısmını [bu web sitesinde](https://onlinehashtools.com/generate-random-md5-hash) oluşturarak belirledim.

**Not:** Yerel ve kendi sunucunuzda barındırılan (self-hosted) bir alternatif için resmi [Minio entegrasyonuna](https://www.home-assistant.io/integrations/minio/) göz atın.

## Kurulum ve yapılandırma
`custom_components` klasörünü yapılandırma dizinize yerleştirin (veya içeriğini mevcut `custom_components` klasörüne ekleyin). Home Assistant yapılandırma arayüzüne veya `configuration.yaml` dosyanıza ekleyin:
```yaml
s3:
  aws_access_key_id: AWS_ACCESS_KEY
  aws_secret_access_key: AWS_SECRET_KEY
  region_name: eu-west-1 # isteğe bağlı bölge, varsayılan us-east-1
```

## Servisler
### Put Servisi
S3 varlığı (entity), S3'e dosya yüklemek için bir put servisi sunar.

Servis çağrısı için örnek veri:
```
{
  "bucket": "my_bucket",
  "key": "my_key/file.jpg",
  "file_path": "/some/path/file.jpg",
  "storage_class": "STANDARD_IA" # isteğe bağlı
  "content_type" : "image/jpeg" # isteğe bağlı
  "tags":  "tag1=aTagValue&tag2=anotherTagValue" # isteğe bağlı
}
```

### Copy Servisi
S3 varlığı (entity), S3 içindeki dosyaları taşımak için bir copy servisi sunar.

Servis çağrısı için örnek veri:

```
{
  "bucket": "my_bucket",
  "key_source": "my_key/file_source.jpg",
  "key_destination": "my_key/file_destination.jpg"
}
```

Öğeleri bucket'lar arasında taşımanız gerekirse bu sözdizimini kullanın:
```
{
  "bucket_source": "my_source_bucket",
  "key_source": "my_key/file_source.jpg",
  "bucket_destintation": "my_destination_bucket",
  "key_destination": "my_key/file_destination.jpg"
}
```

### Delete Servisi
S3 varlığı (entity), S3'ten dosya (nesne) silmek için bir delete servisi sunar.

Servis çağrısı için örnek veri:
```
{
  "bucket": "my_bucket",
  "key": "my_key/file_source.jpg",
}
```

### Sign URL Servisi
S3 varlığı (entity), halihazırda S3'te depolanan içeriğe bir URL ile erişmek için, tanımlı bir geçerlilik süresine sahip önceden imzalanmış (pre-signed) URL'ler oluşturmak adına bir signurl servisi sunar. Bu eylemi S3 copy servisini çağırdıktan sonra çalıştırın. Bu servis, sonraki bir otomasyonda tetikleyici olarak kullanabileceğiniz s3_signed_url türünde bir olay (event) oluşturur. Olay verisi, URL ve önceden imzalanmış URL'den oluşan bir anahtar-değer çifti döndürür.

Servis çağrısı için örnek veri:
```
{
  "bucket": "my_bucket",
  "key": "my_key/file_source.jpg",
  "duration": 300
}
```


## Örnek otomasyon
Aşağıdaki otomasyon, yerel dosya sisteminde oluşturulan dosyaları otomatik olarak S3'e yüklemek için [folder_watcher](https://www.home-assistant.io/integrations/folder_watcher/) kullanır:

```yaml
- id: '1587784389530'
  alias: upload-file-to-S3
  description: 'When a new file is created, upload to S3'
  trigger:
    event_type: folder_watcher
    platform: event
    event_data:
      event_type: created
  action:
    service: s3.put
    data_template:
      bucket: "my_bucket"
      key: "input/{{ now().year }}/{{ (now().month | string).zfill(2) }}/{{ (now().day | string).zfill(2) }}/{{ trigger.event.data.file }}"
      file_path: "{{ trigger.event.data.path }}"
      storage_class: "STANDARD_IA"
```
Not: `folder_watcher`'ı yapılandırmanız gerekir.

## S3'e Erişim
S3 bucket'ınıza bağlanmak için [Filezilla](https://filezilla-project.org/) öneririm, ücretsiz sürümü mevcuttur.
