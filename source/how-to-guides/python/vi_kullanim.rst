.. _vi_kullanim:

===============================
vi editör kullanımı
===============================

UNIX sistemler için en çok kullanılan metin editörüdür. Aynı anda hem komut işletip hem de yazı yazabiliriz. Üç modu vardır. Komut, yazı, satır modu. Yazı moduna geçmek için "i" tuşuna basarak yazı moduna geçilebilir. 
komut satırına `vi dosyadı.uzantısı` yazılır ve ardından insert moduna geçilir.

- :q komutu : programdan çıkmanızı sağlar.
- :w komutu : içeriği kaydetmemize yarar.
- :wq komutu : yapılan değişiklikleri kaydedip çıkmaya yarar.
- :q! komutu : yapılan değişiklikleri kaydetmeden çıkmak istersek kullanabiliriz.

- komut modunda iken x komutu ile istediğiniz kadar harfi silmeniz mümkündür. Yön tuşlarının görevini h, j, k ve l tuşları da yapabilir.
vi' a yeni başlayanların yaptıkları en sık hatalardan biri yazı modundan komut moduna geçerken _ESC_ tuşuna basmamalarıdır. Böylece normal moda geçmiş oluruz. `_w_' komutu `_ZZ_' komutuna benzer çalışır, hafızada halihazırda bulunan dosyayı, editörden çıkmadan diske kaydeder. Eğer isimsiz bir dosyada üzerinde çalışıyorsanız (örneğin vi telefon yerine sadece vi yazarak editöre girmişseniz) dosyayı sabit diske yazmak için `_w_' satır komutunu kullanmalısınız. 
    :w telefon
   yazıp satır moduna geçerek komutu girmelisiniz


Vi editör için bazı kısayollar:

+-------+-----------------------------------------------------+
| komut | görev                                              |
+=======+=====================================================+
| O     | bulunduğunuz satırın üstüne yeni satır ekler.      |
+-------+-----------------------------------------------------+
| 0     | satır başına atlar.                                |
+-------+-----------------------------------------------------+
| o     | bulunduğunuz satırın üstüne yeni sayır ekler. |
+-------+-----------------------------------------------------+
| komut | görev                                              |
+-------+-----------------------------------------------------+
| o     | bulunduğunuz satırın üstüne yeni sayır ekler. |
+-------+-----------------------------------------------------+
| komut | görev                                              |
+-------+-----------------------------------------------------+
| o     | bulunduğunuz satırın üstüne yeni sayır ekler. |
+-------+-----------------------------------------------------+
| komut | görev                                              |
+-------+-----------------------------------------------------+
| o     | bulunduğunuz satırın üstüne yeni sayır ekler. |
+-------+-----------------------------------------------------+
| komut | görev                                              |
+-------+-----------------------------------------------------+
| o     | bulunduğunuz satırın üstüne yeni sayır ekler. |
+-------+-----------------------------------------------------+
| komut | görev                                              |
+-------+-----------------------------------------------------+
| o     | bulunduğunuz satırın üstüne yeni sayır ekler. |
+-------+-----------------------------------------------------+

