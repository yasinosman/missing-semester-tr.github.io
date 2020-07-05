---
layout: lecture
title: "Metaprogramming"
details: build systems, dependency management, testing, CI
date: 2019-01-27
ready: true
video:
  aspect: 56.25
  id: _Ms1Z4xfqv4
---

Aslında "metaprogramming" derken neyi kastediyoruz? İşin aslı, bu terim kod yazmaktan veya 
verimli çalışmaktan farklı olarak, daha çok
_süreçleri_ ifade etmek adına bulabildiğimiz en müşterek, en kolektif kavramdı .
Bu derste; amacı kodunuzu derlemek, test etmek ve bağımlılıkları yönetmek olan sistemleri inceleyeceğiz.
Bu konu bir öğrenci olarak size önemsiz gibi gözükebilir, Fakat stajyerlik dönemizde veya "gerçek dünyaya" atıldığınız zaman
büyük kod tabanlarıyla karşılaşacaksınız, işte o zaman bu kavram sık sık karşınıza çıkacak.
Bahsetmekte fayda var,
"metaprogramming" aynı zamanda "[programları çalıştıran program](https://en.wikipedia.org/wiki/Metaprogramming)", 
anlamında kullanılabilir. Fakat bu tanım, derste anlatılmak istenen bağlamı tam olarak karşılamamaktadır.



# Build systems

LaTeX'de bir makale yazdığınızı düşünün, dökümanınızı elde etmek için hangi kodları 
çalıştırmanız gerekir? 
Peki ya benchmark yapmak için,
elde ettiğiniz sonuçları grafik haline getirmek için,
grafikleri dökümanınıza eklemek için çalıştırdıklarınız? Ya da aldığınız derste size verilen kodu
derlemek ve testlerini yapmak için çalışırdığınız kodlar?

Çoğu projede, kod içersin ya da içermesin,"build process" denilen bir süreç vardır.
Bilgisayara verdiğiniz girdiden, çıktınızı elde etmek için yapılan operasyonlar 
bütünü de denebilir.Genellikle, bu işlem birçok adım ve branch barındırır.
"Bu grafiği elde etmek için bunu 
çalıştırın, sonuçları elde etmek farklı bir şey çalıştırın,
dökümanın son halini elde etmek için daha farklı bir şey 
çalıştırın" vs . Bu derste gördüğümüz çoğu şey gibi, 
bu baş belası durumla karşılaşan ilk kişi siz değilsiniz, 
şansınıza size yardım edecek pek çok araç mevcut!

Bunlar genellikle "build systems" olarak adlandırılır, ve bunlardan _çok_ vardır.
Hangisini kullanacağınız elinizdeki göreve , programlama dili tercihinize
ve projenin büyüklüğüne göre değişir. Aslında temelde hepsi birbirine benzerdir. Siz _bağımlılık_ sayısını ,

_hedef_, ve _kural_ sayısını, birinden diğerine geçerken tanımlarsınız.
Ayrıca build system'e belirli bir hedef istediğinizi söylersiniz,
bu sistemin işi hedefin tüm geçişli bağımlılıklarını bulmaktır, daha sonra son hedef elde edilene kadar tüm kurallar ara hedeflere uygulanır, nihai amaç son hedefi elde etmektir.
İdeal olarak, "build system" bağımlılıkları değişmeyen hedefler için yeniden çalışmaz,
önceki derlemeden elde ettiği sonuçları tekrar kullanır.

`make` en sık kullanılan build system'lerden birisidir, bunu her "UNIX-based" bilgisayarda yüklenmiş olarak 
bulabilirsiniz. kendine özgü handikapları olduğu halde projelerinizi yönetmekte oldukça iyi iş çıkarmaktadır. 
`make` komutunu çalıştırdığınızda, dizininizde bulunan `Makefile` adındakı dosyaya başvurur. 
Bütün hedefler, onların bağımlılıkları ve kurallar bu dosyada tanımlanmıştır. 
Mesela bir tanesini inceleyelim:

```make
paper.pdf: paper.tex plot-data.png
	pdflatex paper.tex

plot-%.png: %.dat plot.py
	./plot.py -i $*.dat -o $@
```

Bu dosyadaki her direktif, sağ tarafı kullanarak sol tarafı nasıl elde edeceğimize dair gereken kurallardır. 
Başka bir ifadeyle, sağ tarafta tanımlanan şeyler bağımlılıklardır, sol taraftakiler ise hedeftir.
Girintili blok ise, bağımlılıklardan
hedefi elde etmek için gereken sıralı programlardır.  
`make` de , ilk direktif aynı zamanda ilk hedefi de tanımlar. 
Eğer`make`'i argüman olmadan çalıştırırsanız, derleyeceği hedef budur. 
Alternatif olarak, böyle bir şey de çalıştırabilirsiniz:

`make plot-data.png`, `make` şimdi bu hedefi derleyecektir.

Kuraldaki `%` işareti "pattern"dir, yani bir düzendir, soldaki ve sağdaki aynı string ifadeleri eşleştirir. Örneğin, 

hedef olarak `plot-foo.png` istenirse, `make` bağımlılıklara bakacaktır. Bunlar `foo.dat` ve
`plot.py` dosyalarıdır. Şimdi `make` kaynak dizin olmadan çalıştırılırsa ne olur ona bakalım.

```console
$ make
make: *** No rule to make target 'paper.tex', needed by 'paper.pdf'.  Stop.
```
`make` bize `paper.pdf` dosyasını çalıştırmak için, `paper.tex` dosyasına ihtiyacı olduğunu, fakat bunu 

gerçekleştirebilmek için gerekli talimatlara sahip olmadığını söylüyor.

Haydi bu dosyayı oluşturalım!

```console
$ touch paper.tex
$ make
make: *** No rule to make target 'plot-data.png', needed by 'paper.pdf'.  Stop.
```

Hmm, ilginç,  `plot-data.png` dosyasını yapmak için bir kural var, fakat bu

düzen kuralı(pattern rule). Kaynak dosyası mevcut olmadığı için (`foo.dat`), `make`
basitçe bu dosyayı yapamayacağını söylüyor. Şimdi bütün dosyaları yaratmayı deneyelim:

```console
$ cat paper.tex
\documentclass{article}
\usepackage{graphicx}
\begin{document}
\includegraphics[scale=0.65]{plot-data.png}
\end{document}
$ cat plot.py
#!/usr/bin/env python
import matplotlib
import matplotlib.pyplot as plt
import numpy as np
import argparse

parser = argparse.ArgumentParser()
parser.add_argument('-i', type=argparse.FileType('r'))
parser.add_argument('-o')
args = parser.parse_args()

data = np.loadtxt(args.i)
plt.plot(data[:, 0], data[:, 1])
plt.savefig(args.o)
$ cat data.dat
1 1
2 2
3 3
4 4
5 8
```

Şimdi `make`i çalıştırırsak ne olur?

```console
$ make
./plot.py -i data.dat -o plot-data.png
pdflatex paper.tex
... lots of output ...
```

Bakın, bizim için bir PDF dosyası oluşturdu!

Peki`make`i yeniden çalıştırırsak ne olur?

```console
$ make
make: 'paper.pdf' is up to date.
```

Hiçbir şey olmadı! Neden? Çünkü ihtiyaç duymadı. 
Bağımlılıklara baktı ve hedeflerin bu bağımlılıklara uygun olduğunu 
tespit etti, yeni eklenen bir bağımlılık yok bu yüzden yeniden çıktı üretmeye de gerek yok.
Bunu, `paper.tex` dosyasını değiştirerek ve yeniden `make` çalıştırarak kontrol edebiliriz:

```console
$ vim paper.tex
$ make
pdflatex paper.tex
...
```

Dikkat edin `make` , `plot.py` dosyasını _yeniden çalıştırmaya_ ihtiyaç duymadı; çünkü `plot-data.png` 
dosyasının hiçbir bağımlılığı değişmedi!



# Bağımlılık Yönetimi



Daha yüksek seviyelerde, yazılım projeniz, kendileri proje olan bağımlılıklara sahip olabilir.
Projeniz, yüklü programlara (örneğin `python`), sistem paketlerine (örneğin `openssl`),
 ya da yazılım dilinin içinde 
gelen kütüphanelere (örneğin `matplotlib`) bağımlı olabilir.
 Bugünlerde, çoğu bağımlılıklar _repository_ (depo) 
üzerinden erişilebilir durumda, bu depolar fazla miktarda bağımlılıklara
 tek bir yerden erişmemize olanak sağlar. Bu 
sayede bu bağımlılıkları yüklemek kolay ve zahmetsiz bir hal alır.

Mesela `apt` aracılığıyla eriştiğimiz Ubuntu paket depoları(Ubuntu package repositories) bu konu hakkında güzel bir 
örnek teşkil eder , Hakeza Ruby kütüphaneleri için _RubyGems_, 
Python kütüphaneleri için _PyPi_, veya Arch Linux 
paketleri için Arch kullanıcı depoları(Arch User Repository) bağımlılık yönetimi için işlevsel bir kullanım sağlarlar.
Bu depoların çalışma mekanikleri birbirlerinden farklı olduğu için, herhangi biri
hakkında daha fazla detaya inmeyeceğiz. Bunun yerine her depoda ortak bulunan terimlerden bahsedeceğiz. Bunlardan 
ilki _versioning_'dir(versiyonlama).

Çoğu proje, her yayımladığı sürüm için bir _version number_(sürüm numarası)na 
sahiptir. Genellikle bunlar 8.1.3 veya 
64.1.20192004 gibi sayılardır. Bunlar çoğu zaman -fakat her zaman değil- numerik sayılardır.
sürüm numaraları birçok amaca hizmet eder, 
bunlardan en önemlisi yazılımın çalıştığını garanti altına almaktır. 

Örnek vermek gerekirse, benim bir kütüphaneme yeni bir sürüm çıkardığımı hayal edin.
Diyelim ki bu yeni sürümde ben bir metodun ismini değiştirdim,
eğer birisi benim kütüphaneme bağımlılığı olan yazılımını derlemeye çalıştığında yazılım 
hata verecektir. Çünkü çağırdığı metod ben ismini değiştirdiğim için artık mevcut değil! 
Sürüm numarası işte bu noktada çok önemli. 
Yukarıda örneğini verdiğim durumda kullanıcı benim çıkardığım yeni sürümü değil de eski sürümü kullanarak kendi yazılımını başarıyla derleyip çalıştırabilir.

Fakat bu da tam olarak ideal bir kullanım senaryosu değil.
Ya ben kütüphanemin genel arayüzünü _değiştirmeyen_ (Buna "API" diyoruz) bir güvenlik yaması yayınladıysam ve eski sürümlerin hepsi bu yamayı almak zorundaysa ne olacak? 
İşte burada sürüm numarasındaki farklı basamaklar devreye giriyor.
Her basamağın anlamı projeden projeye farklılık 
gösterebilir. Fakat genel standart [_semantik sürümleme_](https://semver.org/) olarak adlandırılır.
Semantik sürümlemede, her sürüm numarası şu şekilde oluşur: **major.minor.patch**.  

Bu sürümleme yönteminde kurallarşu şekildedir:
 <ol>
 <li>Eğer yeni sürüm API'yi değiştirmiyorsa, patch numarasını (son rakam) arttırın.</li>
 <li>Eğer API'ye geriye uyumlu bir <strong>ekleme</strong> yaptıysanız, minör numarasını (ortadaki rakam) arttırın</li>
 <li>Eğer API'ye geriye uyumlu <strong>olmayan bir ekleme</strong>  yaptıysanız, major numarasını (ilk rakam) artırın. </li>
</ol>

Bu yöntemin büyük avantajları vardır. Diyelim ki benim projem senin projene bağımlı,
Benim uygulamamı geliştirdiğim major versiyona gelen son sürümü kullanmam da _sakınca yoktur_. 
Başka bir deyişle, eğer ben senin kütüphanenin `1.3.7` sürümüne bağımlıysam, `1.3.8`, `1.6.1` hatta `1.3.0` gibi sürümleri kullanmam sorun çıkarmayacaktır.
Fakat `2.2.4`  versiyonu muhtemelen problem yaratacaktır çünkü artık major sürüm değişmiştir.

Python dilinin sürüm numaraları buna çok güzel bir örnektir.
Belki farkındasınızdır, Python 2 ve Python 3 olmak üzere iki farklı Python versiyonu vardır,
bu iki sürüm birbiriyle fazla uyumlu değildir ve birinde yazılan kod diğerinde muhtemelen çalışmaz.
Çünkü major sürümleri farklıdır. Yukarıdaki örneğe benzer şekilde Python'un 3.5 sürümünde yazılan kod yüksek ihtimalle 3.7'de sorunsuz çalışacaktır, fakat 2.4 sürümünde çalışma olasılığı hayli düşüktür.

Bağımlılık yönetim sistemleriyle uğraşırken, _lock files_ adında bir dosyayla karşılaşabilirsiniz.
Lock File kısaca _o anki_ bağımlılıklarınızın sürümlerini tutar.
Genelde bağımlılıklarınızı yeni bir versiyona güncellemek için ayrıyetten bir program çalıştırırsınız,
bu yöntemin gereksiz derleme yapmaktan, 
yeniden çalıştırılabilir uygulama dosyaları elde etmek ya da bağımlılığın bizim 
haberimiz olmadan kendini güncellemesini engellemek gibi(ki bu durumun yazılımın çökmesine neden olabilir) pek çok 
sebebi vardır.

Bağımlılık kitleme (dependency locking) kavramının aşırı yapıldığı duruma _vendoring_ denir. 
Vendoring'de bağımlılıklarınızın bütün kodlarını kendi projesine kopyalarsınız, 
bu sizin bağımlılıklar üzerinde tam bir kontrol elde etmenizi sağlar.
Bu sayede kendi arzunuza göre bağımlılıklar üzerinde ekleme-çıkarma yapabilirsiniz.
Fakat bağımlılıklara herhangi bir güncelleme geldiği zaman otomatik güncellemeyeceği için sizin 
ayrıyetten yeni güncellemeleri projenize kendiniz eklemeniz lazım,
bu da fazla bağımlılıklara sahip projelerde yorucu bir işlem haline gelebilir.


# Sürekli Entegrasyon Sistemleri(Continuous Integration Systems)

Halihazırda büyük veya büyümeye devam eden projelerde çalışırken,
sadece kod yazmaktan farklı görevleriniz de olacaktır.
Projenizin dökümantasyonunu güncellemeniz gerekebilir, 
derlenmiş versiyonunu başka bir yere yüklemeniz gerekebilir,
kodu PyPi'de yayınlamanız gerekebilir, test yazmanız gerekebilir vb. 
Hatta Github'da aldığınız her pull request sonrası kodunuzu incelemeniz veya benchmark yapmanız bile gerekebilir.
Bu tür ihtiyaçlar ortaya çıktığında bakmamız gereken şey _sürekli entegrasyondur_(_continuous integration_).


Continuous integration yada kısa adıyla CI, "kodunuz değiştiğinde çalışan işlemleri" karşılamak için bulunmuş komple 
bir terimdir. Farklı tipte continuous integration hizmeti veren pek çok firma vardır,
bu hizmetler genelde açık kaynak projeler için ücretsizdir.
Travis CI, Azure Pipelines ve Github Actions ünlü servislerden bazılarıdır. 
Bu servisler kabaca şu şekilde çalışır: 

Siz deponuza(repository) belirli bir durum, değişiklik olduğunda deponuzun nasıl davranacağına dair bir yönerge 
dosyası eklersiniz. 
Buna dair bir örnek vermek gerekirse, en çok kullanılan yönergelerden birisi "birisi depoya 
ekleme yaptığında, test kodunu çalıştır" yönergesidir.

Birisi depoya ekleme yaptığında bu talimat çalışır, CI servisini aldığınız platform sizin için bir sanal makine oluşturur (duruma göre birden fazla da olabilir) ve sizin 
verdiğiniz yönerge doyasına göre kodu çalıştırır, daha sonra sonuçları sizin için kaydeder.
Bu işleme "test kodu düzgün çalışmazsa beni uyar" tarzında ayarlar ekleyerek sonuçtan haberdar olabilirsiniz.

CI sistemlere örnek olarak şuan bulunduğunuz websitesi örnek verilebilir,
bu websitenin kodları Github Pages'de bulunuyor.
Açık kaynak bir proje olan Jekyll website oluşturma aracını kullanarak yaratılan websitemize, herhangi bir 
ekleme yaptığımızda (ki bunu depomuzdaki `master` branch'ine yapıyoruz ) Github Pages CI araçlarını çalıştırır ve 
bizim için entegrasyon işlemlerini halleder.
Bu, websayfasını güncellemeyi çok kolay bir hale getirir. lokalde değişiklikleri yapın,
daha sonra git aracılığıyla commit edin ve gerisine karışmayın!
CI geri kalan işlemlerin hepsini halledecektir.
 


## Testle ilgili kısa bir açıklama


Çoğu büyük yazılım projesi kendi bünyesinde "test suite" barındırır (tam karşılamasa da Türkçe'ye 'test odası' olarak 
çevirebiliriz). 
Genel olarak test konseptinden bahsettik fakat iş hayatınızda karşılaşabileceğiniz terminolojileri de 
açıklamakta fayda var:
 
 <ol>
<li>Test Suite (Test Odası): Bütün yapılan testler için kullanılan kolektif bir terimdir</li>
<li>Unit Test (Birim Test): Sadece mikro ölçekte, kodun ufak bir kısmını test etme işlemine denir</li>
<li>Integration Test (Entegrasyon Testi): Sistemdeki farklı özelliklerin yada bileşenlerin, _birbirleriyle uyumlu_ çalışıp çalışmadığını kontrol etmek içim makro ölçekte yapılan teste denir.</li>
<li>Regression Test (Regresyon Test): Daha önceye hataya(bug) sebep olmuş bir kodun, düzeltilmiş halinin yeniden hata verip vermediğini test etme işlemine denir</li>
<li>Mocking: Bir fonksiyonun, modülün gereksiz kısımlarını test etmemek için, sahte bir implementasyonla test etmeye denir</li>
</ol>

# Alıştırmalar

1. Çoğu makefile'lar size `clean` adında bir hedef sağlarlar. Bunun amacı `clean` adında dosya üretmek değildir, 
aksine make ile yeniden yapılabilecek dosyaları temizlemektir. Şimdi derleme sürecini geriye almanın bir yolunu 
düşünün. Yukarıdaki `paper.pdf` `Makefile` dosyası için bir `clean` hedefi belirleyin. Hedef
   [phony](https://www.gnu.org/software/make/manual/html_node/Phony-Targets.html) dosyası olacak.
     [`git

   ls-files`](https://git-scm.com/docs/git-ls-files) komutlarını yararlı bulabilirsiniz, incelemenizde fayda var.
   Başka make hedefleri ise bu adreste 

   [listelenmiştir](https://www.gnu.org/software/make/manual/html_node/Standard-Targets.html#Standard-Targets).

2. Bağımlılıklar için sürüm gereksinimlerini belirtmenin çeşitli yollarına göz atın [Rust'ın build system'i](https://doc.rust-lang.org/cargo/reference/specifying-dependencies.html) buna güzel bir örnek olarak verilebilir.

   Çoğu paket depoları benzer syntax'a sahiptir. her birinde kullanılan ortak karakterler vardır
   (_caret, tilde, wildcard, comparison, and multiple gibi_), Bu karakterlerin önem arzettiği bir kullanım senaryosu 
   oluşturun.
 
3. Git kendi başına bir CI servisi gibi kullanılabilir. 
 
   Herhangi bir git deposunda  `.git/hooks` dosyasını bulabilirsiniz. Bu dosyada şu anda aktif olmayan, fakat 
   belirli bir aksiyon olduğunda çalışan scriptler bulunur. Şimdi bir
    
   [`pre-commit(commit öncesi)`](https://git-scm.com/docs/githooks#_pre_commit) hook'u yazın.
    
   Bu hook `make paper.pdf` komutunu çalıştırmayı denesin fakat `make`
   komutu hata verirse commit yapmasın. Bu; derlenemeyen, çalışmayan, uygulamayı bozan commit'lerin yapılmasını 
   engeller.
 
4. [GitHubPages](https://help.github.com/en/actions/automating-your-workflow-with-github-actions) kullanarak 
   otomatik yayınlanan bir sayfa oluşturun.
    
   Deponuza, shell dosyalarını kontrol eden `shellcheck`'i çalıştıran bir [GitHub Action](https://github.com/features/actions) ekleyin.  (mesela [ bu örnek bunu başarmanın yollarından bir tanesidir](https://github.com/marketplace/actions/shellcheck)). Daha sonra `shellcheck`'in çalıştığından emin olun!
 
 
5. [Kendi](https://help.github.com/en/actions/automating-your-workflow-with-github-actions/building-actions)
 
   [`proselint`](http://proselint.com/) çalıştıran Github action'ınızı yazın. 
   [`write-good`](https://github.com/btford/write-good) uygulamasını
 
   kendi deponuzdaki `.md` dosyalarında çalışacak şekilde ayarlayın . Daha sonra çalışıp çalışmadığını test etmek 
   için yazım hatası bulunan bir pull request gönderin.

