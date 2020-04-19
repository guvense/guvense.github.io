---
layout: post
title: Golang and Errors
image: /img/golang.png
tags: [go, programming, translation]
comments: true
---

[Orijinal makale](https://talks.golang.org/2012/splash.article#TOC_16)

**1. Ön Söz**

Bu yazı Rob Pike tarafından _SPLASH 2012_ konderansında verilen konuşmanın önemli noktalarının derlenmesidir.

Go programlama dili Google altyapısındaki bazı sorunlara çözüm bulmak amacıyla 2007 yılının sonlarında tasarlanmıştır. Günümüz bilgisayar donanımları, çoğunlukla kullanılan C++, Java ve Python uyumlu değildir. _Multicore processors, Networked systems, Massive computation clusters ve web programming_ gibi teknolojilerin getirdiği sorunlar, doğru çözümlerin bulunmasından ziyade günü kurtaracak çözümler sunuluyordu. Ayrıca uygulamaların ölçeklenebilirliği değişti, günümüz sunucu taraflı programları on milyonlarca kod satırından oluşmakta ve bu uygulamalar yüzlerce hatta binlerce programcı tarafından neredeyse hergün güncellenmekteydi. Ne yazık ki, derleme zamanı, gelişmiş _compilation cluster_ 'da bile, dakikalara hatta saatlere kadar uzadı.

Go bu _environment_ içinde çalışmayı daha üretken hale getirmek için tasarlandı. _Built-in concurrency_ ve _garbage collection_ gibi iyi bilinen yönlerinin yanı sıra, GO'nun tasarımı iyi hazırlanmış bağımlılık yönetimi, sistemin büyümesine göre adapte olabilen yazılım mimarisi ve uygulama parçaları arasındaki sağlam sınırlara dayanmaktadır.

Bu makalede amaçlanan, bu gibi sorunların hafif ve uygun bir programlama dili tasarlanırken nasıl ele alındığını açıklamaktır. Google'da karşılaşılan sektöre dayalı sorunlardan örnekler ve açıklamalar aktarılıcaktır.

**2. Giriş**

GO dili, Google tarafından geliştirilen, derlenen, eşzamanlı, _garbage-collected_, statik tipli bir programlama dilidir. Açık kaynak olarak sunulur.

Go programlama dili, verimli, ölçeklenebilir ve üretkendir. Bazın programcılar Go ile çalışırken keyif alırken, diğerleri sığ ve hatta sıkıcı bulabilir.Bu makalede, bu gibi durumların neden çelişki barındırmadığını açıklayacağız. Go, Google' da yazılım geliştirme alanında karşılaşılan sorunlara çözüm bulmak amacıyla tasarlandı. Ve bu süreç sonunda, çığır açan bir araştırma dili olmasa da, büyük yazılım projeleri için mükemmel bir araç olan bir dilin geliştirilmesine yol açtı.

**3. Google' da Go**

Go programlama dili Google tarafından Google'ın problemlerine çözüm bulması amacıyla geliştirildi. Ve Google'ın ciddi anlamda büyük problemlemleri var.

Donanım ve yazılım karmaşık ve büyüktür. C++ ve çoğunlukla Java ve Python ile yazılmış milyonlarca satır yazılım bulunmaktadır. Binlerce mühendis büyük bir parça olan bu yazılım sisteminin yalnızca tek bir parçası üzerinde çalışır. Bu nedenle bu sistem içinde günden güne önemli değişiklikler meydana gelir. [Custom-designed distributed build system](http://google-engtools.blogspot.com/2011/06/build-in-cloud-accessing-source-code.html)
geliştirmeyi ölçeklenebilir kılsada, hala sistem karmaşık ve büyüktür.

Ve elbette, tüm bu yazılım, az sayıda bağımsız, ağa bağlı işlem kümesi olarak kabul edilen zilyonlarca makine üzerinde çalışır.

![DataCenter](/img/datacenter.jpg)

Kısaca Google' da yazılım geliştirme süreci karmaşıktır, yavaş hatta hantal bile olabilir. Fakat etkilidir.

Go projesinin amacı geliştirme sürecinde bu hantallığı ve yavaşlığı azaltmaktır. Böylece bu süreci üretken ve ölçeklenebilir kılmaktır. Bu programlama dili geniş ölçekli yazılım sistemlerini yazan, okuyan, bakımını sağlayan kişiler tarafından ve bu kişiler için tasarlandı.

Bu nedenler, Go' nun amacı yazılım dili tasarımı üzerine araştırma yapmaktan ziyade, tasarımcılar için ve birlikte çalışan mühendisler için çalışma ortamını geliştirmek ve bu çalışma için uygun hale getirmektir.
Go dili programla dili araştırmasından ziyade, yazılım mühendisliği ile ilgilir. Daha açıkça belirtmek gerekirse, yazılım mühendiliğinin hizmet kısmındaki dil tasarımı denilebilir.

Ancak bir dil yazılım mühendisliğine nasıl yardımcı olabilir? Bu makalenin geri kalanı bu sorunun cevabıdır.

**4. Pain points**

Go duyurulduğunda, bazıları tarafından gelişmiş dillerde olması gereken özelliklerin veya metodojilerin bulunmadığı iddia edildi. Go dili bu gibi önemli özellikler olmadan nasıl kayda değer olabilir? Buna cevabımız Go'nun sahip olduğu özelliklerin büyük ölçekli yazılım geliştirmeyi zorlaştıran sorunları ele almasıdır.

- Yavaş derleme
- Controlsüz bağımlılıklar
- Her programcının dilin farklı bir aracını kullanması
- Programın anlaşılabilirliğinde sorun (zor okunan kod, eksik veya hatalı döküman vb.)
- Kod tekrarı
- Güncellemelerin maliyeti
- Versyon karmaşası
- Atomatik araçların yazım zorluğu
- _cross-language builds_

Bir dilin asli özellikleri bu sorunları ele almaz. Yazılım mühendisliğine daha geniş bir bakış açısı gerekiyor ve Go'nun tasarımında bu sorunların çözümlerine odaklanmaya çalıştık.

Basit olarak, örneğin, program yapısının temsilini göz önünde bulundurun. Bazıları Go'nun C türü süslü parantezlerle sağlanan blok yapısına, Python veya Haskell deki gibi boşlukların kullanılması gerektiğini düşünerek itiraz ettiler. Fakat, biz diller arası yapıların neden olduğu derleme ve test hatalarını izleme konusunda geniş deneyime sahibiz. Örneğin, Başka bir dilde, örneğin bir SWIG çağırma yoluyla gömülü olan Python snippet'i, çevredeki kodun girintisindeki bir değişiklik nedeniyle kurnazca ve görünmez bir şekilde kırılmıştır.
Bizim bu duruma karşı bakış açımız, girinti için boşlukların kullanılması küçük programlar için daha sağlıklı olsada, ölçeklenebilir değil ve daha büyük ve karışık kod üzerinde, daha fazla sorun yaratabilir. Güvenlik ve güvenilirlik açısından kolaylıktan vazgeçmek daha iyidir, bu nedenle Go'nun küme parantezle oluşturulmuş blokları vardır.
