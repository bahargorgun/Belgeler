.. _vi_kullanim:

===============================
vi editör kullanımı
===============================

UNIX sistemler için en çok kullanılan metin editörüdür. Aynı anda hem komut işletip hem de yazı yazabiliriz. Üç modu vardır. Komut, yazı, satır modu. Yazı moduna geçmek için "i" tuşuna basarak yazı moduna geçilebilir. 
komut satırına `vi dosyadı.uzantısı` yazılır ve ardından insert moduna geçilir.


.. list-table:: Vi editör için bazı kısayollar:
   :header-rows: 1

   * - Komut
     - Görev
   * - O
     - Bulunduğumuz satırın üstüne yeni satır ekler.
   * - o
     - Bulunduğımuz satırın altına yeni satır ekler.
   * - cw
     - Kelimenin sonuna kadar olan kısmı siler.
   * - x 
     - karakter silme işlemi
   * - dw
     - İmleçten sonraki kelimeleri siler.
   * - d$ 
     - Satır sonuna kadar siler.
   * - u
     - Son yapılan değişiklikler geri alınır.  
   * - r
     - Karakteri değiştirmek için kullanılır.
   * - /
     - Arama yapmak için kullanılır.
   * - e
     - İmlecin bulunduğu kelimenin sonuna gitmek için kullanılır.
  
Dosyadaki işlemleriniz bittiğinde *ESC* tuşuna basılarak aşağıda belirtilen komutlardan istenen komut yazılır.

- :q komutu : programdan çıkmanızı sağlar.
- :w komutu : içeriği kaydetmemize yarar.
- :wq komutu : yapılan değişiklikleri kaydedip çıkmaya yarar.
- :q! komutu : yapılan değişiklikleri kaydetmeden çıkmak istersek kullanabiliriz.