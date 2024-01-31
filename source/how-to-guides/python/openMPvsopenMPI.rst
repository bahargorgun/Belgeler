.. _Python_paralellism:

=====================================================
Python'da Paralel Programlama
=====================================================
Paralelleştirme, bir görevin daha hızlı tamamlanabilmesi için aynı anda birden fazla işlemci veya işlem birimi kullanarak eş zamanlı olarak gerçekleştirilmesi sürecidir.
Birçok modern bilgisayar ve işlemci, çoklu çekirdekli yapılara sahiptir. Bu, birden fazla işlemcinin aynı anda çalışabilmesi anlamına gelir. Paralelleştirme, bu çoklu işlemcilerin eş zamanlı olarak kullanılmasını sağlayarak işlem süresini kısaltabilir.

.. note:: 
    Python'da bulunan GIL (Global Interpreter Lock) birden fazla iş parçacığının paralel olarak ilerlemesini engeller. 
    GIL, iş parçacığı güvenliği ve veri yarışları içeren kodlar yazmayı zorlaştırmak için işe yarar, ancak bunları yaparken Python'da paralel çoklu iş parçacığı ile çalışmayı önler.
   


.. grid:: 4

    .. grid-item-card:: :ref:`Çoklu işleme(Multiprocessing)`
        :text-align: center
    .. grid-item-card::  :ref:`OpenMP`
        :text-align: center
    .. grid-item-card:: :ref:`OPENMPI`
        :text-align: center
    .. grid-item-card:: :ref:`mpi4py`
        :text-align: center
    
  
.. figure:: /assets/python-howto/images/paralellism.png
 :width: 400px
 :align: center

*https://scicomp.aalto.fi/triton/tut/parallel/*

.. _Çoklu işleme(Multiprocessing):

=====================================================
Çoklu İşleme(Multiprocessing)
=====================================================


Birden fazla işlemi aynı anda yapmak anlamına gelir. Yapılan işleri farklı çekirdekler üzerinde paylaştırmak iş yükünü dağıtacağı için daha hızlı çalışmayı sağlar.

.. note:: 

    Çoklu işleme(multiprocessing) ve çoklu iş parçacağı(multithreading) kavramlarını karıştırmamak gerekir.
    Modern CPU'lar birden fazla çekirdeğe sahiptir; işlemleri paralel olarak çalıştırmak istediğimizde bu çekirdekleri 
    kullanırız. Çoklu işleme(multiprocessing) tam olarak burada işimize yarar. Ancak çoklu iş parçacığı, aynı anda birden fazla iş parçacağını(threads) tek işlemde
    çalıştırır ve çekirdeklerin her birinden en üst düzeyde verim almayı amaçlar.




+++++++++
Python Multiprocessing Modülü
+++++++++

**Pool sınıfı**

Pool sınıfı, bir işlem havuzu oluşturmanıza ve birden fazla işlemi paralel olarak çalıştırmanıza olanak tanır.
Temel amacı, verilen görevi birden fazla işlem arasında paylaştırarak işlemci çekirdeklerini etkin bir şekilde kullanmaktır. Bu, hesaplamalı yükleri azaltmak, çoklu işlemi paralel olarak gerçekleştirmek veya aynı işlemi farklı veriler üzerinde paralel olarak çalıştırmak gibi durumlarda faydalıdır.



.. tabs::

    .. tab:: imap() vs imap_unordered()
        
        .. code-block:: python3

            
            import multiprocessing

            def kare_al(x):
                return x * x

            liste = [1, 2, 3, 4, 5]

            # `multiprocessing.Pool()` nesnesi kullanılarak `imap()` fonksiyonunun bir çıktısı alınır.
            # Liste şeklinde alınmasının sebebi imap sonucunda "IMapIterator" nesnesi dönmesidir.

            with multiprocessing.Pool() as pool:
                kareler = list(pool.imap(kare_al, liste))

            print(kareler)



        .. code-block:: output

                [1, 4, 9, 16, 25]

        .. code-block:: python3
            
            import multiprocessing

            def kare_al(x):
                return x * x


            def main():
            # Listedeki her bir sayının karesini alır.
            liste = [1, 2, 3, 4, 5]
            kareler = []

            # `imap_unordered()` fonksiyonunu kullanarak, `kare_al()` fonksiyonunu kullanarak bir listedeki her bir sayının karesini alır.
            with multiprocessing.Pool() as pool:
                kareler = pool.imap_unordered(kare_al, liste)

            # Kareleri yazdırır.
            for result in kareler:
                print(result)

        .. code-block:: output

            [1, 4, 9, 25, 16]

        .. note:: 

             imap ile, sonuçlar hazır olur olmaz Iterator nesnesinden elde edilirken, girdilerin sıralaması korunur. imap_unordered ile sonuçlar, girdilerin sırası ne olursa olsun, hazır olur olmaz döndürülür.
            
        

 
    .. tab:: map() vs map_async()
        .. code-block:: python

            import multiprocessing

            def square(x):
                return x * x

            def callback(result):
                for r in result:
                    print(r, multiprocessing.current_process().name)

            numbers = [1, 2, 3, 4, 5]

            # map_async kullanımı
            print("map_async kullanımı:")
            pool_async = multiprocessing.Pool(processes=2)
            result_async = pool_async.map_async(square, numbers, callback=callback)
            result_async.wait()
            pool_async.close()
            pool_async.join()

            # map kullanımı
            print("\nmap kullanımı:")
            pool_map = multiprocessing.Pool(processes=2)
            result_map = pool_map.map(square, numbers)
            pool_map.close()
            pool_map.join()

            # Sonuçları göster
            print("\nMap_async Sonuçları:", result_async.get())
            print("map Sonuçları:", result_map)

        .. code-block:: output
            
            map_async kullanımı:
            1 MainProcess
            4 MainProcess
            9 MainProcess
            16 MainProcess
            25 MainProcess

            map kullanımı:

            Map_async Sonuçları: [1, 4, 9, 16, 25]
            map Sonuçları: [1, 4, 9, 16, 25]
        
        .. note:: 

            map_async() fonksiyonu bir AsyncResult döndürürken, map() fonksiyonu hedef fonksiyondan dönen değerlerin bir yinelenebilirini döndürür.
            map_async() fonksiyonu geri dönüş değerleri ve hatalar üzerinde geri arama fonksiyonlarını çalıştırabilirken, map() fonksiyonu geri arama(callback) fonksiyonlarını desteklemez.
            map_async() fonksiyonu, hedef görev fonksiyonlarını, görev yürütülürken çağıranın bloklanamayacağı veya bloklanmaması gereken süreç havuzuna vermek için kullanılmalıdır.

            map() işlevi, hedef görev işlevlerini, çağıranın tüm işlev çağrıları tamamlanana kadar engelleyebileceği veya engellemesi gereken işlem havuzuna vermek için kullanılmalıdır.


    .. tab:: apply() vs apply_async()
        
        .. code-block:: python3

            
            import multiprocessing as mp
            import time

            def square(x):
                time.sleep(2)
                return x*x

                result_list = []
                def log_result(result):
                
                    result_list.append(result)

                def apply_async_with_callback():
                    pool = mp.Pool()
                    for i in range(10):
                        pool.apply_async(square, args = (i, ), callback = log_result)
                    pool.close()
                    pool.join()
                    print(result_list)

                result_list_apply= []
                def apply_func():
                
                    pool = mp.Pool()
                    for i in range(10):
                        results = pool.apply(square, args = (i,))
                        result_list_apply.append(results)
                    pool.close()
                    pool.join()
                    print(result_list_apply)

                if __name__ == '__main__':
                    print("Apply_async fonksiyonu için")
                    apply_async_with_callback()
                    print("Apply() fonksiyonu için")
                    apply_func()


        .. code-block:: output

            Apply_async fonksiyonu için : 
            [16, 0, 81, 4, 49, 1, 36, 25, 9, 64]
            Apply() fonksiyonu için :
            [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]

        .. note:: 

            apply() fonksiyonu Sonuç hazır olana kadar bloke eder. Bu bloklar göz önüne alındığında, apply_async() paralel olarak iş yapmak için daha uygundur. Ayrıca, square fonksiyonu yalnızca havuzdaki işçilerden birinde yürütülür.



    .. tab:: Pooling

        .. code-block:: python

            from multiprocessing import Pool
            import time
            import math

            N = 6500000

            def cube(x):
                return math.sqrt(x)

            if __name__ == "__main__":
                # Önce çoklu işleme ile çalıştıralım.
                start_time = time.perf_counter()
                with Pool() as pool:
                  result = pool.map(cube, range(10,N))
                finish_time = time.perf_counter()
                print("Program {} saniyede tamamlandi. - multiprocessing kullanarak".format(finish_time-start_time))
                
                # Sonrasında ise seri bir şekilde çalıştıralım.
                
                start_time = time.perf_counter()
                result = []
                for x in range(10,N):
                    result.append(cube(x))
                finish_time = time.perf_counter()
                print("Program {} saniyede tamamlandi. - seri bir şekilde".format(finish_time-start_time))
                print(-----)

    .. tab:: Pooling SLURM
        
        .. code-block:: bash
        
            #!/bin/bash

            #SBATCH -J pool_ornek2_mpi
            #SBATCH -p hamsi
            #SBATCH -N 1
            #SBATCH -n 4
            #SBATCH -c 7
            #SBATCH -A {kullanıcı_adı}
            #SBATCH -o pool_ornek2

            module purge
            module load centos7.9/comp/gcc/7

            echo "SLURM_NODELIST $SLURM_NODELIST"
            echo "NUMBER OF CORES $SLURM_NTASKS
            srun python3 file3.py

    .. tab:: Pooling Output
        
        .. code-block:: bash
        
            SLURM_NODELIST hamsi27
            NUMBER OF CORES 4
            Program 5.328839018999133 saniyede tamamlandi. - multiprocessing kullanarak
            Program 7.353244483994786 saniyede tamamlandi - seri bir sekilde 
            -----
            Program 9.177415629965253 saniyede tamamlandi. - multiprocessing kullanarak
            Program 5.0710767389973626 saniyede tamamlandi. - seri bir sekilde 
            -----
            Program 7.6777121660416014 saniyede tamamlandi. - multiprocessing kullanarak
            Program 18.970815424981993 saniyede tamamlandi. - seri bir sekilde 
            -----
            Program 12.452059313014615 saniyede tamamlandi. - multiprocessing kullanarak
            Program 21.805212138977367 saniyede tamamlandi. - seri bir sekilde


Pool Ornek 1 açıklaması:

#. ``Pool(processes=2)``  Pool sınıfını kullanarak bir işlem havuzu oluşturuyoruz ve bu işlem havuzunu bir `with` bloğu içinde yönetiyoruz. Bu işlem havuzunda aynı anda en fazla 2 işlem çalıştırılabilir.
#. ``pool.map()`` fonksiyonunun çok uzun iterable'lar için yüksek bellek kullanımına neden olabileceğini unutmayın. Daha iyi verimlilik için açık chunksize seçeneğiyle imap() veya imap_unordered() kullanmayı tercih edebilirsiniz.
#.   ``pool.apply_async(os.getpid, ())`` ifadesi işlem havuzundaki bir işlemi paralel olarak başlatır ve sonucunu almak için AsyncResult nesnesini döndürür. Sonucu elde ettiğinizde, bu nesne, işlemin yürütülmesi sırasında elde edilen işlem kimliğini (PID) içerecektir. Apply() fonksiyonuna göre Paralel işlemler için daha uygundur.
  


.. _OpenMP:

=====================================================
OpenMP
=====================================================

OPENMP bir bilgisayardaki iş parçacığı sisteminde çalışabilen bir kod oluşturmak için kullanılan bir paralel programlama modelidir. 
Paylaşımlı bellek işlemcileri için taşınabilir ve ölçeklenebilir bir programlama modeli sunar. İlk amacı döngüleri parallelleştirmek ve indirgemeler yapmaktır.
İş parçacıkları aynı kaynaklar için rekabet edip aynı verileri güncellediklerinden OpenMP'de yanlış paylaşım, çekişme ve bellek bant genişliği(memory bandwith) sınırlamaları nedeniyle performans sorunları yaşayabilir.

Python'da OpenMP benzeri paralel programlama yeteneklerine erişmek için, CPython yani Python'ın C dili tabanlı uygulaması ile uyumlu olan **"Numba"** gibi kütüphaneleri kullanabilirsiniz. 

Paylaşımlı bellek sistemleri : 

.. image:: https://nyu-cds.github.io/python-mpi/fig/01-shared-mem.png
    :align: center
*https://nyu-cds.github.io/python-mpi/fig/01-shared-mem.png*



=====================================================
Numba
=====================================================


Numba, Python programlarını otomatik olarak hızlandırmak için kullanılan bir Just-In-Time (JIT) derleme kütüphanesidir. Numba, özellikle hesaplamalı kodları hızlandırmak için tasarlanmıştır. Python kodunu C benzeri yüksek hızlı makine koduna dönüştürerek işlemci hızından faydalanmanıza olanak tanır. Numba, özellikle büyük diziler üzerindeki işlemleri hızlandırmak için kullanışlıdır ve genellikle numpy dizileri ile birlikte kullanılır.

.. note:: 
    JIT bir programın yürütülmesi sırasında yapılan derlemedir.
    Yani, yürütme öncesinde değil, çalışma zamanında makine koduna çeviri gerçekleşir.
    Derleme işlemi çalışma zamanında gerçekleştiğinden, geleneksel derleyicilere göre daha fazla optimizasyon imkanı sunar.

.. tabs::

    .. group-tab:: numba_python.py

            .. code-block:: python

                from numba import njit,prange
                import numpy as np
                import time

                #ikinci çalıştırmada @njit decaratorünü kullanarak JIT derleyicisi ile çalışır. 
                @njit(parallel=True)
                def func(x, y):
                    return x + y

                x = 0.01 * np.arange(1000000).reshape((1000,1000))
                y = 0.02 * np.arange(1000000).reshape((1000,1000))

                #Sıradan derlenmiş kod yavaş çalışacaktır.
                start = time.time()
                z1 = func(x, y)
                end = time.time()
                print("İlk fonksiyonun çalışması " + str(end - start) + " saniye aldı.")

                # Numba sayesinde JIT derleyicisini kullanarak daha hızlı çalışan bir kod elde edilir.
                start = time.time()
                z2 = func(x, y)
                end = time.time()
                print("İkinci fonksiyonun çalışması " + str(end - start) + " saniye aldı.")

    .. group-tab:: numba_python.slurm
        
        .. code-block:: bash

            #!/bin/bash
            #SBATCH -p barbun 
            #SBATCH -N 1
            #SBATCH -n 4
            #SBATCH -A {kulllanıcı_adı}
            #SBATCH -o numba_deneme.out
            #SBATCH -J numba_deneme


            eval "$(/truba/sw/centos7.9/lib/anaconda3/2021.11/bin/conda shell.bash hook)"
            conda activate numba_deneme

            srun python numba_deneme.py

    .. group-tab:: numba_python.out

        - Ilk fonksiyonun calismasi 0.9587094783782959 saniye aldi.
        - Ikinci fonksiyonun calisması 0.0076372623443603516 saniye aldi. 
        - Ilk fonksiyonun calismasi 0.9604973793029785 saniye aldi.
        - Ikinci fonksiyonun calisması 0.00766754150390625 saniye aldi. 
        - Ilk fonksiyonun calismasi 0.9557361602783203 saniye aldi.
        - Ikinci fonksiyonun calisması 0.007283210754394531 saniye aldi.
        - Ilk fonksiyonun calismasi 0.9614343643188477 saniye aldi.
        - Ikinci fonksiyonun calisması 0.007330894470214844 saniye aldi.


.. _OpenMPI:

=====================================================
OpenMPI
=====================================================


MPI, paralel hesaplama ortamında işlemler arasındaki iletişimi ve koordinasyonu sağlamak için yaygın olarak kullanılan bir standarttır. OpenMPI ise paralel işlemleri yönetmek için kullanılan bir MPI aracıdır. Bu paralel işlemler paylaşılan bir bellek üzerinde olmadığı için iletişim kurmaları gerekir. MPI protokolleri tam olarak burada işimize yarar.
Bu iletişimde olma hali bazen karmaşıklık yaratabileceği için ağ özelliklerine dikkat edilmelidir. Ağ özellikleri programın performansını etkileyebilir. 
Daha fazla bilgi için `<https://docs.truba.gov.tr/education/openmpi/index.html>`_  adresini ziyaret edebilirsiniz.

.. note:: 
    MPI ve paylaşılan bellek paralelliği aynı uygulama tarafından yapıldığında buna genellikle hibrit paralelleştirme denir. Bu modeli kullanan programlar hem birden fazla görev hem de görev başına birden fazla çekirdek gerektirebilir.

.. _mpi4py:

=====================================================
mpi4py
=====================================================


mpi4py, paralel ve dağıtık hesaplama için Message Passing Interface (MPI) standardına yönelik bir Python kütüphanesidir. 

1. Conda ortamımızı aktifleştirerek başlarız. 
   
   .. code-block:: 

    eval "$(/truba/sw/centos7.9/lib/anaconda3/2021.11/bin/conda shell.bash hook)"

2. Daha sonra mpi4py çalışmalarımız için bir environment oluştururuz.
   
   .. code-block:: 

    conda create --name fast-mpi4py python=3.8 -y

3. Ortamımızı aktifleştiririz.
   
   .. code-block:: 

    conda activate fast-mpi4py

   

4. .. code-block:: 
   
    pip install mpi4py --no-cache-dir
 
   yazarak yükleme işlemimizi tamamlarız.

.. tabs::

    .. group-tab:: hello_mpi.py

        .. code-block:: python

            from mpi4py import MPI
            import sys

            def print_hello(rank, size, name):
                msg = "Hello World, I am process {0} of {1} on {2}.\n"
                sys.stdout.write(msg.format(rank,size,name))

            if __name__ == "__main__":
                size = MPI.COMM_WORLD.Get_size()
                rank = MPI.COMM_WORLD.Get_rank()
                name = MPI.Get_processor_name()

            print_hello(rank,size,name)

    .. group-tab:: hello_mpi.slurm

        .. code-block:: bash

            #!/bin/bash
            #SBATCH -p barbun             #İşin çalıştırılacağı kuyruk adı
            #SBATCH -N 1                  #n adet taskın başlatılacağı node sayısı
            #SBATCH -n 4                  #başlatılacak görev sayısı
            #SBATCH -A {kullanıcı_adı}    #İşi kuyruğa gönderen kullanıcını ismi
            #SBATCH -o hello_mpi          #çıktıların yazılacağı dosya ismi
            #SBATCH -J hello_mpi          #İşin kuyrukta görülecek adı

            eval "$(/truba/sw/centos7.9/lib/anaconda3/2021.11/bin/conda shell.bash hook)"
            conda activate fast-mpi4py
            module purge
            module load centos7.9/lib/openmpi/4.1.5-gcc-7
            mpirun -np 4 python3 hello_mpi.py

    .. group-tab:: hello_mpi2.slurm

        .. code-block:: bash

            #!/bin/bash
            #SBATCH -p barbun             #İşin çalıştırılacağı kuyruk adı
            #SBATCH -N 4                  #n adet taskın başlatılacağı node sayısı
            #SBATCH -n 4                  #başlatılacak görev sayısı
            #SBATCH -A {kullanıcı_adı}    #İşi kuyruğa gönderen kullanıcını ismi
            #SBATCH -o hello_mpi2          #çıktıların yazılacağı dosya ismi
            #SBATCH -J hello_mpi2          #İşin kuyrukta görülecek adı

            eval "$(/truba/sw/centos7.9/lib/anaconda3/2021.11/bin/conda shell.bash hook)"
            conda activate fast-mpi4py
            module purge
            module load centos7.9/lib/openmpi/4.1.5-gcc-7
            mpirun -np 4 python3 hello_mpi.py


    .. group-tab:: hello_mpi.out
        - Hello World, I am process 0 of 4 on barbun44.yonetim.
        - Hello World, I am process 1 of 4 on barbun44.yonetim.
        - Hello World, I am process 2 of 4 on barbun44.yonetim.
        - Hello World, I am process 3 of 4 on barbun44.yonetim.

    .. group-tab:: hello_mpi2.out
        - Hello World, I am process 3 of 4 on barbun51.yonetim.
        - Hello World, I am process 1 of 4 on barbun10.yonetim.
        - Hello World, I am process 2 of 4 on barbun50.yonetim.
        - Hello World, I am process 0 of 4 on barbun9.yonetim.

.. image:: https://blogonparallelcomputing.files.wordpress.com/2017/04/launchmpi1.png
    :width: 600px
    :align: center
*https://blogonparallelcomputing.files.wordpress.com/2017/04/launchmpi1.png*

++++++++++++++++++
OPENMPI VS OPENMP
++++++++++++++++++

- OPENMP ile program yazmak ve hata ayıklamak daha kolaydır.
- OpenMP paylaşımlı belleklerde çalışabilirken OPENMPI hem paylaşımlı hem dağıtık belleklerde çalışabilir.
- MPI Seri sürümden paralel sürüme geçmek için daha fazla programlama değişikliği gerektirir, OPENMP ise programda modifikasyona çok ihtiyaç duymaz.
- OpenMPI'da dağıtık sistemlerde çalışırken düğümler arasındaki iletişim ağı performansı sınırlayabilir. OpenMP'de ise iletişim paylaşımlı bellek içerisinde yapıldığı için, bu iletişim performansı etkilemez.




.. image:: /assets/python-howto/images/integrated.png
    :width: 250px
    :height: 100px
    :align: center

İfadesinin integralini farklı örneklerle alıp, harcanan sürenin farkını aşağıdaki gibi gözlemleyebiliriz.

.. tabs:: 
    .. group-tab:: serial_example.py

        .. code-block:: python

                import math
                from time import perf_counter

                def f(x, y):
                return x * y

                def integrate_2d(f, x_min, x_max, y_min, y_max, n):
                
                dx = (x_max - x_min) / n
                dy = (y_max - y_min) / n

                integral = 0.0
                for i in range(n):
                    for j in range(n):
                    x = x_min + i * dx
                    y = y_min + j * dy
                    integral += f(x,y) * dx * dy

                return integral


                if __name__ == "__main__":
                    start=perf_counter() 
                    integral = integrate_2d(f,0,math.pi,0,math.pi,10000)
                    end=perf_counter()

                    print("Harcanan sure : ", (end-start))
                    print("Integral degeri: ", integral) 

        .. code-block:: output
                
                Harcanan sure :  28.698472655989463
                Integral degeri:  24.347402547472264
    .. group-tab:: serial_with_numba.py

        .. code-block:: python

            import math
            from time import perf_counter
            from numba import njit 

            @njit
            def integrate_2d(x_min, x_max, y_min, y_max, n):

                dx = (x_max-x_min) / float(n)
                dy = (y_max-y_min) / float(n)

                integral = 0.0
                for i in range(n):
                    for j in range(n):
                        x = (x_min + i) * dx
                        y = (y_min + j) * dy
                        integral += x*y * dx * dy

                return integral

            if __name__ == "__main__":
            start=perf_counter()
            integral= integrate_2d(0, math.pi, 0, math.pi, 10000)
            end=perf_counter()

            print("Harcanan sure :" , (end-start))
            print("Integral degeri:", integral)

        .. code-block:: output

            Harcanan sure : 0.6155008749919944
            Integral degeri: 24.347402547472264


    .. group-tab:: threading.py
        
        .. code-block:: python

            import threading
            import math
            from time import perf_counter

            def f(x, y):
                return x * y

            numthreads = 4
            partial_integrals = [None] * numthreads

            def integrate_2d_threading(f, x_min, x_max, y_min, y_max, n, numthreads, threadindex):
                global partial_integrals

                dx = (x_max - x_min) / float(n)
                dy = (y_max - y_min) / float(n)

                integral = 0.0
                workload = n / numthreads
                begin = int(workload * threadindex)
                end = int(workload * (threadindex + 1))
               
                for i in range(begin, end):
                    for j in range(n):
                        x = (x_min + i) * dx
                        y = (y_min + j) * dy
                        integral += f(x, y) * dx * dy

                partial_integrals[threadindex] = integral 
                
                return partial_integrals

            if __name__ == "__main__":
                start = perf_counter()
                threads = []

                for i in range(numthreads):
                    t = threading.Thread(target=integrate_2d_threading, args=(f, 0, math.pi, 0, math.pi, 10000, 4, i))
                    threads.append(t)
                    t.start()

                for t in threads:
                    t.join()

                integral = sum(partial_integrals)
                end = perf_counter()

                print("Harcanan sure:", (end - start))
                print("Integral degeri:", integral)    

        .. code-block:: output

            Harcanan sure: 9.187280416954309
            Integral degeri: 24.347402547473603


    .. group-tab:: multiprocessing.py
        
        .. code-block:: python
           
            import math
            from time import perf_counter
            import multiprocessing
            from multiprocessing import Array


            def f(x, y):
            return x * y
            numprocess = 4
            partial_integrals = Array('d',[0]*numprocess, lock=False)

            def integrate_2d_multiprocess(f, x_min, x_max, y_min, y_max, n,numprocess,processindex):
                global partial_integrals;
                h = math.pi / float(n)


                integral = 0.0
                workload= n/numprocess
                begin = int(workload*processindex)
                end= int(workload*(processindex+1))

            for i in range(begin,end):
                for j in range(n):
                x = (x_min + i) * h
                y = (y_min + j) * h
                integral += f(x,y)*h*h
            partial_integrals[processindex]= integral
            return partial_integrals

            if __name__ == "__main__":
                start=perf_counter()
                process = []
                for i in range(numprocess):
                    p= multiprocessing.Process(target=integrate_2d_multiprocess, args=(f,0,math.pi,0,math.pi,10000,4,i))
                    process.append(p)
                    p.start()

                for p in process:
                    p.join()
                integral = sum(partial_integrals)
                end=perf_counter()

                print("Harcanan sure: ", (end-start))
                print("Integral degeri: ", integral)

        .. code-block:: output
               
                Harcanan sure:  8.090660422982182
                Integral degeri:  24.347402547473603


    .. group-tab::  mpi_example.py
        .. code-block:: python

            import math
            from time import perf_counter
            from mpi4py import MPI

            comm = MPI.COMM_WORLD
            nprocs = comm.Get_size()
            myrank = comm.Get_rank()


            def f(x,y):
            
                return x*y 

            def integrate_2d_mpi(f,x_min, x_max, y_min, y_max, n,nprocs,myrank):
            
                dx = (x_max-x_min) / float(n)
                dy = (y_max-y_min) / float(n)	


                sum = 0.0
                workload= n/nprocs
                begin = int(workload*myrank)
                end= int(workload*(myrank+1))

                for i in range(begin,end):
                    for j in range(n):
                        x = (x_min + i) * dx
                        y = (y_min + j) * dy
                        sum += f(x,y) * dx * dy
                partial_integrals =  sum
                return partial_integrals

                if __name__ == "__main__":
                    start=perf_counter()
                

                p= integrate_2d_mpi(f,0,math.pi,0,math.pi,10000,nprocs,myrank)
                print("nprocs {},my rank {}".format(nprocs,myrank))
                integral = comm.reduce(p, op=MPI.SUM, root=0)
                
                end=perf_counter()
                if myrank == 0:
                print("Harcanan sure: ", (end-start))
                print("Integral degeri: ", integral)


        .. code-block:: output

               Harcanan sure:  5.541638394002803
               Integral degeri:  24.347402547472264

.. tabs:: 
    .. group-tab:: 1D Integral Multiprocessing Örneği

        .. code-block:: python

            import math
            from time import perf_counter

            def trapezoidal_rule(f, a, b, n_sub):
                #n_sub parametresi, toplam alt bölgenin sayısını belirler. Daha yüksek bir n_sub değeri, daha hassas bir integral sonucu elde etmenizi sağlar.
                h = (b - a) / n_sub
                integral = 0.5 * (f(a) + f(b))
                for i in range(1, n_sub):
                    integral += f(a + i * h)
                integral *= h
                return integral

            def f(x):
                return x
            from multiprocessing import Process
            import os

            def integrate_1d_multiprocess(processindex, f, x_min, x_max, n_sub, numprocess):
                width = (x_max-x_min)/numprocess
                sub_interval_start = x_min + processindex*width
                sub_interval_end = x_min + (processindex+1)*width

                result = trapezoidal_rule(f, sub_interval_start, sub_interval_end, n_sub)

                return result

            from multiprocessing import Pool

            #Belirli bir aralıkta (x_min ile x_max), belirli bir fonksiyonu (f(x)) ve belirli bir alt bölge sayısı 
            #(n_sub) ile trapezoidal integral hesaplamak için paralel işlemleri başlatır.
            def main():
                numprocess = 4
                x_min = 0
                x_max = 4
                n_sub = 100000

                start = perf_counter()
                with Pool(processes=numprocess) as p:
                    result = p.starmap(
                        integrate_1d_multiprocess,
                        [(i, f, x_min, x_max, n_sub, numprocess) for i in range(numprocess)],
                    )

                end = perf_counter()

                print("Her bir alt processin sonucu", result)
                print("Sonuc:", sum(result))
                print("Harcanan zaman:", end - start)

            if __name__ == "__main__":
                main()

        
   
    .. group-tab:: (n = 4) SLURM dosyası

        .. code-block:: bash

            #!/bin/bash

            #SBATCH -p barbun
            #SBATCH -N 1
            #SBATCH -n 4
            #SBATCH -c 1
            #SBATCH -J integration1D_serial
            #SBATCH -o multiprocess.out
            #SBATCH --time=00-02:00
            #SBATCH -A bgorgun

            module purge

            srun python3 multiprocess.py

        .. code-block:: output

            Her bir alt processin sonucu [0.4999999999999999, 1.5000000000000004, 2.5, 3.4999999999999996]
            Sonuc: 8.0
            Harcanan zaman: 0.2734628692269325
            Her bir alt processin sonucu [0.4999999999999999, 1.5000000000000004, 2.5, 3.4999999999999996]
            Sonuc: 8.0
            Harcanan zaman: 0.2781460005789995
            Her bir alt processin sonucu [0.4999999999999999, 1.5000000000000004, 2.5, 3.4999999999999996]
            Sonuc: 8.0
            Harcanan zaman: 0.2822933550924063
            Her bir alt processin sonucu [0.4999999999999999, 1.5000000000000004, 2.5, 3.4999999999999996]
            Sonuc: 8.0
            Harcanan zaman: 0.28384475968778133


    .. group-tab:: (n = 1) SLURM Dosyası

        .. code-block:: bash

            #!/bin/bash

            #SBATCH -p barbun
            #SBATCH -N 1
            #SBATCH -n 4
            #SBATCH -c 1
            #SBATCH -J integration1D_serial
            #SBATCH -o multiprocess.out
            #SBATCH --time=00-02:00
            #SBATCH -A bgorgun

            module purge

            srun python3 multiprocess.py

        .. code-block:: output

            Her bir alt processin sonucu [0.4999999999999999, 1.5000000000000004, 2.5, 3.4999999999999996]
            Sonuc: 8.0
            Harcanan zaman: 0.2714084889739752

    .. group-tab:: Multithreading Örneği

        .. code-block:: python3  

            import math
            from time import perf_counter
            import threading

            def trapezoidal_rule(f, a, b, n_sub=1000):
                h = (b - a) / n_sub
                integral = 0.5 * (f(a) + f(b))
                for i in range(1, n_sub):
                    integral += f(a + i * h)
                integral *= h
                return integral

            import os 

            def integrate_1d_thread(threadindex, f, x_min, x_max, n_sub, numthreads, result_lock, result_list):
                width = (x_max - x_min) / numthreads
                sub_interval_start = x_min + threadindex * width
                sub_interval_end = x_min + (threadindex + 1) * width

                local_result = trapezoidal_rule(f, sub_interval_start, sub_interval_end, n_sub)
                print("Task assigned to thread: {}".format(threading.current_thread().name))

                print("ID of process running integrate_1d_thread task : {}".format(os.getpid()))
                with result_lock:
                    result_list.append(local_result)

            def f(x):
                return x        

            def main():
                numthreads = 4
                x_min = 0
                x_max = 4
                n_sub = 10000

                #İş parçacıkları arasında paylaşılan bir kaynağa aynı anda birden çok iş parçacığının müdahalesini önlemek için Lock kullanılır. 


                result_lock = threading.Lock()
                result_list = []
            
                start = perf_counter()

                threads = []
                for i in range(numthreads):
                    thread = threading.Thread(target=integrate_1d_thread, args=(i, f, x_min, x_max, n_sub, numthreads, result_lock, result_list))
                    threads.append(thread)
                    thread.start()

                for thread in threads:
                    thread.join()

                end = perf_counter()

                print("Her bir alt thread'in sonucu", result_list)
                print("Sonuç:", sum(result_list))
                print("Harcanan zaman:", end - start)

            if __name__ == "__main__":
                main()


    .. group-tab:: MT (n=4) SLURM dosyası

        .. code-block:: bash

                #!/bin/bash

            #SBATCH -p barbun
            #SBATCH -N 1
            #SBATCH -n 4
            #SBATCH -c 1
            #SBATCH -J integration1D_serial
            #SBATCH -o multithreads.out
            #SBATCH --time=00-02:00
            #SBATCH -A bgorgun

            module purge

            srun python3 multithreads.py

        .. code-block:: output

                Task assigned to thread: Thread-1
                ID of process running integrate_1d_thread task : 435227
                Task assigned to thread: Thread-2
                ID of process running integrate_1d_thread task : 435227
                Task assigned to thread: Thread-3
                ID of process running integrate_1d_thread task : 435227
                Task assigned to thread: Thread-4
                ID of process running integrate_1d_thread task : 435227
                Her bir alt thread'in sonucu [0.499999999999999, 1.4999999999999996, 2.5, 3.500000000000001]
                Sonuç: 7.999999999999999
                Harcanan zaman: 0.021674709860235453
                Task assigned to thread: Thread-1
                ID of process running integrate_1d_thread task : 435228
                Task assigned to thread: Thread-2
                ID of process running integrate_1d_thread task : 435228
                Task assigned to thread: Thread-3
                ID of process running integrate_1d_thread task : 435228
                Task assigned to thread: Thread-4
                ID of process running integrate_1d_thread task : 435228
                Her bir alt thread'in sonucu [0.499999999999999, 1.4999999999999996, 2.5, 3.500000000000001]
                Sonuç: 7.999999999999999
                Harcanan zaman: 0.024767708964645863
                Task assigned to thread: Thread-1
                ID of process running integrate_1d_thread task : 435226
                Task assigned to thread: Thread-2
                ID of process running integrate_1d_thread task : 435226
                Task assigned to thread: Thread-3
                ID of process running integrate_1d_thread task : 435226
                Task assigned to thread: Thread-4
                ID of process running integrate_1d_thread task : 435226
                Her bir alt thread'in sonucu [0.499999999999999, 1.4999999999999996, 2.5, 3.500000000000001]
                Sonuç: 7.999999999999999
                Harcanan zaman: 0.026993043022230268
                Task assigned to thread: Thread-1
                ID of process running integrate_1d_thread task : 435225
                Task assigned to thread: Thread-2
                ID of process running integrate_1d_thread task : 435225
                Task assigned to thread: Thread-3
                ID of process running integrate_1d_thread task : 435225
                Task assigned to thread: Thread-4
                ID of process running integrate_1d_thread task : 435225
                Her bir alt thread'in sonucu [0.499999999999999, 1.4999999999999996, 2.5, 3.500000000000001]
                Sonuç: 7.999999999999999
                Harcanan zaman: 0.03282354190014303




    .. group-tab:: Multithreading (n=1) SLURM dosyası


        .. code-block:: bash

                #!/bin/bash
                #SBATCH -p barbun
                #SBATCH -N 1
                #SBATCH -n 1
                #SBATCH -c 4
                #SBATCH -J integration1D_threads
                #SBATCH -o multithreads.out
                #SBATCH --time=00-02:00
                #SBATCH -A bgorgun

                module purge

                srun python3 multithreads.py


        .. code-block:: output

                Task assigned to thread: Thread-1
                ID of process running integrate_1d_thread task : 413043
                Task assigned to thread: Thread-2
                ID of process running integrate_1d_thread task : 413043
                Task assigned to thread: Thread-3
                ID of process running integrate_1d_thread task : 413043
                Task assigned to thread: Thread-4
                ID of process running integrate_1d_thread task : 413043
                Her bir alt thread'in sonucu [0.499999999999999, 1.4999999999999996, 2.5, 3.500000000000001]
                Sonuç: 7.999999999999999
                Harcanan zaman: 0.04141010716557503



        

        








            




















