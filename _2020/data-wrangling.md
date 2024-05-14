---
layout: lecture
title: "Data Wrangling"
date: 2020-01-16
ready: true
video:
  aspect: 56.25
  id: sz_dsktIjt4
---

Daha önce sahip olduğunuz belirli bir türdeki veriyi 
farklı bir türe dönüştürmeye çalışmış mıydınız? 
Bu ders, tamamen sözkonusu bu dönüşümle ilgilidir.  
Özellikle, elinizdeki herhangi bir veriyi 'text' veya 'binary' gibi dilediğiniz herhangi bir formata dönüştürmekle ilgilidir.

Aslında önceki derslerde zaten veri dönüşümü (data wrangling) gerçekleştirmiştik. 
Örneğin `|` operatörünü kullandığımız çoğu zaman bir veri türünü farklı bir veri türüne dönüştürmüş oluruz. 
Örneğin `journalctl | grep -i intel` komutu  sistemdeki içinde 'Intel' geçen bütün kayıtları bulmaktadır 
(Burada küçük/büyük harf önemsizdir).  
Bu işlem ilk bakışta 'data wrangling' olarak görülmese de burada veriler bir türden (sistemdeki bütün kayıtlar) 
farklı bir türe (yalnızca içinde 'intel' geçen kayıtlar) dönüşmektedir. 
Yani 'data wrangling' elimizde hangi araçların bulunduğunu ve bu araçları nasıl kullanabileceğimizi 
bilmekle ilgili bir işlemdir. 

'Data wrangling' yapmak için temel olarak iki şeye gerek duymaktayız. 
Değiştireceğimiz, üzerinde oynayacağımız bir tür veri ve daha sonrasında bu veri ile gerçekleştireceğimiz herhangi bir işlem. 
Bu bağlamda sistem kayıtları oldukça iyi bir örnektir, çünkü çoğu zaman sistem kayıtlarını inceleyerek çeşitli işlemler gerçekleştirmek isteriz ancak bütün sistem kayıtlarını okumak pek mümkün değildir. 
Aşağıda, sistem kayıtlarını inceleyerek sunucumuza ('myserver') giriş yapmaya çalışan kişileri tespit edebilmemizi 
sağlayan bir dizi komut bulunmaktadır:

```bash
ssh myserver journalctl
```
Bu komut bizim istediğimizden çok daha fazla çıktı üretecektir. Bu yüzden aşağıdaki gibi bir komut yazmayı deneyebiliriz:

```bash
ssh myserver journalctl | grep sshd
```

Burada yerel bilgisayarımızda bir remote dosyası yayınlamak için `grep` kullanıyoruz. 
`ssh` oldukça kompleks bir işlem olduğundan bu kavram gelecek derste ayrıntılı bir şekilde incelenecektir.  
Konuya geri dönecek olursak, bu komuttan elde ettiğimiz veri bizim için hala çok fazla 
olduğundan bu veriyi biraz daha özelleştirebiliriz. 

```bash
ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' | less
```

Burada ekstra tırnak işareti kullanmamızın sebebi, sistem kayıtlarımızı bilgisayara aktarıp 
filtrelemek yerine bu filtreleme işlemini uzak sunucuda gerçekleştirip daha sonra bilgisayarımızda 
zaten filtrelenmiş olan veriyi daha kolay bir şekilde işlemektir. 
Komutun sonundaki `less` , bir `pager` oluşturarak komutun çıktısı üzerinde yukarı aşağı kaydırmamızı 
sağlamakta ve okuma işlemimizi kolaylaştırmaktadır. 
Komut satırımızda sistemimizle ilgili hata ayıklama yaparken veriden tasarruf 
etmek adına filtrelenmiş verileri bir dosyaya kaydedip daha sonra bu dosyadan okuma yapabiliriz.

```console
$ ssh myserver 'journalctl | grep sshd | grep "Disconnected from"' > ssh.log
$ less ssh.log
```

Bu komut halen istemediğimiz kadar çok fazla sonuç üretmekte, 
bu sonuçları sadeleştirmek için başvurabileceğimiz oldukça farklı yöntem bulunmakta. 
Örnek olarak `sed` komutunu kullanabiliriz.

`sed` bir çeşit `stream editor` işlevi görmektedir. 
`sed` komutunu kullanırken çoğu zaman içeriğe doğrudan müdahele etmek yerine dosyaları 
nasıl düzenleyeceğimizi belirten komutlar kullanırız. 
Dosyaları düzenlememizi sağlayan bu komutlardan çok fazla bulunmaktadır. 
Ancak bunlardan en yaygın olarak kullanılanı `s` komutudur. Örneğin:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed 's/.*Disconnected from //'
```

Yukarıdaki kod, bir çeşit `regular expression` (düzenli ifade) oluşturmamızı sağlar. 
Regular expressionlar metinleri belirli bir filtreleme işleminden geçirmemizi sağlar.  
`s` komutu şu şekilde yazılır:`s/REGEX/SUBSTITUTION/`; burada` REGEX` kullanmak istediğimiz 
regular expressiona ve `SUBSTITUTION` eşleşen metnin yerine koymak istediğiniz metne karşılık gelmektedir.

## Regular expressions (Düzenli ifadeler)

Regular expressions, günümüzde yaygın olarak kullanılan faydalı bir kavram olduğundan 
incelememiz bizim iyiliğimize olacaktır. 
Yukarıda kullandığımız şeye bakarak başlayalım: `/.*Disconnected from/`. 
Burada da görüldüğü gibi regular expressionlar çoğu zaman (her zaman olmasa da) ` /` ile çevrelenir. 
Regular expression kullanırken çoğu ASCII karakteri normal anlamlarını taşısa da 
"özel" anlamlar barındıran bazı karakterler de bulunmaktadır. 
Aşağıda regular expression oluştururken sıkça kullanılan karakterler verilmiştir:
 
  - `.` yeni satır hariç "herhangi bir tek karakter"
  - `*` önceki eşleşmenin sıfır veya daha fazlası
  - `+` önceki eşleşmenin bir veya daha fazlası
  - `[abc]` `a`, `b` ve `c` karakterlerinden herhangi biri
  - `(RX1 | RX2)` ya `RX1` ya da `RX2` ile eşleşen bir şey
  - `^` satırın başlangıcı
  - `$` satırın sonu

`sed` kullarak regular expression oluştururken bu ifadelerin çoğundan önce 
bir `\` koymamız gerekir. Ya da `-E` de kullanabiliriz.

Bu nedenle, ` /.*Disconnected from / `a baktığımızda herhangi bir sayıda 
karakterle başlayan ve ardından `Disconnected from `
değişmez dizesiyle başlayan herhangi bir metinle eşleştiğini görüyoruz. 
Bizim de ulaşmaya çalıştığımız sonuç buydu. 
Ancak kötü niyetli birisinin kullanıcı adını "Disconnected from" yapıp giriş yapmaya çalışsa ne olurdu?

```
Jan 17 03:13:00 thesquareplanet.com sshd[2631]: Disconnected from invalid user Disconnected from 46.97.239.16 port 55920 [preauth]
```

Regular expressionlardaki `*` ve `+` karakterleri "açgözlü" dür. 
Onlar mümkün olduğunca çok metin ile eşleşmeye çalışırlar. 
Yani, aslında elimizde sadece aşağıdaki sonuç kalmış olurdu:

```
46.97.239.16 port 55920 [preauth]
```

Bu tam olarak istediğimiz sonuç olmayabilir. 
Bazı regular expression uygulamalarında, daha iyi filtreleme için `*` veya `+` yi `?` ile sonlandırabilirsiniz, 
ancak ne yazık ki `sed` bunu desteklemez. Ancak biz de Perl'in komut satırı moduna geçebiliriz, bu da şu yapıyı destekler:

```bash
perl -pe 's/.*?Disconnected from //'
```

Biz eğitimin kalan kısmında `sed` kullanmaya devam edeceğiz, 
çünkü `sed` komutu bu tür işlerde işimizi oldukça kolaylaştıran yaygın bir komuttur. 
`sed` komutu ayrıca bir eşleşme içeren satırları yazdırmada, her kullanıldığında 
yeni bir işlem gerçekleştirmede, bir şeyleri aramada da kullanılabilir. 
`sed` komutu başlı başına ayrı bir konu olsa da, bazı işler için kullanılacak bazı farklı yöntemler de bulabiliriz. 

Kurtulmamız gereken son bir sonek de var. 
Bunu nasıl yapabiliriz? Yalnızca kullanıcı adını takip eden metni eşleştirmek biraz zordur,
özellikle de kullanıcı adı boşluklara ve benzeri semboller içeriyor ise. 
Yapmamız gereken tüm çizgiyi eşleştiren bir komut yazmak:

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user .* [^ ]+ port [0-9]+( \[preauth\])?$//'
```

Bir [regular expression hata ayıklayıcı](https://regex101.com/r/qqbZqh/2) ile neler olup bittiğine bakalım. 
Tamam, başlangıç hala eskisi gibi. Ardından, "user" değişkenlerinden herhangi birini eşleştiriyoruz (günlüklerde iki önek var). 
Ardından, kullanıcı adının bulunduğu herhangi bir karakter dizesiyle eşleşiriz. 
Sonra tek bir kelimeyle eşleşiyoruz (`[^] +`; boşluk olmayan karakterlerin boş olmayan herhangi bir sırası). 
Sonra "port" kelimesinin ardından bir dizi rakam gelir. Sonra muhtemelen 
`[preauth]` soneki ve sonra satırın sonu.

Bu tekniği kullanırsa bir kullanıcının kullanıcı adı `Disconnected from` olsa bile bizi zor duruma sokmayacaktır. 
Nedenini fark edebildiniz mi?


Bununla ilgili bir sorun var ve bu tüm günlüğün boşalması. 
Sonuçta kullanıcı adını korumak istiyoruz.
Bunun için `capture groups` kullanabiliriz. Parantezle çevrili bir regular expression ile eşleşen tüm metinler, 
numaralandırılmış bir capture group (yakalama grubu) içerisinde saklanır. 
Bunlar substituion kısmında (ve bazı motorlarda, desenin kendisinde bile!) `\ 1`,` \ 2`, `\ 3` şeklinde saklanabilir.

```bash
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

Tahmin edebileceğiniz gibi, gerçekten karmaşık regular expressionlar yaratabilirsiniz. 
Örneğin, [bu makale](https://www.regular-expressions.info/email.html) bir e-posta adresi ile nasıl eşleşebileceğinizle ilgili. 
Bu [oldukça zor](https://emailregex.com/) bir işlemdir. 
Ve bu konu hakkında çok sayıda [tartışma](https://stackoverflow.com/questions/201323/how-to-validate-an-email-address-using-a-regular-expression/1917982) yapılmıştır.
Ve insanlar bu regular expressionları [test](https://fightingforalostcause.net/content/misc/2006/compare-email-regex.php) etmiş ve [test matrisleri](https://mathiasbynens.be/demo/url-regex) geliştirmiştir. 
Hatta belirli bir sayının [asal sayı](https://www.noulakaz.net/2007/03/18/a-regular-expression-to-check-for-prime-numbers/ ) 
olup olmadığını belirlemek için bir regular expression (normal ifade) bile yazabilirsiniz.
Regular expressionlar öğrenmesi çok zor olsa da  işimize oldukça fazla yarayabilmektedir.

## Data wrangling'e dönüş
Yukarıdaki bölümün bir sonucu olarak elimizde aşağıdaki gibi bir kod kalıyor:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
```

`sed` komutu, metin enjekte etme  (text injecting) (`i` komutunun yardımıyla), 
satırları yazdırma (`p`komutu ile), dizine göre satır seçme ve daha pek çok ilginç şeyi yapabilir. 
Bu konuda detaylı bilgi için `man sed` i araştırabilirsiniz.

Neyse. Şu an sahip olduklarımız, giriş yapmaya çalışan tüm kullanıcı adlarının bir listesini veriyor. 
Ancak bu bizim bir işimize yaramaz. Ortak olanları arayalım:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
```

`sort` komutu girdileri sıralamamızı sağlar. `uniq -c`, aynı sayıda ardışık satırları tek bir satıra daraltır 
ve bunları  belirlenmiş bir sayıda tekrar eder. 
Muhtemelen bunu sıralamak ve sadece en yaygın girişleri tutmak isteriz. Bunun için de:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
```

`sort -n` komutu sonuçları sayısal olarak sıralar. 
`-k1, 1` komutu 'yalnızca boşluklarla ayrılmış ilk sütuna göre sırala' anlamına gelmektedir. 
`,n` kısmı ise sonuçları n. satıra kadar sıralamamızı sağlar. 
Eğer burada bir değer belirtmeseydik sonuçlar varsayılan olarak satırın sonuna kadar sıralanacaktı.

En az yaygın olanları isteseydik, `tail` yerine `head` kullanabilirdik. 
Ayrıca `sort -r` komutu da ters sırada sıralama yapmamızı sağlar.

Tamam, peki her satıra bir kullanıcı adı vermek isteseydik ne yapabilirdik?

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | awk '{print $2}' | paste -sd,
```

`paste` komutu, satırları (` -s`) belirli bir tek karakterli sınırlayıcı (` -d`) ile birleştirmemizi sağlar. 
Peki ya `awk` ne işe yarar?

## awk -- farklı bir editör

`awk` , genel olarak metin (text stream) işlenirken kullanılan bir programlama dilidir. 
`awk` hakkında öğrenebileceğiniz çok sayıda şey bulunmasına rağmen biz bu bölümde yalnızca temelleri inceleyeceğiz.

İlk olarak `{print $2}` komutunu inceleyelim. 
`awk` programları genellikle metinlerin nasıl olması gerektiğiyle ilgili bir blok ve 
eğer metin istenen şekildeyse ne yapılacağıyla ilgili bir blok içermektedir.
Varsayılan, yani bizim yukarıda kullandığımız pattern (yani metinler için istenen özellik), 
bütün satırlar ile eşleşebilir.
`$0` komutu bütün satırların içeriğini belirlememize, `$1` ve `$n` komutları ise 
boşluklarla ayrılmış `$n`. (n'inci) bölgeyi belirlememize yardımcı olur.
Özetle yukarıdaki komutla demek istediğimiz, her satır için, ikinci alanın içeriğini (yani kullanıcı adını) yazdırmış oluyoruz.

Daha da karmaşık bir örnek vermek gerekirse, aşağıdaki kod `c` ile başlayıp `e` ile biten kullanıcı adlarını yazdırmamızı sağlar.

```bash
 | awk '$1 == 1 && $2 ~ /^c[^ ]*e$/ { print $2 }' | wc -l
```

Buradaki pattern, satırın ilk alanının 1'e eşit olması ve ikinci alanın 
verilen regular expression ile eşleşmesi gerektiğini belirtir. 
İkinci kısımda ise kullanıcı adını yazdırır, daha sonra da çıktı sayısını `wc -l` komutu ile sayarız.

`awk` bir programlama dili olduğuna göre bu işlemi aşağıdaki gibi de yapabiliriz:

```awk
BEGIN { rows = 0 }
$1 == 1 && $2 ~ /^c[^ ]*e$/ { rows += $1 }
END { print rows }
```

Burada `BEGIN` komutu girdilerin başlangıcıyla `END` komutu ise sonuyla ilgilenmektedir. 
Burada her satır başına ilk alandaki sayıyı ekleyip en sonunda bu sayıyı yazdırırız. 
Bu sayede `grep` ve `sed` komutundan bile kurtulabiliriz çünkü `awk` [çok fazla işlem](https://backreference.org/2010/02/10/idiomatic-awk/) gerçekleştirebilir.

## Veri analizi

Regular expressionlar ve çeşitli farklı yöntemler ile elde ettiğimiz veriler ile 
farklı işlemler yapabiliriz. 
Örneğin aşağıdaki komut bütün satırlardaki sayıları toplamamızı sağlar: 

```bash
 | paste -sd+ | bc -l
```
 
Veya daha ayrıntılı olarak:

```bash
echo "2*($(data | paste -sd+))" | bc -l
```

Çok farklı şekilde sonuçlar elde edebiliriz.
[`st`](https://github.com/nferraz/st) komutu da oldukça faydalıdır:

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | awk '{print $1}' | R --slave -e 'x <- scan(file="stdin", quiet=TRUE); summary(x)'
```

R, genellikle veri analizi ve 
[çizim (plotting)](https://ggplot2.tidyverse.org/) için kullanılan garip bir programlama dilidir.
Bu bölümde R hakkında çok fazla ayrıntıya girilmeyecek, 
ancak `summary` komutu bir matris çözerek sonuçları bize sunar.

Ancak sadece basit bir çizim yapmak istiyorsak `gnuplot` da kullanabiliriz.

```bash
ssh myserver journalctl
 | grep sshd
 | grep "Disconnected from"
 | sed -E 's/.*Disconnected from (invalid |authenticating )?user (.*) [^ ]+ port [0-9]+( \[preauth\])?$/\2/'
 | sort | uniq -c
 | sort -nk1,1 | tail -n10
 | gnuplot -p -e 'set boxwidth 0.5; plot "-" using 1:xtic(2) with boxes'
```

## Argüman üretmek için data wrangling

Bazen bir veri listesine dayanarak yüklenecek veya kaldırılacak şeyler bulabiliriz. 
Şimdiye kadar öğrendiğimiz data wrangling yöntemleri ve `xargs` işimizi bu bölümde oldukça kolaylaştıracaktır.

```bash
rustup toolchain list | grep nightly | grep -vE "nightly-x86" | sed 's/-x86.*//' | xargs rustup toolchain uninstall
```

## Binary veriler ile data wrangling yapmak

Çalışmanın bu bölümüne kadar hep metin verileri işlemiştik. 
Ancak bu işlemleri binary veriler için de yapabiliriz. 
Örneğin kendi kameramızdan `ffmpeg` verisi yakalayıp bunu `grayscale` veriye dönüştürüp sıkıştırabilir, 
daha sonra SSH aracılığı ile uzaktaki bir bilgisayara gönderip orada sıkıştırmayı geri açıp, 
videomuzu oraya kopyalayıp görüntüleyebiliriz.

```bash
ffmpeg -loglevel panic -i /dev/video0 -frames 1 -f image2 -
 | convert - -colorspace gray -
 | gzip
 | ssh mymachine 'gzip -d | tee copy.jpg | env DISPLAY=:0 feh -'
```

# Egzersizler

1. Bu interaktif [regular expression örneklerini](https://regexone.com/) tamamlayın.
2. `/usr/share/dict/words` dizininde, en az üç adet 'a' harfi içeren ve 's' ile bitmeyen kelimelerin sayısını bulun. 
Daha sonra bu kelimelerin en yaygın son iki harfinin ne olduğunu tespit edin. 
(Burada `sed` komutunu `y` komutu ile birlikte kullanmak veya `tr` programı size büyük/küçük harf konusunda yardım edebilir.) 
Bu kelimelerde kaç farklı ikili harf kombinasyonu olduğunu hesaplayın ve 
hangi kombinasyonların bu kelimelerde bulunmadığını belirlemeye çalışın.
3. Kendi yerinde değiştirme (in-place substitution) yapmak için 
`sed s/REGEX/SUBSTITUTION/ input.txt > input.txt` gibi bir komut kullanabilirsiniz. 
Ancak bu kötü bir fikirdir. Bu işlemi gerçekleştirmek için `man sed` komutunu araştırın ve bu komutu kullanın.
4. Sisteminizin son 10 boot (önyükleme) süresinin ortalama, medyan ve maksimum değerlerini bulun. 
Bunu yapmak için Linux sistemlerde `journalctl` , macOS'ta ise `log show` komutunu kullanabilirsiniz. 
Linux sistemlerde sonuç aşağıdaki gibi görünebilir:

   ```
   Logs begin at ...
   ```
   and
   ```
   systemd[577]: Startup finished in ...
   ```
   macOS sistemlerde ise, [kaynak](https://eclecticlight.co/2018/03/21/macos-unified-log-3-finding-your-way/):
   ```
   === system boot:
   ```
   and
   ```
   Previous shutdown cause: 5
   ```
5. Son 3 yeniden başlatmanız arasında paylaşılmayan önyükleme iletilerini bulmaya çalışın. 
(burada `journalctl` komutunun `-b` flagini inceleyebilirsiniz.) 
Bu görevi gerçekleştirmek için adım adım ilerleyin. 
Öncelikle yalnızca son üç yeniden başlatmaya dair kayıtları bulun. 
Bu esnada `sed '0,/STRING/D'` komutunu kullanabilirsiniz. 
Ardından bu satırlardaki değişkenleri (zaman belirteçleri gibi) kaldırın. 
Ardından input satırlarını kaldırın ve satırların toplam sayısını bulun (burada `uniq` komutunu kullanın). 
Son olarak da  sayısı 3 olan satırları ortadan kaldırın.
6. Internette bir veri kümesi bulun. [örnek 1](https://stats.wikimedia.org/EN/TablesWikipediaZZ.htm), 
[örnek 2](https://ucr.fbi.gov/crime-in-the-u.s/2016/crime-in-the-u.s.-2016/topic-pages/tables/table-1), 
[örnek 3](https://www.springboard.com/blog/free-public-data-sets-data-science-project/).
Daha sonra bu verileri `curl` kullanarak `fetch` edin. 
Eğer HTML verisi çekiyorsanız [`pup`](https://github.com/EricChiang/pup) , 
JSON verisi çekiyor iseniz [`jq`](https://stedolan.github.io/jq/) size yardımcı olabilir. 
Bu verilerde tek bir komut ile bir sütundaki minimum ve maksimum değerleri 
ve herhangi iki sütun arasındaki toplam farkı bulun.
