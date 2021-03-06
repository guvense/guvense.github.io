---
layout: post
title: Go at Google Language Design in the Service of Software Engineering (Türkçe Çeviri)
image: /img/golang.png
tags: [go, programming, translation]
comments: true
---

[Orijinal makale](https://talks.golang.org/2012/splash.article#TOC_16)

**1. Ön Söz**

Bu yazı Rob Pike tarafından _SPLASH 2012_ konderansında verilen konuşmanın önemli noktalarının derlenmesidir.

Go programlama dili Google altyapısındaki bazı sorunlara çözüm bulmak amacıyla 2007 yılının sonlarında tasarlanmıştır. Günümüz bilgisayar donanımları, çoğunlukla kullanılan C++, Java ve Python ile uyumlu değildir. _Multicore processors, Networked systems, Massive computation clusters ve web programming_ gibi teknolojilerin getirdiği sorunlara, doğru çözümlerin bulunmasından ziyade günü kurtaracak çözümler sunulmaktadır. Ayrıca uygulamaların ölçeklenebilirliği değişmekte, günümüz sunucu taraflı programları on milyonlarca kod satırından oluşmakta.Ve bu uygulamalar yüzlerce hatta binlerce programcı tarafından neredeyse her gün güncellenmektedir. Ne yazık ki, derleme zamanı, gelişmiş _compilation cluster_ 'da bile, dakikalara hatta saatlere kadar uzadı.

Go bu _environment_ içinde çalışmayı daha üretken hale getirmek için tasarlandı. _Built-in concurrency_ ve _garbage collection_ gibi iyi bilinen yönlerinin yanı sıra, GO'nun tasarımı iyi hazırlanmış bağımlılık yönetimi, sistemin büyümesine göre adapte olabilen yazılım mimarisi ve uygulama parçaları arasındaki sağlam sınırlara dayanmaktadır.

Bu makalede amaçlanan, bu gibi sorunların uygun bir programlama dili tasarlanırken nasıl ele alındığını açıklamaktır. Google'da karşılaşılan sektöre dayalı sorunlardan örnekler ve açıklamalar aktarılıcaktır.

**2. Giriş**

GO dili, Google tarafından geliştirilen, derlenen, eşzamanlı, _garbage-collected_, statik tipli bir programlama dilidir. Açık kaynak olarak sunulur.

Go programlama dili, verimli, ölçeklenebilir ve üretkendir. Bazı programcılar Go ile çalışırken keyif alırken, diğerleri sığ ve hatta sıkıcı bulabilir.Bu makalede, bu gibi durumların neden çelişki barındırmadığını açıklayacağız. 
Go, Google' da yazılım geliştirme alanında karşılaşılan sorunlara çözüm bulmak amacıyla tasarlandı. Ve bu süreç sonunda, çığır açan bir araştırma dili olmasa da, büyük yazılım projeleri için mükemmel bir araç olan bir dilin geliştirilmesine yol açtı.

**3. Google' da Go**

Go programlama dili Google tarafından Google'ın problemlerine çözüm bulması amacıyla geliştirildi. Ve Google ciddi anlamda büyük problemler ile karşılaşmaktadır.

Donanım ve yazılım karmaşık ve büyüktür. C++ ve çoğunlukla Java ve Python ile milyonlarca satır yazılım kodu yazılmıştır. Binlerce mühendis büyük bir parça olan bu yazılım sisteminin yalnızca tek bir parçası üzerinde çalışır. Bu nedenle bu sistem içinde günden güne önemli değişiklikler meydana gelir. [Custom-designed distributed build system](http://google-engtools.blogspot.com/2011/06/build-in-cloud-accessing-source-code.html)
geliştirmeyi ölçeklenebilir kılsada, hala sistem karmaşık ve büyüktür.

Ve elbette, tüm bu yazılım, az sayıda bağımsız, ağa bağlı işlem kümesi olarak kabul edilen zilyonlarca makine üzerinde çalışır.

![DataCenter](/img/datacenter.jpg)

Kısaca Google' da yazılım geliştirme süreci karmaşıktır, yavaş hatta hantal bile olabilir. Fakat etkilidir.

Go projesinin amacı geliştirme sürecinde bu hantallığı ve yavaşlığı azaltmaktır. Böylece bu süreci üretken ve ölçeklenebilir kılmaktır. Bu programlama dili, geniş ölçekli yazılım sistemlerini yazan, okuyan, bakımını sağlayan kişiler tarafından ve bu kişiler için tasarlandı.

Bu nedenle, Go' nun amacı yazılım dili tasarımı üzerine araştırma yapmaktan ziyade, tasarımcılar için ve birlikte çalışan mühendisler için çalışma ortamını geliştirmek ve bu çalışma ortamı için uygun hale getirmektir.
Go dili, programla dili araştırmasından ziyade, yazılım mühendisliği ile ilgilir. Daha açıkça belirtmek gerekirse, yazılım mühendiliğinin hizmet kısmındaki dil tasarımı denilebilir.

Ancak bir dil yazılım mühendisliğine nasıl yardımcı olabilir? Bu makalenin geri kalanı bu sorunun cevabıdır.

**4.Sorunlu Noktalar**

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

Bir dilin asli özellikleri bu sorunları ele almaz. Yazılım mühendisliğine daha geniş bir bakış açısı gerektiği açıktır. Ve Go'nun tasarımında bu sorunların çözümlerine odaklanılmaya çalışılmıştır.

Basit olarak, örneğin, program yapısının bir örneğini göz önünde bulundurun. Bazıları Go'nun C türü süslü parantezlerle sağlanan blok yapısına, Python veya Haskell deki gibi boşlukların kullanılması gerektiğini düşünerek itiraz ettiler. Fakat, biz diller arası yapıların neden olduğu derleme ve test hatalarını tespit etme konusunda geniş deneyime sahibiz. Örneğin, başka bir dilde, örneğin bir SWIG çağırma yoluyla gömülü olan Python snippet'i, çevreleyen kodun girintisindeki bir değişiklik nedeniyle dikkatten kaçan ve görünmez bir şekilde bozulmuştur.
Bizim bu duruma karşı bakış açımız, girinti için boşlukların kullanılması küçük programlar için daha sağlıklı olsada, ölçeklenebilir değil ve daha büyük ve karışık kod üzerinde, daha fazla sorun yaratabilir. Güvenlik ve güvenilirlik açısından kolaylıktan vazgeçmek daha iyidir, bu nedenle Go'nun küme parantezle oluşturulmuş blokları vardır.

**5. C ve C++ daki Bağımlılıklar**

Paket bağımlılıklarının ele alınmasında ölçeklendirmenin ve diğer sorunların daha önemli bir örneği ortaya çıkıyor. Tartışmaya paket bağımlıklarının C ve C++'da nasıl çalıştıklarını gözden geçirerek başlıyoruz.

_ANSI C_, ilk standartlaştırılması 1989'da, _header_ dosyalarında bulunan **#ifndef** _koruyucuları_ fikrini öne sürdü. Şimdi her yerde bulunan bu fikir, her _header_ dosyasının bir koşullu derleme yan ifadesi ile parantez içine alınmasıdır, böylece dosya hatasız bir şekilde birden çok kez _include_ edilebilir. Örneğin, Unix _header_ dosyası olan **<sys/stat.h>** aşağıdaki gibidir:

```
/* Large copyright and licensing notice */
#ifndef _SYS_STAT_H_
#define _SYS_STAT_H_
/* Types and other definitions */
#endif
```

Buradaki amaç, C önişlemcisinin dosyayı okuması ancak ikici ve sonraki okumalarda bu bölümü dikkate almamasıdır. Bir sembol olan **_SYS_STAT_H_**, dosyanın ilk okunmasında tanımlanır ve koruyucu görevi üstlenir. Diğer okumalarda tanımlı olduğu için tekrar tanımlanmaz.

Bu tasarımın bir çok faydalı özelliği varıdr, en önemlisi ise, her bir *header* dosyası diğer *header* dosyaları dahil olsa bile tüm bağımlılılarını güvenli bir şekilde _#include_ edebilir.

Fakat bu tasarımın ölçeklenmesi oldukça kötüdür.

1984'te, Unix **ps** komutunun kaynağı olan bir ps.c derlemesinin, tüm ön işlemlerin tamamlanmasına kadar 37 kez _<sys/stat.h>_ 'ı _#include_ ettiği gözlendi. Hatta bunu yaparken, içerikler 36 kere atılsa bile, çoğunlukta olan C _implementation_'ları 37 kere dosyayı açacak, okuyacak ve yazacaktır. Aslında, bu davranış C ön işlemcisinin potansiyel olarak karmaşık makro semantiği için gereklidir.

C programlarındaki yazılımın etkisi _#include_ ifadesinin kademeli olarak birikmesidir. Bunların etkisi programın çalışmasını sekteye uğratmayacaktır, ayrıca bu ifadelere gereksinim olup olmadığı anlamakta bir hayli zordur. _#include_ ifadesini silmek ve programı yeniden derlemek bu bilinmezliği test etmek için yeterli değildir, çünkü başka bir _#include_ silinen _#include_ u içerebilir.

Teknik konuşmak gerekirse, bu duruma bir çözüm bulunabilir. _#ifndef_ korumalarının kullanımıyla ilgili uzun vadeli problemleri fark eden **Plan 9** kütüphanelerinin tasarımcıları, **ANSI** standardı olmayan farklı bir yaklaşım benimsediler. **Plan 9**'da başlık dosyalarının _#include_ ifadelerini içermesi yasaklandı ve _#include_ ifadesinin tümünün üst düzey C dosyasında olması gereksinimi belirtildi. Bu tasarımın elbette bağımlıkların doğru sıralanması gibi önemli bir gereksinim içermekteydi. Ancak dökümantasyonlar yardımcı oldu ve olumlu sonuçlar alındı. Sonuç olarak, bir C kaynak dosyası ne kadar bağımlılığa sahip olursa olsun, kaynak dosyası derlenirken her _#include_ ifadesi tam olarak bir kez okundu. Ve elbette, _#include_ ifadesinin çıkarılarak gerekli olup olmadığını anlamak da kolaylaştı.

**Plan 9** yaklaşımımın en önemli sonucu daha hızlı bir derleme sürecidir. Toplam I/O işlemleri *#ifndef* koruyucularının kullanılmasına göre önemli miktarda azaldı. 

**Plan 9**'un dışında koruyucular C ve C++ için kabul edilen bir tasarımdır. Aslında, C ++ daha detaylı şekilde aynı yaklaşımı kullanarak sorunun daha da kötüleşmesine yol açar. Tasarımsal olarak, C++ programları genel olarak her bir *class* veya birden fazla küçük çaplı class başına bir header dosyası olarak tasarlanmıştır. Örnek olarak *< stdio.h >*'ı alabiliriz. Böylece bağımlılık ilişkisi daha da karmaşık hale gelmiş. Kütüphane bağımlılığında ziyade tür hiyerarşisini yansıtır hale gelmiştir. Buna ek olarak, C++ daki header dosyaları basit sabitler ve fonksiyon tanımlamalarından ziyade gerçek kod olarak tanımlanan tip, metot ve *template* tanımlamalarını içerir. Böylece, C++ derleyiciye derlenmesi zor olan kodu yollamakla kalmaz, derleyici her çağrıldığında bu bilgiyi yeniden işlemek durumunda kalır. Büyük çaplı bir C++ binary'si derlendiğinde, derleyici binlerce kez *< string >* header dosyasına göre string ifadesini nasıl tanımlayacağını anlamaya çalışır. (Bir kayda göre, 1984 yılları civarında Tom Cargill tarafından gözlenen bağımlık yönetimi için C önişlemcisinin kullanılması durumunda, C++ için uzun vadede bir dezavantaj olucağı dile getirildi.)

Google'da ise, yalnızca bir C++ *binary* dosyasının oluşturulması, yüzlerce farklı *header* dosyasını açıp okunmasına neden olucaktır. 2007 yılında, Google mühendisleri oldukça büyük bir *binary* dosyasının derlemesini gerçekleştirdiler. Bu dosya binlerce dosyadan oluşmaktaydı. Eğer basitçe birleştirilseydi toplamı 4.2 megabyte olacaktı. *#include* işlemleri bittikten sonra, 8G'lık içerik derleyiciye bırakılır buda her bir C++ kaynak kodunun kendini 2000 byte şişirmesine yol açacaktır.

Diğer yandan, 2003 yılında Google'ın derleme sistemi basit bir *Makefile*'dan daha iyi yönetilen ve daha açık bağımlıklarla oluşturulan dosya bazlı tasarıma dönüştürüldü. Tipik bir *binary* dosyası sadece doğru bağımlıkların kaydedilmesinden ötürü yaklaşık %40 küçüldü. Bu duruma rağmen, C++(veya C) özellikleri, otomatik olarak bu bağımlıkların doğrulanmasını sağlamayacaktır ve bugün hala bizim büyük çaplı Google'a ait C++ *binary*'lerindeki bağımlılık gereksinimini tam olarak anlayamıyoruz.

Bu kontrolsüz bağımlıklarının ve ölçekleme sorununun sonucu olarak, Google *binary*'lerinin tek bir bilgisayarda derlenmesi pek de pratik olmadığı ortaya çıktı. BU yüzden geniş dağıtılmış derleme sistemeleri tasarlandı. Ve bu sistemlere, birden fazla makine dahil edilerek, daha fazla önbellek ile, ve daha fazla karmaşıklık ile (bu derleme sistemi kendi başına büyük bir programdır) Google' da derlemeyi pratik bir hale dönüştürdü, fakat hala hantal olmaya devam ediyor.

Dağıtılmış derleme sistemi ile birlikte düşünülecek olsa dahi, büyük bir Google derlemesi dakikalar almaktadır. 2007'deki bu *binary* dağıtılmış derleme sistemi kullanılarak 45 dakika aldı, şuan ki sürümü 27 dakika almakta, tabii bu süreçte programın kendisi ve bağımlılıkları büyüdü. Derleme sisteminin ölçeklenmesini sağlamak için gerekli olan bu mühendislik çabası yazılım inşa edildikçe artmaya devam etmektedir.

**6. Go'ya Giriş**

Derleme yavaşladığında, bu konu hakkında kafa yormanın zamanı gelmiştir. Go'nun kökeniyle ilgili, 45 dakikalık bir derleme sürecinde go fikrinin ortaya çıktığına dair söylenen bir efsane vardır. Sonuç olarak Google Web Servisleri gibi programlar için yazılacak olan yeni bir dilin tasarlanmasının gerektiği ortaya çıktı. Bu dilde Google programcılarının hayatını kolaylaştıracağı net olarak görüldü.

Şimdiye kadar tartışma bağımlılıklara odaklanmış olsa da, dikkat edilmesi gereken birçok konu var.Herhangi bir dilin bu bağlamda başarılı olması için dikkat edilmesi gereken başlıca noktalar şunlardır:

- Büyük programlar ve çok sayıda programcıdan oluşan takımları ile ölçeklenebilir olmalıdır
- Kabaca C gibi bilindik olmalıdır. Google daki daha kariyerinin başındaki programcılar daha çok prosedürel yazılım dillerine aşinadır özellikle C ailesinden olan yazılım dilleri ile. Bu yüzden bu tür programcıların yeni bir dile hızlıca aşina olmaları için dilin çokta radikal olmaması gerekir.
- Modern olmalıdır, C, C++ ve bazı yönlerinden Java oldukça eskidir. Ve bu tarz programlama dilleri, çok sayıda çekirdekten oluşan makinelerden, *networking*'den ve web uygulamaları geliştirmesinden önce tasarlanmıştır. Modern dünyanın *built-in concurrency* gibi daha yeni yaklaşımlarla daha iyi karşılanan özellikleri olmalıdır.

Bunlardan yola çıkarak, Go'nun tasarımına yazılım mühendisliği perspektifinden bakalım.

**7. Go'da Bağımlılıklar**

C ve C++ daki bağımlılıkları detaylı bir şekilde inceledikten sonra, Go'nun bu durumu nasıl ele aldığını görmek için iyi bir başlangıç olacaktır. Bağımlılıklar dil tarafından, sözdizimsel ve semantik olarak dil tarafından tanımlanır. Açık bir şekilde tanımlanmış, nettir ve "hesaplanabilir" yani analiz edilmesi için gerekli araçlar tasarlanabilir.

*package* ifadesinden sonra bir bağımlılıkların *import* edilmesi şu şekildedir. Her bir kaynak dosyası bir veya biden fazla *import* ifadesi içerebilir. *string* sabiti olarak tanımlanan paket *import* ifadesi ile kaynak dosyaya eklenir.

```go
import "encoding/json"
```

Go'yu ölçeklendirmedeki ilk adım, *dependency-wise* olarak tanımlanan kullanılmayan bir bağımlık olması durumunda derleme zamanında hata alınmasıdır. Burada önemli olan *warning* anlamında kullanılar bir **uyarı** değil, **hata**'dır. Yani kullanılmayan bir bağımlılık varsa uygulama derlenmeyecektir. Bu durum yalnızca kaynak dosyada kullanılacak olan bağımlıkları içereceğini garantiler. Bu sayede ekstra kod derlenmeyeceği için derleme süresini kısaltacaktır.

Bir diğer adım ise, *compiler* olarak tanımlanan Go kodunu makine koduna çeviren aracın verimliliği garanti etmesidir. 3 paketli Go programı ve bağımlık grafiğine bir göz atalım: 

- paket A, paket B yi import ediyor *A imports B*
- paket B, paket C yi import ediyor *B imports C*
- paket A, paket C yi import **etmiyor** 

Bu durum A paketinin C paketini B paketi aracı ile dolaylı olarak kullandığı anlamına gelmektedir.A'da kullanılar bazı ifadeler B nin C den aldığı ifadeleri kullanıyor olsa bile, A' nın kaynak kodunda C paketi ile ilgili herhangi bir tanımlayıcı bulunmayacaktır. Örnek vermek gerekise, A, B'de bulunan bir üyesi C den gelen bir *struct*' ı referans alıyor olabilir. Fakat A o üyeyi referans almayacaktır. Daha açıklayıcı bir örnek vermek gerekirse, A' nın B den *formatted I/O* paketi *import* ettiğini düşünelim. Fakat B bu import edilmiş paketi C'den aldığı *buffered I/O* implementasyonunu olarak kullanmaktayken, A kendisi  *buffered I/O* paketini kullanmayacaktır.

Böyle bir programı *build* etme süreci şu şekilde işleyecektir. Öncelikle C derlenecek, bağımlı paketler kendilerine bağımlı paketlerden önce oluşturulmalıdır. Sonra B derlenir, son olarak A derlenir ve daha sonra *link* denilen bir şekilde birbirlerine bağlanırlar.

A derlerdiğinde, derleyici B nin kaynak dosyasını değil *object* dosyasını okur. Bu *object* dosyası onu import edicek paket için gerekli tüm bilgiyi içerir. Örneğin, A paketi içinde şu şekilde bir ifade varsa:

```go
import "B"
```
Bu ifade ile B'nin *object* dosyası okunur ve gerkeli bilgiler A'ya aktarılır. Bu bilgi, B'nin derleme zamanınında C'den ihtiyaç duyduğu het türlü bilgiyi içerir. Diğer bir ifade ile, B derlendiğinde, üretilmiş olan *object* dosyası B' nin *public* olarak sunulmuş tüm arayüzünü içerir.

Derleyici *import* ifadesi ile, ifadenin içine yer alan *string* şeklinde tanımlanan yalnızca bir *object* dosyası çalıştırır. Bu, elbette, bağımlılık yönetimine yönelik *Plan 9 C* (ANSI C'nin aksine) yaklaşımını hatırlatır, ancak aslında, derleyici, Go kaynak dosyası derlendiğinde **header** dosyasını yazacaktır. Bu süreç *Plan 9*'dan daha otomatik ve daha verimlidir. Buna ek olarak veri okunurken, *import* ifadesi değerlendirlirken yanlızca "dışarıya aktarılan" bilgi eklenir, programın genel kaynak kodu eklenmez. Genel derleme süresi üzerindeki etki çok büyük olabilir ve kod tabanı büyüdükçe ölçeklenecektir. Bağımlılık grafiğinin yürütülmesi ve dolayısıyla derlenmesi için gereken süre, C ve C ++ 'ın "*include of include file*" modelinden çok daha az olabilir.

Ek olarak, bu bağımlık yönetime yönelik olan genel yaklaşımın orjinal olmadığını vurgulamak gerekir. Bu fikir, 1970'lere kadar gitmektedir ve Modula-2 ve Ada gibi diller bu yaklaşımı kullanır. C ailesinde ise Java bu yaklaşıma ait öğeler içerir.

Derleme sürecini daha verimli hale getirmek için, *object* dosyasındaki dışa aktarılacak veri ilk olacak şekilde ayarlanabilir. Böylece derleyici bu bölümün sonuca ulaştığında okumayı sonlandırabilir.

Bağımlılık yönetimine bu yaklaşım, Go derlemelerinin C veya C ++ derlemelerinden daha hızlı olmasının en büyük nedenidir. Başka bir faktör, Go'nun dışa aktarma verilerini *object* dosyasına yerleştirmesidir; bazı diller yazarın yazmasını veya derleyicinin bu bilgileri içeren ikinci bir dosya oluşturmasını gerektirir. Bu durum çok fazla dosyanın iki kez açılmasına neden olacaktır. Go'da ise paketin *import* edilmesi için yalnızca bir dosya vardır. Ayrıca, tek dosya yaklaşımı, dışa aktarılacak verilerinin (veya C / C ++ 'da *header* dosyasının) hiçbir zaman *object* dosyasına göre güncelliğini yitiremeyeceği anlamına gelir.

Kaynak kodun nasıl açıldığını görmek amacıyla, önceden C++ ile yazılmış büyük çaplı bir Google programını Go ile yazılmış sürümünün derleme sürecini ölçtük. Yaklaşık 40X, C ++ 'dan elli kat daha iyi (ve daha basit ve dolayısıyla daha hızlı işlenme ) olduğunu bulduk, ancak yine de beklediğimizden daha büyük oldu.
Bunun iki nedeni var. İlk olarak bir hata bulduk: Go derleyicisi, dışa aktarma bölümünde bulunması gerekmeyen önemli miktarda veri üretiyordu. İkincisi, dışa aktarma verileri, geliştirilebilecek karmaşık kodlamalar kullanıyordu. Bu sorunları ele almayı planlıyoruz.

Her şeye rağmen, yapılacak elli katlık bir etki, dakikaları saniyeye çevirir.

Go bağımlık grafiğinin bir başka özelliği ise döngüsel olmamasıdır. Dil, grafikte dairesel bir içe aktarma yapılamayacağını tanımlar.Ek olarak, derleyici ve *linker* her birinin var olup olmadığını kontrol eder. Ara sıra yararlı olmalarına rağmen, dairesel *import*'lar ölçeklenmede önemli sorunlar yaratmaktadır. Derleyicinin daha büyük kaynak dosyaları ile aynı anda ilgilenmesini gerektirir ve bu da artış gösteren derlemeleri yavaşlatır. 

Dairesel *import*'lar bazen sorun çıkmasına neden olabilir. Fakat dallanmayı temiz tutar ve paketler arasında düzgün bir sınır oluşmasını gerektirir. Go'daki tasarım kararlarının çoğunda olduğu gibi, programcıyı öncesinde büyük ölçeklenme sorununu düşünmeye iter. Bu durum daha sonraya bırakılırsa hiçbir zaman tatmin edici bir şekilde ele alınamaz. Standart kütüphanenin tasarımı süresince, bağımlılıkları kontrol edilmesi için büyük çaba harcandı. Küçük bir kodu kopyalamak, bir fonksiyon için büyük bir kütüphaneyi çekmek yerine daha sağlıklı olabilir. Bağımlıkların bu şekilde temiz tutulması kodun yeniden kullanılabilir hale getirecektir. Uygulamada bunun bir örneği olarak, (düşük seviyeli) net paketin, daha büyük ve ağır bir bağımlılık olarak biçimlendirilmiş I/O paketine bağlımlı hale gelmemek için kendi tamsayıdan ondalığa dönüştürme implementasyonuna sahip olması verilebilir.Başka bir örnek ise, "String" dönüştürme paketi olan **strconv**'un, büyük Unicode karakter sınıfı tablolarını çekmek yerine 'yazdırılabilir' karakterler tanımlanmasının özel bir implementasyonuna sahiptir.

**8. Paketler**

Go' nun paket sisteminin tasarımı, kütüphanelerin bazı özelliklerini, *name space*'leri, ve modulleri içerir.
Tüm Go kaynak dosyası, örneğin, *encoding/json/json.go*, **package** ifadesi ile başlar.

```go
package json
```

Burada *json* basit olarak paket ismidir. Paket isimleri genellikle kısa ve nettir.

Bir paketi kullanmak için, paketi *import* eden kaynak dosyasının, paket'in yolunu import ifadesi içinde kullanması gerekiyor.
Buradaki paket yolu ifadesi dil tarafında tanımlanmamıştır. Fakat pratik ve genel olarak *repository*'deki kaynak dosyasının *slash* ile ayrılmış dosya yolu kullanılır.

```go
import "encoding/json"
```

Sonrasında paket ismi kaynak dosyada dosya yolundan bağımsız şekilde kullanılır. Örneğin,

```go
var dec = json.NewDecoder(reader)
```

Bu tasarım daha anlaşılabilir bir yapı olmasına yardımcı olur. 

Paketin yolunun *encoding/json* olmasına rağmen, paketin ismi *json*'dur.Standart *repository*'in dışında, genel anlayış proje isminin veya şirketin dosya yolunun en başına konulmasıdır.

```go
import "google/base/go/log"
```

Paket yollarının benzersiz olduğunu anlamak önemlidir.Ancak paket isimleri için böyle bir gereksinime ihtiyaç bulunmamaktadır. Paket yolu benzersiz olarak kaynak dosyaya *import* edilmesi gerekirken, paket ismi sadece genel bir isimlendirmedir. Bu isimlendirme paketi kullanacak kişilerin kendi kodlarında bu paketi nasıl kullanıcalarına bağlıdır.Paket isminin benzersiz olmasına gerek yoktur ve *import* edilen kaynak dosyası tarafından *import* ifadesi içindeki local bir tanımlayıcı ile ezilebilir.Örnekteki iki *import* ifadesi *package log*' u çağıran iki kaynak dosyasını çağırmaktadır.Fakat ikisini birlikte tek bir kaynak dosyasına *import* etmek istiyorsa, birini **locally** olarak yeniden isimlendirmesi gerekmetedir.

```go
import "log"                          // Standart paket
import googlelog "google/base/go/log" // Google' a özel olarak tanımlanmış paket
```

Tüm şirketlerin kendi *log* paketleri olabilir.Fakat bunun için bu paketlere özgün isim vermelerine gerek yoktur.Buna ters olarak, Go'nun tasarımı paket isimlerinin çakışmasını dikkate alınmasından ziyade kısa ve net tutulmasını önerir.

**9. *Remote* Paketler**

Go' nun paket sisteminin bir diğer önemli özelliği ise paket yoludur.Genellikle isteğe bağlı bir string ile başlar veya farklı bir *repository*'e URL üzerinden erişebilir.

Burada **doozer** paketinin github üzerinden nasıl kullanıldığı gösterilmektedir. Buradaki **go get** komutu **go build** aracını kullanarak repository'i tarar ve paketi indirir.Bir kez indirildiğinde herhangi bir paket gibi kullanılabilir.

```console
$ go get github.com/4ad/doozer // Shell command to fetch package
```

```go
import "github.com/4ad/doozer" // Doozer client's import statement
```
```go
var client doozer.Conn         // Client's use of package
```

Ek olarak **go get** komutunun bağımlıklılıkları yinelemeli olarak indirir.Ayrıca, *import* edilen paketlerin yollarının alan kullanımını URL'lere devredilir. Böylece paket isimlendirmelerini tekilleştirmez ve ölçeklenebilir hale getirir.


**10. Sentaks**

Sentaks bir programlama dilinin kullanıcı arayüzüdür. Sentaks'ın dilin semantiği üzerinde etkisi pek olmasada okunabilirlik ve anlaşılabilirlik anlamında önemli bir etkiye sahiptir.Ayrıca, dilin ayrıştırılması zorsa, otonom araçların yazılımı da aynı şekilde zordur.

Bu yüzden Go'nun tasarım süreci anlaşılabilirlik üzerine kurulmuştur.Bu anlayış Go'ya anlaşılabilir bir sentaks kazandırmıştır. C ailesine ait diğer diller ile karşılaştırıldığında, dilbilgisi açısından, yalnızca 25 anahtar kelime ile, en sade ve az anahtar kelime sayısına sahiptir. (C99 37, C++11 84 ve bu sayılar büyümeye devam etmektedir). Dahada önemlisi, dilbilgisi düzenli olduğu için kolayca ayrıştırılabilir.(Çoğunlukla düzeltebileceğimiz ancak yeterince erken bulamadığımız tuhaflıklar da mevcuttur). C, Java ve özellikle C++' ın aksine, Go tip bilgisi veya sembol tablosu olmadan ayrıştırılabilir. Burada tip bağımlı içerik bulunmamaktadır. Dil bilgisinin anlamlandırmak kolaydır, araçların yazımını kolaylaştırır.

C programcılarını şaşırtan bir detay da Go sentaksının C' den ziyade Paskala benzemesidir. Tanımlanan isim türden önce gelir ve birden fazla anahtar kelime içerir:


```go
var fn func([]int) int
type T struct {a , int}
```

C'ye karşılaştırıldığında

```c
int (*fn)(int[])
struct T {int a, b; }
```

Anahtar kelimeler ile oluşturulan tanımlamalar, kolay bir şekilde hem insanlar için hemde bilgisayarlar için ayrıştırılabilir. Tür için ayrı bir sentaksın olması, C' deki ifade sentaksının aksine ayrıştırma üzerinde önemli bir etkiye sahiptir. Burada dil bilgisine ek bir ifadenin eklendiği görülmektedir, fakat bu durum belirsizliği ortadan kaldırır. Ancak tanımlamaları *initialize* edilmesinde faydalı bir yan etkisi vardır. Bu etki, **var** anahtar kelimesi çıkarıldığında ve değişkenin türü ifadeden çıkarılmasıdır.Aşağıdaki iki adet tanımlama birebir aynıdır. Yalnızca ikincisi daha kısa ve deyimseldir.

```go
var buf *bytes.Buffer = bytes.NewBuffer(x) // açık şekilde belirtilmiş
buf := bytes.NewBuffer(x)                  // türetilmiş
```

Go'daki tanımların sentaksı hakkında daha detaylı bilgi edinmek istiyorsanız, bu blog yazısını inceleyebilirsiniz.

[Go's Declaration Syntax](http://golang.org/s/decl-syntax)

Basit fonksiyonlar için fonksiyon sentaksı gayet açıktır. Verilen örnekte *Abs* fonksiyonunu tanımlanmıştır. Bu fonksiyon yalnızca *T* türünden olan *x*' i parametre olarak kabul eder ve sonucunda yalnızca *float64* değeri döndürür.


```go
func Abs(x T) float64
```

Metot ise özel bir parametre almış bir fonksiyon anlamına gelir. Bu parametre *Receiver* (alıcı) olarak adlandırılır. Receiver standart olan *dot* notasyonu ile fonksiyonlara geçilebilir. Receiver parantez içinde fonksiyon isminden önce olacak şekilde tanımlanır. Verilen örneğin receiver metotu şeklinde tanımlanması:

```go
func (x T) Abs() float64
```

Burada T argumanlı bir variable bulunmaktadır. Go'da fonksiyonlar *first-class* üyedir.


Çevirmen notu:
*first-class*: Fonksiyonel programlamanın temel kavramlarından biri olan bu özellik, foksiyonlarında değişkenler gibi başka bir fonksiyona parametre olarak geçilebilceğini veya başka bir fonksiyonun dönüş değeri olabileceği anlamına gelir.


```go
negAbs := func(x T) float64 {return -Abs(x) }
```

Son olarak, Go'da fonksiyonlar birden çok değer döndürebilir. Genel bir durum ise sonucu ve hatayı birlikte döndürmektir. Örneğin;

```go
func ReadByte() (c byte, err error)

c, err := ReadByte()
if err != nil { ... }
```
Daha sonra hatalar hakkında konuşmaya devam edeceğiz.

Go'da eksik olan özelliklerden biride *default* fonksiyon argumanlarını desteklemiyor oluşudur. Bu durumun nedeni dilin tasarımını sadeleştirmektir. Deneyimlerimizden yola çıkarak *default* argumanların, daha fazla arguman ekleyerek API tasarımının oluşturduğu kusurları düzeltmeyi kolaylaştırdığını görüyoruz. Fakat bu durumun sonucunda çözümlenmesi hatta analaşılması zor etkileşimli argumanlar ortaya çıkacaktır. *Default* argumanların eksikliğinde bir fonksiyonun tüm API  arayüzünü desteklemeyeceği için daha fazla metot veya fonksiyon tanımlaması gerekecektir. Fakat bu durum daha temiz ve anlaşılabilir bir API oluşturulmasına yardım edecektir. Ek olarak bu fonksiyonlar isimlere göre ayrılacağı için hangi parametrenin hangi fonksiyonda var olduğu da isme göre rahatça anlaşılacaktır. Bu durum, netliğin ve okunabilirliğin önemli bir parçası olan *isimlendirmenin* üzerine düşülmesine teşvik edecektir. 

Çevirmen notu:
Default argumanlarını bir Python kodu ile örnekleyelim

```python

def student(firstname, lastname ='Dylan'):
 
     print(firstname, lastname)

student("Bob")  # -> Bob Dylan
student("Bob", "Joe")  # -> Bob Joe
```

Burada görüldüğü gibi methota ikinci parametreyi geçmezsek, ikinci arguman hazır olarak tanımlanmış değeri alacaktır.


*Default* argumanların olmamasının bir hafifletici sonucuda, Go'nun kullanımı kolay, *type-safe* olan *variadic* fonksiyonlara desteği olmasıdır. 

*variadic* fonksiyon örneği:

```go
func printAll(nums ...int) {
    fmt.Print(nums, " ")
}
```

**11. İsimlendirme**

Go, bir değişkenin başka bir paket tarafından erişebilirliğini belirlemek için alışılagelmedik bir yaklaşım belirler.Örneğin, diğer dillerdeki gibi *private* veya *public* anahtar kelimeleri yerine, isimler erişilebilirliğine dair bilgiyi kendi içlerinde saklar. İlk harfin büyük veya küçük olma durumu değikenin erişilebilirliğini belirler.
Eğer değişkenin isminin ilk harfi büyük ise değişken *public* yani diğer paketler tarafından erişilebilir hale gelir.

-   İlk harf büyük ise **Name** örneğin, değişken diğer paketler için erişilebilir hale gelir.
-   İlk harf küçük veya *underscore* ile başlıyorsa **name** veya **_Name** diğer paketler tarafından erişimi kısıtlandırılır.

Bu kural aynı şekilde değişkenler, türler, fonksiyonlar, metotlar, sabitler için de geçerlidir.

Bu durum bizim için kolay bir tasarım kararı değildi. Bir yıldan fazla bir süre değişkenin erişilebilirliğinin nasıl tanımlanacağı ile ilgili çalıştık. Sonunda bu kuralı dile yerleştirdiğimizde, kısa süre sonunda dilin en önemli özelliklerinden biri haline geldiğini farkettik.Bir süre GO'yu kullandıktan sonra başka bir dile geçildiğinde, erişilebilirlik bilgisini anlamanın daha külfetli hala geldiği kanısındayız.

Buradan yine kaynak kodun programcı için açık ve anlaşılabilir olması sonucunu çıkarabiliriz.

Bir başka sadeleştirme ise, Go'nun oldukça sıkı bir *scope* hiyerarşisine sahip olmasıdır:

- global    (önceden tanımlanmış olan değişkenler örneğin *int* ve *string*)
- paket     (paketin tüm kaynak kodları aynı *scope* içinde yer alır)
- *file*    (yanlızca paket içinde *import* edilenler, pratikte çok önemli değil)
- fonksiyon (alışılageldik bir şekilde)
- *scope*   (alışılageldik bir şekilde)

İsim alanı, sınıf veya diğer yapılar için bir *scope* bulunmamaktadır.Go'da isimler oldukça az yerden gelir, ve tüm isimler aynı *scope* hiyerşisini takip eder. Kaynak kodun içinde herhangi bir şekilde verilen bir lokasyonda, bir değişken, nasıl kullanıldığından bağımsız olarak tam olarak bir dil nesnesini belirtir.

Bu durumun kodu daha okunabilir olmasına yol açar. Dikkat edilmesi gereken nokta, metodlar açık bir şekilde *receiver*'ını belirtir ve metodun tipine ve *field*'larına ulaşmak için mutlaka bu receiver methodun kullanılması gerekmektedir. Dilin üstü kapalı olarak tanımladığı *this* anahtar kelimesi yoktur. Bu yüzden her zaman şu şekilde yazılmalıdır.

```go
rcvr.Field
```

(Buradaki *rcvr* **receiver** değişkeni için seçilen isimdir)

Böylece belirli bir tipin her öğesi sözcüksel olarak *receiver* tipinin değerine bağlıdır.Benzer olarak, paket niteliyicisi *import* edilen isimler için de her zaman bu şekilde kullanılır. Örneğin, *Reader* yazımı yerine *io.Reader* yazımı kullanılır. Aslında, standart kitaplıkta *Reader* için veya *Printf* adında birden fazla dışa aktarılan tanımlayıcı vardır.Yine de hangi değişkene atıfta bulunulduğu her zaman açıktır.

Son olarak, bu tür kurallar, *int* gibi üst düzey önceden tanımlanmış adlar dışında, tüm isimlerin mevcut pakette tanımlanmasını garanti eder.

Kısacası isimler yerel değişkenlerdir. C, C++ veya Java gibi dillerde *y* ismi herhangi bir şeye atıfta bulunabilir.Fakat Go'da *y* hatta *Y* her zaman mevcut paketin içinde tanımlanmalıdır. Bu şekilde *x.Y* anlaşılabilir olur. *x* yerel bir değişken olarak paket içinde bulunur ve *Y* *x*'e aittir.

Bu kurallar ölçeklenebilirliğe fayda sağlar çünkü dışa aktarılmış bir ismi pakete dahil etmek hiçbir zaman pakette sorun yaşanmasına neden olmayacaktır.

Bunlara ek olarak, tek bir tip aynı isimli iki metoda sahip olamaz.*x.M* metodu verildiğinde, yalnızca bir *M*, *x* ile ilişkili olabilir. Tekrar etmek gerekirse, metod yanlıza bir isme atıfta bulanacağı için, bu durum tanımlamayı kolaylaştırır. Ayrıca metod çağrımını da basitleştirir.

**12. Semantik - Anlambilim**

Go semantik açıdan genel olarak C diline benzemektedir. Derlenmiş, statik tipli, *pointer*'ler içeren, prosedürel bir programlama dilidir. Tasarımsal açıdan, C ailesine alışık olan programcılar Go diline de aşinalığı bulunmaktadır. Yeni bir dil geliştirildiğinde, önemli olan noktalardan biri de hedef kitlenin bu dili kolay öğrenebilir olmasıdır. Go'yu C ailesine benzetmek,  Java, Javascript, C gibi diller üzerinde çalışmış olan programcılar için öğrenmesini kolaylaştırmıştır.

Go, C sematiği üzerinde dile çok ufak değişiklikler gerçekleştirmiştir. Bu değişiklikler şu şekilde özetlenebilir:

- *pointer* aritmetiği bulunmamaktadır
- üstü kapalı sayısal dönüşümler yoktur
- dizi boyutları ve aşımı daima kontrol edilir
- tipler için takma isimler bulunmamaktadır (type alias)
- ++ ve -- operatorleri *statement* olarak değerlendirilir *expression* olarak değerlendirilmez. *
- Atama bir *expression* değildir. **


Çevirmen notu:

* Bir programlama dilinde *statement*  bir işlemi ifade eder toplama veya çıkarma gibi, *expression* ise bir değeri ifade eder. *Expression*'lar bir değer üretir. C'nin aksine GO' da bu arttırma veya azaltma operatorleri bir değer üretmezler, yani sonuçları başka bir değere atanamaz.


C dilinde örnek verilmesi gerekirse;


```C
int main()
{
    int a = 10;
    int b = a++;
    printf("%d",b);
    return 0;
}
```

Bu kod bloğu derlendiğinde bir hata ile karşılaşılmayacaktır. Arttırma operatörü operantın sağ yanında bulunduğu için öncelikle 10 değeri b ye atanacak, ardından a bir arttırılıp 11 değerine erişecektir. C'de arttırma operatörü *expression* yani değer üreten ifade olduğu için yazdığımız kod derlenecektir.

Şimdi aynı durumu Go da örneklendirelim.

```go
int main()
{
   number := 1
   increasedNumber:= number++
   fmt.Println(increasedNumber)
}
```

Bu durumda *number++* ifadesi *expression* olmadığı göz önüne alındığında, değer üretmeyeceği için bu işlemin sonucunun bir değişkene atanması derleme zamanı hatasına neden olacaktır.

** Yukarda tanımlandığı üzere *expression*'lar bir değer üretir. C'de atama operatörü bir değer üretirken Go'da atama operatörleri değer üretmez.

C dilinde yazılmış olan aşağıdaki kodu inceleyelim.

```C
int main()
{
   int a = 10;
   int p;
   int t = (p=a);
   printf("%d", t);
    
   return 0;
}
```

Yukarda tanımlanan atama operatörü ile atanan değer geri dönüş değeri olarak t değerine atanır. Bu durumda 10 değerinin çıktısı alınır. Yani C'de atama operatörü *expression* olduğu için bir değer üretir.

Go'da aynı örneği incelersek;


```go
int main()
{
    number := 1
	var number2 int
	t := number = number2
	fmt.Println(t)
}
```

Yukarıdaki kod ise Go'da çalışmaz. Atama operatörü *expression* olmadığı için değer üretmez, değer üretilmediği içinde bu işlemin sonucu başka bir değere atanamaz.


Geleneksel C, C ++ ve hatta Java modellerinden çok farklı olan bazı daha büyük değişiklikler de Go'da bulunmaktadır. Aşağıdaki özelliklere dil kendisi destek vermektedir.

- Eşzamanlılık 
- *Garbage Collection*
- *interface* türleri
- *reflection*
- [Type switches](https://tour.golang.org/methods/16)

Bir sonraki bölümlerde yukarıda bahsedilen *concurrency* ve *garbage collection* özellikleri ile ilgili yazılım mühendisliği bakış açısı ile kısaca bahsedilecektir. Daha detaylı bilgi için [GO!](https://golang.org/)











