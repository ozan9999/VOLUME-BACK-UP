## BAŞLAYALIM;
Eğitime başlayabiliriz öncelik docker-compose.yml dosyamızın oluşturulmasında;




## Docker-compose dosyamızın oluşturulması

Docker-compose.yml dosyası, birden fazla Docker container’ını tek bir dosyada tanımlayarak yönetmeyi sağlayan bir orkestrasyon aracıdır. Bu dosya, container’ların nasıl oluşturulacağını, nasıl bağlanacağını, nasıl çalıştırılacağını ve nasıl ölçekleneceğini belirtir.

Önce bulunduğumuz dizinin altında touch komutu ile dosyamızı oluşturuyoruz

```sh
touch docker-compose.yml
```

Sonrasında editör veya vim komutu aracılığı ile dosyanın içine giriyoruz.(Editörden yapılması tavsiye edilir)

```sh
vim docker-compose.yml 
```

Bu noktada işlemleri teker teker yazmak yerine size verilen komutları yazmanız sonrasında ise bunun üstünden yorumlamamız daha doğru olacaktır.Bu doğrultuda aşağıda size vermiş olduğumuz komutları docker-compose.yml dosyanızın içine yapıştırabilirsiniz.

```sh
version: '3.0'

services:
  minio:
    image: minio/minio:latest
    container_name: minio
    networks:
      - ozanet
    ports:
      - 9000:9000
      - 9001:9001
    environment:
      MINIO_ROOT_USER: ozancannn
      MINIO_ROOT_PASSWORD: ozancannn
      MINIO_ACCESS_KEY: ozancannn
      MINIO_ACCESS_SECRET: ozancannn
    command: 
      - server
      - /data/mybucket0
      - --console-address
      - ":9001"
    volumes:
      - /backups:/data/mybucket0
      
             
  web1:
    image: nginx:latest
    ports:
      - "1001:80"
    volumes:
      - bbk:/usr/share/nginx/html
    networks:
      - ozanet

  web2:
    image: nginx:latest
    ports:
      - "1002:80"
    volumes:
      - bbk:/usr/share/nginx/html
    networks:
      - ozanet

  backup:
    image: jareware/docker-volume-backup
    volumes:
      - bbk:/backup/nginx-data:ro
      - /backups:/archive
    networks:
      - ozanet
    environment:
      BACKUP_CRON_EXPRESSION: "* * * * *"   
    
volumes:
  bbk:
  


networks:
  ozanet:
    driver: bridge

```
Şimdi yukarıda oluşturmuş olduğumuz docker-compose.yml dosyamızın mantığını açıklayabiliriz.
- 
 - Minio, açık kaynaklı bir nesne depolama (object storage) sistemidir.Yedeğini alacağımız volume'u burada oluşturduğumuz bir bucket içinde saklayacağız.
 - Nginx, bir web sunucu ve ters proxy sunucu yazılımıdır.Burada bulunan 2 adet nginx sunucularımızın /usr/share/nginx/html dizinlerini bbk volume'na bağladık
 - Jareware Docker volume'larını yedekleme işlemini sağlayan bir araçtır.Burda yaptığımız ise önce jareware containerımızın nginx-data dizinini sadece read only olarak bbk volume'ına bağlamak böylelikle o volume'a nginx serverler tarafından yazılan dosyaları görüp archive dizinine yedekleyebilecek.Sonrasında ise archive dizinimizi ise tekrar lokal makinemizdeki backups dizinine eşleyerek backup'ımızı hem jareaware üstüne hemde lokal makinemize almış olduk.Burdaki BACKUP_CRON_EXPRESSION: "* * * * *" işlemin hemen sonuç alması için her dakika şeklinde ayarlanmıştır.Yani ' * * * * * ' yazarak dakikada birkere backup al demiş olduk.Bu işlem tamamlandığında iki farklı yerde backup dosyamız bulunuyor hale geldi.Şimdi ise bu dosyamızı minio s3 bucket'ımıza aktarma noktasını inceleyelim.
 








## Volume Backuplarımız Minio Bucket'a aktarma
Artık backuplarımızı aldığımıza göre bunları S3 bucketlarımıza aktarma noktasında ne yaptık bu konuyu inceleyebiliriz..Bunun için tekrar docker-compose.yml dosyamızda bir inceleme yapacağız.
Bu noktada odaklanmamız gereken kısımlar docker-compose.yml dosyamızın minio bölümünde command ve volume kısımları.Command kısmında vermiş olduğumuz ''- server - /data/mybucket0''' komutuyla minio containeri ayağa kalkarken içine bir bucket0 eklemiş olduk.Volumes kısmında ise ''volumes:
      - /backups:/data/mybucket0" komutu ile bu containerimdaki bucket'ımı direk olarak daha öncede bahsetmiş olduğum lokalde /backups dizini altında tutmuş olduğum backup dosyalarımın altına bağlamış oldum.Böylelikle aldığım her bakcup lokaldeli /backups dizininden miniodaki /data/mybucket0 pathindeki bucket0'ıma yazılmış olacak.Bu işlem sonrasında docker-compose.yml dosyamızı başkattığımız takdirde backuplar otomatik olarak bucket'a atılmış olacak aslında,bizim tek yapmamız gereken ise dosyayı çalıştırıp kontrol etmek.
      Oyüzden öncelikle docker-compose.yml dosyamızın bulunduğu dizinde 
      
```sh 
docker-compose up
```
komutunu çalıştırıyoruz.

Sonrasında minioya bağlanıp kontrol etmek için önce;
```sh 
docker ps
```
komutunu çalıştırıp container idsini öğreniyoruz.Sonrasında ise;
```sh 
docker exec -it <container id> bash
```
komutu ile data containerımıze shellden bağlanıyoruz.Şimdi bucketa gidip backuplar gelmişmi kontrol edeceğiz bunun için;
```sh 
cd /data/mybucket0
```
komutu ile mybucket0 içine geldikten sonra;
```sh 
ls
``` 
komutunu çalıştırdığımda backup dosyalarımı burda görebiliyorum.

## KAPANIŞ


Tüm işlemler tamamlandı ve sonuç olarak volume backup dosyam jahaware containerinin /archive dizini,lokal makinamın /backups dizini ve minio s3 contaınerımda /data/bucket0 içinde olmak üzere üç farklı lokasyonda tutuluyor hale geldi.

## Son
Sizlere bu senaryorda backup mantığını ve container mimarisinde nasıl kullanılabileceğine dair bir örneğini anlatmaya çalıştım.Bu mantık birçok farklı durumda farklı şekilde uygulanabilir,zira bende uygulanıp anlatabilecek birçok farklı senaryo arasından bunu seçtim.Bu senaryo sadece bir örneği gösteriyor.İlgilendiğiniz için teşekkürler.

























  
