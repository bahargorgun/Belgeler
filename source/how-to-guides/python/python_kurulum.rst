.. _Python_kurulum:

=====================================================

Python Kurulumu
=====================================================

Aşağıdaki komutu terminalimizde çalıştırırız. Bu adım TRUBA sunucumuza bağlanmak için gereklidir. 

..  code-block:: bash

  ssh -l [kullanıcı_adı]  levrek1.ulakbim.gov.tr

++++++++++++++++++
TRUBA Modüllerinden Python Kurulumu
++++++++++++++++++

Sunucularda yoğunluğa sebep olmamak ve hesabınızın askıya alınmaması için paket kurulumlarınızı barbun1 arayüzünde gerçekleştirmeniz tavsiye ederiz.

.. code-block:: bash

  ssh barbun1

komutunu terminalinize yazarak barbun1'e geçiş yapabilirsiniz.

.. code-block:: bash

  module av 2>&1 | grep python

komutunu kullanarak TRUBA'da bulunan Python modüllerini listeleyebilirsiniz.

Sizin için uygun olan versiyonu seçtikten sonra

.. code-block:: bash
  
  module load {seçilen modül}

kullanmak istediğiniz modülü yükleyebilirsiniz.

Yüklediğiniz modülün versiyonunu varsayılan Python versiyonunuz yapmak için belirli komutlara ihtiyaç duyarsınız.
Öncelikle izole bir çalışma alanı oluşturarak başlayabilirsiniz. Bunun için Python'da sanal ortam (virtual environment) oluşturabilirsiniz. Bunun için,

.. code-block:: bash

 virtualenv --python={yüklediğiniz modülün pathi} {sanal ortamınızın ismi}

daha sonra bu sanal ortamı kullanabilmek için

.. code-block:: bash
 
 source {sanal ortamınızın ismi}/bin/activate

komutu ile kolaylıkla Python kurulumunuzu gerçekleştirebilirsiniz. Fakat TRUBA'da bulunmayan bir versiyonla çalışmak istediğinizde aşağıdaki adımları izlemeniz gerekmektedir.

++++

1.Kaynak kod indirilir. (Python 3.10.12 versiyonu için gereken komut satırı aşağıda verilmiştir. İndirilmek istenen versiyona göre güncellenebilir.)

.. code-block:: bash

    wget https://www.python.org/ftp/python/3.10.12/Python-3.10.12.tgz

2.İndirdiğiniz .tgz uzantılı dosyayı yapılandırmak için aşağıdaki komutlar sırasıyla uygulanır.

.. code-block:: bash 

  tar xvf Python-3.10.12.tgz
  cd Python-3.10.12 
  ./configure --enable-optimizations --with-ensurepip=install --prefix=/truba/home/{kullanıcı_adı} --with-pydebug 
  make 
  make install


          
.. warning::
    
    Bu işlemleri yaparken PATH girdinize dikkat etmenizi öneririz. PATH ortam değişkenini tanımlamak için .bashrc dosyasından yararlanabilirsiniz.

===================================
.bashrc dosyası
===================================

Kullanıcıların terminali kullanırken hazır olarak çalışmasını istediği bir dosyadır. 
  - Ortam değişkenlerini ayarlama: .bashrc dosyası, PATH, PS1, EDITOR vb. gibi ortam değişkenlerini tanımlayabilir veya değiştirebilir. 

  - Kısayol komutları tanımlama: sık kullanılan komutları veya uzun komut zincirlerini kısayol olarak tanımlayabilir.

  - Alias (takma ad) tanımlama:  sık kullanılan komutlara veya komut dizilerine takma adlar (alias) atayabilir.

.. note::

  .bashrc dosyasını vi/vim editörlerini kullanarak düzenleyebilirsiniz. 
  Python dosyasını kurarken prefix olarak /truba/home/{kullanıcı_adı} kullandığınızı varsayarsak kullanmanız gereken PATH değişkeni aşağıda verilmiştir.

.. code-block:: bash

  export LD\_LIBRARY\_PATH="/truba/home/{kullanıcı_adı}/python3.10/lib:$LD\_LIBRARY\_PATH"
  export PATH="/truba/home/{kullanıcı_adı}/python3.10/bin/:$PATH"
  alias  python="python3"
  tanımlamalarını .bashrc dosyanızın içine yazmalısınız.
   
Son adımda kontrol etmek için `python ---version` komutunu çalıştırdığınızda yüklemek istediğiniz Python versiyonuna ulaşmalısınız.