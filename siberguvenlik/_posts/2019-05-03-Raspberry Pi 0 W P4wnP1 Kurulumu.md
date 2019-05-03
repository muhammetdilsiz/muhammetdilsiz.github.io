Merhaba, bu yazımızda Raspberry Pi Zero W cihazımıza P4wnP1 kurulumunu gerçekleştireceğiz.

P4wnP1 nedir? P4wnP1 düşük ücretli Raspberry Pi Zero ve Zero W cihazlarında çalıştırabileceğimiz bir USB saldırı platformudur. İçerisinde çeşitli payload‘lar barındırmaktadır.

Proje linki: https://github.com/mame82/P4wnP1/

Öncelikle Raspberry Pi Zero W cihazımız için linkten Raspbian işletim sistemimizi indiriyoruz.

Image dosyasını sdkart‘a yazmak için dd aracını kullanabilirsiniz:

sudo dd if=/image_dosyasinin_adresi/dosya_ismi.img of=/yazacaginiz_disk bs=4M && sync

Yazacağınız diskin ismini sudo fdisk -l komutuyla öğrenebilirsiniz.

Yazma işlemi tamamlandıktan sonra “boot” isminde görünen sdcard‘a giriyoruz ve cmdline.txt dosyasının içindekileri tamamen silerek aşağıdakileri yazıp kaydediyoruz:

dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=/dev/mmcblk0p2 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait modules-load=dwc2,g_ether quiet init=/usr/lib/raspi-config/init_resize.sh

Aynı dizin içerisine ssh isminde boş bir dosya açıyoruz (SSH‘ın aktif edilmesi için). Ve config.txt dosyasının sonuna “dtoverlay=dwc2” ekleyerek Raspberry‘mize Sdcard‘ı takıp güç veriyoruz. Kullanmak için ekrana sahip olanlar oradan internete bağlayabilirler. Olmayanlar için sd kartımızı çıkarmadan bu komutları düzenleyerek;

network={
ssid=”ag_ismi_buraya”
psk=”sifre_buraya”
proto=RSN
key_mgmt=WPA-PSK
pairwise=CCMP
auth_alg=OPEN
}

#Not(Ağınıza göre bu bilgileri kullanarak düzenleme yapabilirsiniz):
#proto : WPA2 için RSN , WPA1 için WPA
#key_mgmt : kişisel ağ için WPA-PSK , kurumsal ağ için WPA-EAP
#pairwise WPA2 için CCMP ve WPA1 için TKIP
#auth_alg : otomatik bağlanma OPEN, diğer ayarlar için LEAP ve SHARED

rootfs‘deki /etc/wpa_supplicant dizini içerisinde bulunan “wpa_supplicant.conf” ismindeki dosyaya ekleyerek internet bağlantımızı gerçekleştirebiliriz.

Netdiscover veya arp-scan ile IP adresini keşfedip “ssh pi@ip_adresi” şeklinde bağlantımızı gerçekleştiriyoruz. Şifremiz ise “raspberry”

sudo apt-get update
sudo apt-get install git
git clone https://github.com/mame82/P4wnP1.git
cd P4wnP1 && ./install.sh

komutlarını sırasıyla çalıştırdığımızda (biraz uzun sürebilir) cihazımız hazır hale geliyor.

Güç girişinden kullandığımız cihazı artık USB‘den hedef bilgisayara takıyoruz. Driver yüklendiğinde resimdeki gibi bir ağ açılıyor, ona bağlanıyoruz. Devamını 2. bilgisayarım olmadığı için telefon üzerinden “Termius” isimli programı kullanarak gerçekleştirdim.

Ağ Adı: P4wnP1
Şifre: MaMe82-P4wnP1

![](https://i1.wp.com/imguploads.net/images/2019/01/10/WhatsApp-Image-2019-01-10-at-21.06.58.jpg)

Ağa bağlandıktan sonra ssh bağlantımızı gerçekleştireceğiz. “ssh pi@172.16.0.1” komutu ve “raspberry” şifresini giriyoruz. Cihazımıza bağlanmış olduk.

![](https://i0.wp.com/imguploads.net/images/2019/01/10/WhatsApp-Image-2019-01-10-at-21.06.581.jpg)

Biz network_only payload‘ı ile çalıştık. Siz burada P4wnP1 klasörüne girip sudo nano setup.cfg komutu ile dosyayı açıp en alta indiğinizde diğer payloadları görebilirsiniz.
Merak edenler için ilgili kısmı burada paylaşıyorum.

>
=====================
# Payload selection
 =====================

PAYLOAD=network_only.txt

#PAYLOAD=wifi_covert_channel/hid_only_delivery64.txt # WiFi covert channel (HID only delivery), insert P4wnP1 to target, press NUMLOCK rapidly to infect … remove P4wnP1 and provided it with Power, lock in via WiFi and use the C2 server for the covert channel
#
#PAYLOAD=wifi_covert_channel/hid_only_delivery32.txt # 32bit version untested
#
#PAYLOAD=wifi_covert_channel/hid_only_delivery64_bt_only.txt # WiFi covert channel version which use Bluetooth instead of a WiFi AP to provide C2 server access (no visible P4wnP1 WiFi, at all)
#
#PAYLOAD=nexmon/karma.txt # Experimental Rogue AP in Karma mode using Nexmon (seemoo-lab) firmware for Monitor/Injection (+ MaMe82 KARMA firmware mod) and Responder
#
#PAYLOAD=nexmon/karma_bt_upstream.txt
#
#PAYLOAD=hid_mouse.txt # HID mouse demo: Shows different ways of positioning the mouse pointer, using P4wnP1‘s MouseScript languag
#
#PAYLOAD=hid_backdoor_remote.txt # AutoSSH “reachback” version of hid_backdoor (see payload comments for details)
#
#PAYLOAD=wifi_connect.txt
#
#PAYLOAD=stickykey/trigger.txt # Backdoor Windows LockScreen with SYSTEM shell, triggered by NUMLOCK, trigger SCROLLLOCK to revert the changes
#
#PAYLOAD=hakin9_tutorial/payload.txt # steals stored plain credentials of Internet Explorer or Edge and saves them to USB flash drive (for hakin9 tutorial)
#
#PAYLOAD=Win10_LockPicker.txt # Steals NetNTLMv2 hash from locked Window machine, attempts to crack the hash and enters the plain password to unlock the machin on success
#
#PAYLOAD=hid_backdoor.txt # under (heavy) development
#
#PAYLOAD=hid_frontdoor.txt # HID covert channel demo: Triggers P4wnP1 covert channel console by pressing NUMLOCK 5 times on target (Windows)
#
#PAYLOAD=hid_keyboard.txt # HID keyboard demo: Waits till target installed keyboard driver and writes “Keyboard is running” to notepad
#
#PAYLOAD=hid_keyboard2.txt # HID keyboard demo: triggered by CAPS-, NUM- or SCROLL-LOCK interaction on target
<
Kullanmak istediğiniz payload‘ın başındaki “#” işaretini kaldırıp mevcut kullanılan payload‘ın başına “#” ekliyoruz. Sonrasında sudo shutdown now komutu ile Raspberry‘mizi güvenli biçimde kapatıp bilgisayarımıza tekrardan takıyoruz. Kolay gelsin 🙂

>
*USB Kablonuzun OTG olmasına dikkat edin.
>
**Diğer Raspberry versiyonları için PoisonTap (https://samy.pl/poisontap/)‘ı deneyebilirsiniz.
