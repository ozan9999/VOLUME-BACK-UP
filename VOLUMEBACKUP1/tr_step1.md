# Docker Volume Backup
## 



Docker volume, containerlarınızda sakladığınız verileri kalıcı hale getirmek için kullanabileceğiniz bir yöntemdir. Docker volume ile verileriniz host sistemi üzerinde fiziksel olarak ayrı bir yerde depolanır ve containerlar tarafından erişilebilir. Docker volume kullanmanın faydaları arasında verileri yedeklemek ve geri yüklemek, verileri daha iyi yönetmek, containerları daha güvenli ve stabil hale getirmek sayılabilir.


## Bu eğitimde yapacaklarımız ise kısa şöyle özetlenebilir;

- Bir docker-compose.yml dosyası ile 2 adet nginx,1 adet jareware ve 1 adet minio container oluşturacağız.
- Bu 2 adet nginx serverlarımızın geliştire yapılacak olan dosyalarını BBK adında bir volume'a bağlayacağız
- Ardından jareware/docker-volume-backup containerımız ile bu volume'ü önce bu containerın üstüne
- Sonrasında eşlenik bir dizinle tekrar bu containerımızdan local makinemizin üstüne alacağız.
- Son olarak ise minio containerımıza bağlanıp bir bucket oluşturduktan sonra, volume back upımızı bazı komutlarla bu bucket'ın içine atacağız.

## Odaklanmamız gereken nokta ise;
Eğitim boyunca birçok farklı komut kullanacağız, fakat bu eğitim özelinde ana nokta bu yedekleme işleminin neden yapıldığını anlamak, ve bu doğrultuda dockor-compose.yml dosyasının inceleyerek oluşturduğumuz yapının anlaşılması olacaktır.Şimdi eğitime başlayabiliriz.


