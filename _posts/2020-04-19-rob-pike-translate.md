---
layout: post
title: Go at Google Language Design in the Service of Software Engineering
image: /img/golang.png
tags: [go, programming, translation]
comments: true
---

[Orijinal makale](https://talks.golang.org/2012/splash.article#TOC_16)

**1. Ön Söz**

Bu yazı Rob Pike tarafından _SPLASH 2012_ konderansında verilen konuşmanın önemli noktalarının derlenmesidir.

Go programlama dili Google altyapısındaki bazı sorunlara çözüm bulmak amacıyla 2007 yılının sonlarında tasarlanmıştır. Günümüz bilgisayar donanımları, çoğunlukla kullanılan C++, Java ve Python ile uyumlu değildir. _Multicore processors, Networked systems, Massive computation clusters ve web programming_ gibi teknolojilerin getirdiği sorunlara, doğru çözümlerin bulunmasından ziyade günü kurtaracak çözümler sunulmaktadır. Ayrıca uygulamaların ölçeklenebilirliği değişmekte, günümüz sunucu taraflı programları on milyonlarca kod satırından oluşmakta.Ve bu uygulamalar yüzlerce hatta binlerce programcı tarafından neredeyse hergün güncellenmekte. Ne yazık ki, derleme zamanı, gelişmiş _compilation cluster_ 'da bile, dakikalara hatta saatlere kadar uzadı.

Go bu _environment_ içinde çalışmayı daha üretken hale getirmek için tasarlandı. _Built-in concurrency_ ve _garbage collection_ gibi iyi bilinen yönlerinin yanı sıra, GO'nun tasarımı iyi hazırlanmış bağımlılık yönetimi, sistemin büyümesine göre adapte olabilen yazılım mimarisi ve uygulama parçaları arasındaki sağlam sınırlandırılmalara dayanmaktadır.

Bu makalede amaçlanan, bu gibi sorunların hafif ve uygun bir programlama dili tasarlanırken nasıl ele alındığını açıklamaktır. Google'da karşılaşılan sektöre dayalı sorunlardan örnekler ve açıklamalar aktarılıcaktır.

**2. Giriş**

GO dili, Google tarafından geliştirilen, derlenen, eşzamanlı, _garbage-collected_, statik tipli bir programlama dilidir. Açık kaynak olarak sunulur.

Go programlama dili, verimli, ölçeklenebilir ve üretkendir. Bazı programcılar Go ile çalışırken keyif alırken, diğerleri sığ ve hatta sıkıcı bulabilir.Bu makalede, bu gibi durumların neden çelişki barındırmadığını açıklayacağız. Go, Google' da yazılım geliştirme alanında karşılaşılan sorunlara çözüm bulmak amacıyla tasarlandı. Ve bu süreç sonunda, çığır açan bir araştırma dili olmasa da, büyük yazılım projeleri için mükemmel bir araç olan bir dilin geliştirilmesine yol açtı.

**3. Google' da Go**

Go programlama dili Google tarafından Google'ın problemlerine çözüm bulması amacıyla geliştirildi. Ve Google'ın ciddi anlamda büyük problemlemleri bulunmaktadır.

Donanım ve yazılım karmaşık ve büyüktür. C++ ve çoğunlukla Java ve Python ile yazılmış milyonlarca satır yazılım bulunur. Binlerce mühendis büyük bir parça olan bu yazılım sisteminin yalnızca tek bir parçası üzerinde çalışır. Bu nedenle bu sistem içinde günden güne önemli değişiklikler meydana gelir. [Custom-designed distributed build system](http://google-engtools.blogspot.com/2011/06/build-in-cloud-accessing-source-code.html)
geliştirmeyi ölçeklenebilir kılsada, hala sistem karmaşık ve büyüktür.

Ve elbette, tüm bu yazılım, az sayıda bağımsız, ağa bağlı işlem kümesi olarak kabul edilen zilyonlarca makine üzerinde çalışır.

![DataCenter](/img/datacenter.jpg)

Kısaca Google' da yazılım geliştirme süreci karmaşıktır, yavaş hatta hantal bile olabilir. Fakat etkilidir.

Go projesinin amacı geliştirme sürecinde bu hantallığı ve yavaşlığı azaltmaktır. Böylece bu süreci üretken ve ölçeklenebilir kılmaktır. Bu programlama dili, geniş ölçekli yazılım sistemlerini yazan, okuyan, bakımını sağlayan kişiler tarafından ve bu kişiler için tasarlandı.

Bu nedenle, Go' nun amacı yazılım dili tasarımı üzerine araştırma yapmaktan ziyade, tasarımcılar için ve birlikte çalışan mühendisler için çalışma ortamını geliştirmek ve bu çalışma ortamı için uygun hale getirmektir.
Go dili, programla dili araştırmasından ziyade, yazılım mühendisliği ile ilgilir. Daha açıkça belirtmek gerekirse, yazılım mühendiliğinin hizmet kısmındaki dil tasarımı denilebilir.

Ancak bir dil yazılım mühendisliğine nasıl yardımcı olabilir? Bu makalenin geri kalanı bu sorunun cevabıdır.

**4. Sıkıntılı Noktalar**

Go duyurulduğunda, bazıları tarafından gelişmiş dillerde olması gereken özelliklerin veya metodojilerin bulunmadığı iddia edildi. Go dili bu gibi önemli özellikler olmadan nasıl kayda değer olabilir? Buna cevabımız Go'nun sahip olduğu özelliklerin büyük ölçekli yazılım geliştirmeyi zorlaştıran sorunları ele almasıdır.

- Yavaş derleme
- Controlsüz bağımlılıklar
- Her programcının dilin farklı bir aracını kullanması
- Programın anlaşılabilirliğinde sorun (zor okunan kod, eksik veya hatalı döküman vb.)
- Kod tekrarı
- Güncellemelerin maliyeti
- Sürüm karmaşası
- Otomatik araçların yazım zorluğu
- Karışık dil derlenmesi

Bir dilin asli özellikleri bu sorunları ele almaz. Yazılım mühendisliğine daha geniş bir bakış açısı gerektiği açıktır. Ve Go'nun tasarımında bu sorunların çözümlerine odaklanmaya çalıştık.

Basit olarak, örneğin, program yapısının bir örneğini göz önünde bulundurun. Bazıları Go'nun C türü süslü parantezlerle sağlanan blok yapısına, Python veya Haskell deki gibi boşlukların kullanılması gerektiğini düşünerek itiraz ettiler. Fakat, biz diller arası yapıların neden olduğu derleme ve test hatalarını izleme konusunda geniş deneyime sahibiz. Örneğin, başka bir dilde, örneğin bir SWIG çağırma yoluyla gömülü olan Python snippet'i, çevreleyen kodun girintisindeki bir değişiklik nedeniyle dikkatten kaçan ve görünmez bir şekilde bozulmuştur.
Bizim bu duruma karşı bakış açımız, girinti için boşlukların kullanılması küçük programlar için daha sağlıklı olsada, ölçeklenebilir değil ve daha büyük ve karışık kod üzerinde, daha fazla sorun yaratabilir. Güvenlik ve güvenilirlik açısından kolaylıktan vazgeçmek daha iyidir, bu nedenle Go'nun küme parantezle oluşturulmuş blokları vardır.

**5. C ve C++ daki Bağımlılıklar**

Paket bağımlılıklarının ele alınmasında ölçeklendirmenin ve diğer sorunların daha önemli bir örneği ortaya çıkıyor. Tartışmaya paket bağımlıklarının C ve C++'da nasıl çalıştıklarını gözden geçirerek başlıyoruz.

_ANSI C_, ilk standartlaştırılması 1989'da, _header_ dosyalarında bulunan **#ifndef** _koruyucuları_ fikrini öne sürdü. Şimdi her yerde bulunan fikir, her _header_ dosyasının bir koşullu derleme yan ifadesi ile parantez içine alınmasıdır, böylece dosya hatasız bir şekilde birden çok kez _include_ edilebilir. Örneğin, Unix _header_ dosyası olan **<sys/stat.h>** aşağıdaki gibidir:

```
/* Large copyright and licensing notice */
#ifndef _SYS_STAT_H_
#define _SYS_STAT_H_
/* Types and other definitions */
#endif
```

Buradaki amaç, C önişlemcisinin dosyayı okuması ancak ikici ve sonraki okumalarda bu bölümü dikkate almamasıdır. Bir sembol olan **_SYS_STAT_H_**, dosyanın ilk okunmasında tanımlanır ve koruyucu görevi üstlenir. Diğer okumalarda tanımlı olduğu için tekrar tanımlanmaz.

Bu tasarımın bir çok faydalı özelliği varıdr, en önemlisi ise, her bir _header_ dosyası diğer _header_ dosyaları dahil olsa bile tüm bağımlılılarını güvenli bir şekilde _#include_ edebilir.

Fakat bu tasarımın ölçeklenmesi oldukça kötüdür.

1984'te, Unix **ps** komutunun kaynağı olan bir ps.c derlemesinin, tüm ön işlemlerin tamamlanmasına kadar 37 kez _<sys/stat.h>_ 'ı _#include_ ettiği gözlendi. Hatta bunu yaparken, içerikler 36 kere atılsa bile, çoğunlukta olan C _implementation_'ları 37 kere dosyayı açacak, okuyacak ve yazacaktır. Aslında, bu davranış C ön işlemcisinin potansiyel olarak karmaşık makro semantiği için gereklidir.

C programlarındaki yazılımın etkisi _#include_ ifadesinin kademeli olarak birikmesidir. Bunların etkisi programın çalışmasını sekteye uğratmayacaktır, ayrıca bu ifadelere gereksinim olup olmadığı anlamakta bir hayli zordur. _#include_ ifadesini silmek ve programı yeniden derlemek bu bilinmezliği test etmek için yeterli değildir, çünkü başka bir _#include_ silinen _#include_ u içerebilir.

Teknik konuşmak gerekirse, bu duruma bir çözüm bulunabilir. _#ifndef_ korumalarının kullanımıyla ilgili uzun vadeli problemleri fark eden **Plan 9** kütüphanelerinin tasarımcıları, **ANSI** standardı olmayan farklı bir yaklaşım benimsediler. **Plan 9**'da başlık dosyalarının _#include_ ifadelerini içermesi yasaklandı ve _#include_ ifadesinin tümünün üst düzey C dosyasında olması gereksinimi belirtildi. Bu tasarımın elbette bağımlıkların doğru sıralanması gibi önemli bir gereksinim içermekteydi. Ancak dökümantasyonlar yardımcı oldu ve olumlu sonuçlar verdi. Sonuç olarak, bir C kaynak dosyası ne kadar bağımlılığa sahip olursa olsun, kaynak dosyası derlenirken her _#include_ ifadesi tam olarak bir kez okundu. Ve elbette, _#include_ ifadesinin çıkarılarak gerekli olup olmadığını anlamak da kolaylaştı.

**Plan 9** yaklaşımımın en önemli sonucu daha hızlı bir derleme sürecidir. Toplam I/O işlemleri _#ifndef_ koruyucularının kullanılmasına göre önemli miktarda azaldı. (Burada ki durum şu şekilde ifade edilebilir. Tekrarlanmamısından dolayı daha az object file okuyacağı için, bu maaliyet azalmıştır)

**Plan 9**'un dışında koruyucular C ve C++ için kabul edilen bir tasarımdır. Aslında, C ++ daha detaylı şekilde aynı yaklaşımı kullanarak sorunun daha da kötüleşmesine yol açar. Tasarımsal olarak, C++ programları genel olarak her bir _class_ veya birden fazla küçük çaplı _class_ başına bir _header_ dosyası olarak tasarlanmıştır. Örnek olarak _<stdio.h>_'ı alabiliriz. Böylece bağımlılık ilişkisi daha da karmaşık hale gelmiş. Kütüphane bağımlılığında ziyade tür hiyerarşisini yansıtır hale gelmiştir. Buna ek olarak, C++ daki _header_ dosyaları basit sabit ve fonksiyon tanımlamalarından ziyade gerçek kod olarak tanımlanan _type_, _method_ ve _template declaration_ içerir. Böylece, C++ derleyiciye derlenemesi zor olan kodu yollamakla kalmaz, derleyici her çağrıldığında bu bilgiyi yeniden işlemek durumunda kalır. Büyük çaplı bir C++ binary'si derlendiğinde, derleyici binlerce kez _<string>_ _header_ dosyasına göre _string_ ifadesini nasıl tanımlayacağını anlamaya çalışır. (Bir kayda göre, 1984 yılları civarında Tom Cargill tarafından gözlenen bağımlık yönetimi için C önişlemcisinin kullanılması durumunda, C++ için uzun vadede bir dezavantaj olucağı dile getirildi.)

Google'da ise, yalnızca bir C++ _binary_ dosyasının oluşturulması, yüzlerce farklı _header_ dosyasını açıp okunmasına neden olucaktır. 2007 yılında, Google mühendisleri oldukça büyük bir _binary_ dosyasının derlemesini gerçekleştirdiler. Bu dosya binlerce dosyadan oluşmaktaydı. Eğer basitçe birleştirilseydi toplamı 4.2 megabyte olacaktı. _#include_ işlemleri bittikten sonra, 8G'lık içerik derleyiciye bırakılır buda her bir C++ kaynak kodunun kendini 2000 byte şişirmesine yol açacaktır.

Diğer yandan, 2003 yılında Google'ın derleme sistemi basit bir _Makefile_'dan daha iyi yönetilen ve daha açık bağımlıklarla oluşturulan dosya bazlı tasarıma dönüştürüldü. Tipik bir _binary_ dosyası sadece doğru bağımlıkların kaydedilmesinden ötürü yaklaşık %40 küçüldü. Bu duruma rağmen, C++(veya C) özellikleri, otomatik olarak bu bağımlıkların doğrulanmasını sağlamaz ve bugün hala bizim büyük çaplı Google' a ait C++ _binary_'lerindeki bağımlılık gereksinimini tam olarak anlayamıyoruz.

Bu kontrolsüz bağımlıklarının ve ölçekleme sorununun sonucu olarak, Google sunucu _binary_'lerinin tek bir bilgisayarda derlenmesi pek de pratik olmadığı ortaya çıktı. BU yüzden geniş dağıtılmış derleme sistemeleri tasarlandı. Ve bu sistemlere, birden fazla makine dahil edilerek, daha fazla önbellek ile, ve daha fazla karmaşıklık ile (bu derleme sistemi kendi başına büyük bir programdır) Google' da derlemeyi pratik bir hale dönüştürdü, fakat hala hantal olmaya devam ediyor.

Dağıtılmış derleme sistemi ile birlikte düşünülecek olsa dahi, büyük bir Google derlemesi dakikalar almaktadır. 2007'deki bu _binary_ dağıtılmış derleme sistemi kullanılarak 45 dakika aldı, şuan ki sürümü 27 dakika almakta, tabii bu süreçte programın kendisi ve bağımlılıkları büyüdü. Derleme sisteminin ölçeklenmesini sağlamak için gerekli olan bu mühendislik çabası yazılım inşa edildikçe artmaya devam etmektedir.

**5. Go'ya Giriş**

Derleme yavaşladığında, bu konu hakkında kafa yormanın zamanı gelmiştir. Go'nun kökeniyle ilgili, 45 dakikalık bir derleme sürecinde go fikrinin ortaya çıktığına dair söylenen bir efsane vardır. Sonuç olarak Google Web Servisleri gibi programlar için yazılacak olan yeni bir dilin tasarlanmasının gerektiği ortaya çıktı. Bu dilde Google programcılarının hayatını kolaylaştıracağı ortada olduğu görüldü.

Şimdiye kadar tartışma bağımlılıklara odaklanmış olsa da, dikkat edilmesi gereken birçok konu var.Herhangi bir dilin bu bağlamda başarılı olması için dikkat edilmesi gereken başlıca noktalar şunlardır:

- Büyük programlar ve çok sayıda programcıdan oluşan takımları ile ölçeklenebilir olmalıdır
- Kabaca C gibi bilindik olmalıdır. Google daki daha kariyerinin başındaki programcılar daha çok prosedürel yazılım dillerine aşinadır özellikle C ailesinden olan yazılım dilleri ile. Bu yüzden bu tür programcıların yeni bir dile hızlıca aşina olmaları için dilin çokda radikal olmaması gerekir.
- Modern olmalıdır, C, C++ ve bazı yönlerinden Java oldukça eskidir. Ve bu tarz programlama dilleri, çok sayıda çekirdekten oluşan makinelerden, _networking_'den ve web uygulamaları geliştirmesinden önce tasarlanmıştır. Modern dünyanın _built-in concurrency_ gibi daha yeni yaklaşımlarla daha iyi karşılanan özellikleri olmalıdır.

Bunlardan yola çıkarak, Go'nun tasarımına yazılım mühendisliği perspektifinden bakalım.

**6. Go'da Bağımlılıklar**

C ve C++ daki bağımlılıkları detaylı bir şekilde inceledikten sonra, Go'nun bu durumu nasıl ele aldığını görmek için iyi bir başlangıç olacaktır. Bağımlılıklar dil tarafından, sözdizimsel ve semantik olarak dil tarafından tanımlanır. Açık bir şekilde tanımlanmış, nettir ve "hesaplanabilir" yani analiz edilmesi için gerekli araçlar tasarlanabilir.
