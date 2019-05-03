Merhabalar, PCMan FTP Server uygulamasının 2.0.7 sürümünde bulunan stack buffer overflow zafiyeti için exploit geliştirmeyi olabildiğince basitleştirerek anlatmaya çalışacağım.

>
Kullanacağım uygulamalar: Immunity Debugger, PCMan FTP Server 2.0.7, VMware

Çalışmayı VMware uygulamasında çalıştırdığım Microsoft Windows XP Professional (Version 2002)’de yapacağım. İşletim sistemi, uygulama sürümleri vs. değiştiğinde ilgili kodların değişebileceğini unutmayın.

Başlamadan önce anlatımda değineceğimiz iki register’ın ne iş yaptığını bilmekte fayda var.

~~~
ESP: Stack pointer. Stack veri yapısının son giren elemanını gösterir.

EIP: Instruction pointer. CPU’nun an itibariyle code segment’i içerisindeki hangi instruction’i çalıştıracağını gösterir.
~~~

Öncelikle PCMan FTP Server uygulamasını sürükleyip Immunity Debugger üzerine bırakarak açıyoruz.

![](https://imguploads.net/images/2018/12/05/168d5b6c6dcbdab5f.png)

Daha sonra imleçle gösterdiğim tuşa veya F9 tuşuna basarak uygulamayı çalıştırıyoruz.

![](https://imguploads.net/images/2018/12/05/2.png)

FTP Server fare imleci ile gösterdiğim bölgede yazan IP üzerinde çalışmakta. NMAP aracı ile port taraması yapıp kullandığı portu görebilirsiniz.

![](https://imguploads.net/images/2018/12/05/3.png)

FTP Server’a bağlanmaya çalıştığımızda kullanıcı adı ve şifre soruyor. Biz de yanıt olarak aşırı veri girip stack’in dolup taşmasını sağlayacağız. Eğer stack taşıyorsa zafiyeti bulduk demektir. Python ile resimdeki kodu yazarak USER parametresi ile 3000 tane ’A’ harfini gönderdik.

![](https://imguploads.net/images/2018/12/05/4.png)
![](https://imguploads.net/images/2018/12/05/5.png)

Ve stack taştığından dolayı program crash oluyor. Resimde gösterdiğim gibi gönderdiğimiz ’A’ harfleri stack’tan taşarak EIP değeri üzerine yazılmış. (A harfinin hexadecimal değeri 41’dir).

![](https://imguploads.net/images/2018/12/05/6.png)

Peki bu EIP değeri bizim gönderdiğimiz kaçıncı ’A’ harfinden etkilendi? Bunu tespit edebilirsek göndereceğimiz veri ile EIP üzerine istediğimiz değerleri yazdırabiliriz. Burada yardımımıza Metasploit-Framework yetişiyor. Resimdeki gibi pattern_create.rb aracını kullanarak yeteri miktarda (3000’de crash olduğu için bu değeri verdik) desen oluşturuyoruz.

![](https://imguploads.net/images/2018/12/05/7.png)

İmleçle gösterdiğim yere veya Ctrl+F2 tuşlarına basarak crash olan programı baştan başlatıyoruz. (Her defasında exploitimizi çalıştırmadan önce baştan başlatmayı unutmayın)

![](https://imguploads.net/images/2018/12/05/8.png)

EIP değerinin değiştiğini görüyoruz. Desenimizde yer alan değerlerin hexadecimal karşılığı buraya yazıldı.

![](https://imguploads.net/images/2018/12/05/9.png)

Bu değeri alıp yine Metasploit-Framework’un pattern_offset.rb aracına vererek stack’in 2003 byte sonrasında taştığını ve EIP’ye yazmaya başadığını öğreniyoruz.

![](https://imguploads.net/images/2018/12/05/10.png)

Daha iyi anlaşılması için resimdeki kodu çalıştıralım. Bakalım 2003 byte sonrasında EIP’yi manuple edebilecek miyiz:

![](https://imguploads.net/images/2018/12/05/11.png)

Stack ’A’ harfi ile dolu olmasına rağmen EIP’ye ’B’ harfinin hexadecimal karşılığı olan 42 değerinin başarıyla yazıldığını görüyoruz.

![](https://imguploads.net/images/2018/12/05/12.png)

Şimdi EIP’ye JMP ESP adresini yazdırarak stack’e atlayacağız. Shellcode’u oraya yazdıracağız. JMP ESP adresi bulmak için e yazan yere tıklıyoruz. (veya ALT+E’ye basabilirsiniz.)

![](https://imguploads.net/images/2018/12/05/13.png)

Açılan penceredeki modüllerden shell32.dll’yi seçtim. Çift tıklayarak açıyoruz. Assembly kodunun olduğu alana sağ tıklayarak search for –> command seçerek “JMP ESP”yi aratıyoruz.

![](https://imguploads.net/images/2018/12/05/14.png)

Aşağıdaki resimde gösterdiğim gibi (mavi ile işaretli olan kısmın solunda) JMP ESP adresini bulmuş olduk.

![](https://imguploads.net/images/2018/12/05/15.png)

Bu adresi işlemcimizin kullandığı sıralama sistemi Little-Endian olduğu için sağdan itibaren başlayarak kodumuza yazıyoruz.

![](https://imguploads.net/images/2018/12/05/16.png)

Sırada badchar tespiti yapmak var. Uygulamalar tarafından engellenen bazı karakterler var. Bunları tespit ederek yazacağımız shellcode’u bu karakterlerden arındıracağız. chars değişkeni içerisine bütün hexadecimal karakterleri (internetten kolayca bulabilirsiniz) yazdırdıktan sonra exploit’imizi tekrar çalıştırıyoruz. (Resimde göstermeyi unutmuşum s.send(’USER’ + buffer + eip + chars + ’\\r\\n’) olarak chars değişkenini eklemeyi unutmayın.)

![](https://imguploads.net/images/2018/12/05/17.png)

Exploit’i çalıştırıp program crash olduktan sonra sağ üstteki ESP değerine sağ tıklayıp ’Follow in Dump’a tıklıyoruz..İlk badchar’ımız en baştan gösteriyor kendini. “\\x00”

![](https://imguploads.net/images/2018/12/05/18.png)

“\\x00” badchar’ını chars değişkenimizden çıkarıp programı ve exploiti tekrar çalıştırıyoruz. ’Follow in Dump’ işlemini tekrardan yaptığımızda bu sefer chars değişkenimizdeki sıralamaya göre ’09’ dan sonra gelen ’0a’ karakterimizin de badchar olduğunu anlıyoruz.

![](https://imguploads.net/images/2018/12/05/19.png)

“\\x0a” karakterini de chars değişkenimizden çıkardıktan sonra işlemleri tekrar uyguluyoruz. “\\x0d” karakterinin de badchar olduğunu farkettik.

![](https://imguploads.net/images/2018/12/05/21.png)

İşlemleri tekrar uyguladığımızda chars değişkenimizin tamamının (“FF” e kadar) stack’e yazıldığını gördük. Demek ki chars değişkenimizin içerisinde başka badchar kalmamış.

![](https://imguploads.net/images/2018/12/05/23.png)

Sırada Metasploit-Framework’ten reverse shell payload’ımızı üretmek var. LHOST’a kendi IP adresimizi, LPORT’a kullanacağımız portu, -b parametresiyle de badchar’ları girip shellcode’umuzu üretiyoruz.

![](https://imguploads.net/images/2018/12/05/24.png)

Üretilen shellcode’u exploit’imizin içerisine yerleştiriyoruz. Shellcode’u stack’e göndermeden önce 30 tane NOP (x86 işlemci mimarisi için “\\x90”) yazdık. Bunun sebebi kodun güvenilirliğini artırmak. No Operation’un kısaltması olan bu NOP değeri işlemcinin bu değeri gördüğünde işlem yapmadan devam etmesini sağlar.

![](https://imguploads.net/images/2018/12/05/25.png)

Netcat ile kullandığımız portu dinlemeye başlıyoruz.

![](https://imguploads.net/images/2018/12/05/26.png)

FTP server uygulamasını ve exploitimizi tekrardan çalıştırdığımızda gönderdiğimiz shellcode stack içerisinde çalıştı ve artık içerdeyiz 🙂

![](https://imguploads.net/images/2018/12/05/27.png)