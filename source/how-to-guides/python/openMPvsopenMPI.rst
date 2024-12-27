.. _Python_paralellism:

=====================================================
Python'da Paralel Programlama
=====================================================
Paralelleştirme, performansı artırmak ve hesaplamaları hızlandırmak amacıyla, birden fazla görevin aynı anda yürütülmesi sürecine denir. Büyük bir görevin, birden fazla işlemci, çekirdek veya farklı makinelerde eşzamanlı olarak çalıştırılabilecek daha küçük ve bağımsız alt görevlere bölünmesini içerir. 
Bu sayede, işlemler daha verimli hale gelir ve toplam işlem süresi önemli ölçüde kısalabilir.

.. warning:: 

    Python'da bulunan GIL (Global Interpreter Lock) birden fazla iş parçacığının paralel olarak ilerlemesini engeller. 
    GIL, iş parçacığı güvenliği ve veri yarışları içeren kodlar yazmayı önlemek için işe yarar, ancak bunları yaparken Python'da paralel iş parçacıkları halinde çalışmayı önler. Bu yüzden Python'da paralel programlamadan yararlanırken belirli kütüphanelere ihtiyaç duyarız.
    


.. grid:: 3

    .. grid-item-card:: :ref:`Çoklu işleme(Multiprocessing)`
        :text-align: center
    .. grid-item-card::  :ref:`OpenMP`
        :text-align: center
    .. grid-item-card:: :ref:`OpenMPI`
        :text-align: center
   
    
  
.. figure:: /assets/python-howto/images/paralellism.png
 :width: 400px
 :align: center

`kaynakça <https://scicomp.aalto.fi/triton/tut/parallel/>`_

.. _Çoklu işleme(Multiprocessing):

===============================
Çoklu İşleme(Multiprocessing)
===============================


Multiprocessing (çoklu işlem), birden fazla işlemci çekirdeğinin veya işlemcinin aynı anda çalışabilmesi için kullanılan bir programlama yöntemidir. Bu paylaştırma işlemi, iş yükünü dağıtarak daha hızlı 
çalışmayı sağlayabilir.

..
    .. note:: 

    **Çoklu işlem (multiprocessing)** ve **çoklu iş parçacağı (multithreading)** kavramlarını karıştırmamak gerekir.
    **Multiprocessing** (Çoklu İşlem), birden fazla bağımsız işlem başlatarak çalışır. Her işlem, kendi bellek alanına ve kendi Python interpreter'ına sahiptir. Yani ana process, birden fazla processe bölünerek gerçekleşir. Bu yöntem, CPU-bound (işlemci ağırlıklı) işlemler için uygundur.
    **Multithreading** (Çoklu İş Parçacığı), aynı işlem içinde birden fazla iş parçacığı (thread) çalıştırır. Tüm iş parçacıkları, aynı bellek alanını paylaşır ve aynı Python interpreter'ını kullanır. Bu yöntem, I/O-bound (girdi/çıktı ağırlıklı) işlemler için uygundur.

===============================
Python Multiprocessing Modülü
===============================

**Pool sınıfı**

Pool sınıfı, belirtilen sayıda belirtilen sayıda işlemciden oluşan bir işlem havuzu oluşturulmasıyla başlar.
Temel amacı, verilen görevi bu havuz içersinde paylaştırarak kaynakları etkin bir şekilde kullanmaktır.

.. tabs::
    .. tab:: Pooling (pooling_pid.py)

        .. code-block:: python

            from multiprocessing import Pool, current_process
            import math

            N = 6

            def process_id(x):
                # Hangi process ID'de çalıştığını gösterelim.
                process_id = current_process().pid
                print(f"Process ID: {process_id}, İşleniyor: {x}")
                return process_id

            if __name__ == "__main__":
             
                print("Multiprocessing ile yurutme basladi.") 

                with Pool(processes=4) as pool:  # Process sayısı 4 olarak belirledik.
                
                    result = pool.map(process_id, range(1, N))
            
                print("Multiprocessing ile yürütme sona erdi.")
                print("Seri yürütme başladı.")

                result = []
                for x in range(1, N):
                    result.append(process_id(x))
            
                print("Seri yürütme sona erdi.") 

    .. tab:: Pooling.sh
        
        .. code-block:: bash
        
            #!/bin/bash

            #SBATCH -p barbun
            #SBATCH -N 1
            #SBATCH -n 1
            #SBATCH -c 4
            #SBATCH -J pooling_pid
            #SBATCH -o pooling_pid.out
            #SBATCH --time=00-05:00
            #SBATCH -A kullanici_adi

            eval "$(/truba/sw/centos7.9/lib/anaconda3/2023.03/bin/conda shell.bash hook)"

            echo "SLURM_NODELIST $SLURM_NODELIST"
            echo "NUMBER OF CORES $SLURM_NTASKS"
            
            srun python3 pooling_pid.py

    .. tab:: Pooling Output
        
        .. code-block:: bash
        
            SLURM_NODELIST barbun52
            NUMBER OF CORES 1
            Multiprocessing ile yurutme basladi.
            Process ID: 74280, İşleniyor: 2
            Process ID: 74280, İşleniyor: 5
            Process ID: 74282, İşleniyor: 3
            Process ID: 74281, İşleniyor: 4
            Process ID: 74279, İşleniyor: 1
            Multiprocessing ile yürütme sona erdi.
            Seri yürütme başladı.
            Process ID: 74278, İşleniyor: 1
            Process ID: 74278, İşleniyor: 2
            Process ID: 74278, İşleniyor: 3
            Process ID: 74278, İşleniyor: 4
            Process ID: 74278, İşleniyor: 5
            Seri yürütme sona erdi.



#. ``Pool(processes=4)`` ifadesi ile 4 işlemli bir havuz (pool) oluşturulur. Bu işlem havuzu bir `with` bloğu içinde yönetilir. N = 6 belirleyerek (1,2,3,4,5) sayılarının processlere dağılmasını sağlıyoruz. 2 ve 5 sayılarının aynı processlerde yer almasının sebebi, paralelleştirme işlemi başlatılırken 4 adet process belirlememizdir. 


.. code-block::
    
    sbatch pooling.sh

sbatch komutunu kullanarak işi kuyruğa gönderebilirsiniz.

#. ``pool.map()`` fonksiyonunun çok uzun veri yapıları için yüksek bellek kullanımına neden olabileceğini unutmayın. Daha iyi verimlilik için chunksize seçeneğiyle imap() kullanmayı tercih edebilirsiniz.
  


.. tabs::

    .. tab:: imap() vs imap_unordered()
        
        .. code-block:: python

            
            from multiprocessing import Pool
            import time

            def square(x):
                # Bir işlemin süresini simüle etmek için gecikme ekleyebiliriz.
                time.sleep(0.5)
                return x * x

            if __name__ == "__main__":
                numbers = [1, 2, 3, 4, 5]

                # Multiprocessing ile `imap` kullanarak
                with Pool(processes=4) as pool:
                    print("imap() sonuçları:")
                    result_imap = pool.imap(square, numbers)
                    for value in result_imap:
                        print(value)  # Sonuçlar sıralı olarak döner ve tek tek alınır.
                    
                    print("\n")

                # Multiprocessing ile `imap_unordered` kullanarak
                with Pool(processes=4) as pool:
                    print("imap_unordered() sonuçları:")
                    result_imap_unordered = pool.imap_unordered(square, numbers)
                    for value in result_imap_unordered:
                        print(value)  # Sonuçlar sıraya göre döner (garanti edilmez)




        .. code-block:: output

                imap() sonuçları:
                1
                4
                9
                16
                25


                imap_unordered() sonuçları:
                4
                1
                16
                9
                25

    

        .. note:: 

            imap() ile, sonuçlar hazır olur olmaz Iterator nesnesinden elde edilirken girdilerin sıralaması korunur. imap_unordered() ile sonuçlar, girdilerin sırası ne olursa olsun, sonuç hazır olur olmaz döndürülür.
            imap() ve imap_unordered() fonksiyonları map() fonksiyonlarına kıyasla çok büyük veri setleri ile çalışırken daha verimli olabilir çünkü veriler bir seferde yüklenmek yerine parça parça işlenir.
        

 
    .. tab:: map() vs map_async()

        .. code-block:: python

            import multiprocessing

            from multiprocessing import Pool, current_process
            import math

            def square(x):
                # Hangi process ID'de calistigini gösterir.
                process_id = current_process().pid
                result = x * x
                print(f"Process ID: {process_id}, İ?~_leniyor: {x}, Sonuç: {result}")
                return result
            #map_async fonksiyonu callback fonksiyonu ile kullanılmaya elverişlidir. 
            def callback(result):
                #callback fonksiyonu ana süreçte çalışır.
                print("Callback fonksiyonunda sonuçlar:")
                for r in result:
                    print(r, current_process().pid)

            numbers = [1, 2, 3, 4]

            # map() kullanımı
            with Pool(processes=4) as pool:
                print("Map() fonksiyonu çıktıları")
                result = pool.map(square, numbers)
                print("Sonuc tipi :", type(result))
                

            # map_async() kullanımı
            with Pool(processes=4) as pool:
                print("Map_async() fonksiyonu çıktıları")
                result_async = pool.map_async(square, numbers, callback=callback, chunksize=1)
                
                #Map_async fonksiyonu sonuç nesnesini üretmek için tüm işlemin tamamlanmasını bekler.
                result_async.wait()
                print("Sonuc tipi :", type(result_async))


            pool.close()
            pool.join()

        .. code-block:: output
            
            Map() fonksiyonu çıktıları
            Process ID: 152936, İşleniyor: 1, Sonuç: 1
            Process ID: 152937, İşleniyor: 2, Sonuç: 4
            Process ID: 152938, İşleniyor: 3, Sonuç: 9
            Process ID: 152939, İşleniyor: 4, Sonuç: 16
            Sonuc tipi:  <class 'list'>
            Map_async() fonksiyonu çıktıları
            Process ID: 152943, İşleniyor: 1, Sonuç: 1
            Process ID: 152944, İşleniyor: 2, Sonuç: 4
            Process ID: 152946, İşleniyor: 4, Sonuç: 16
            Process ID: 152945, İşleniyor: 3, Sonuç: 9
            Callback fonksiyonunda sonuçlar:
            1 152929
            4 152929
            9 152929
            16 152929
            Sonuc tipi:  <class 'multiprocessing.pool.MapResult'>

        .. note:: 

            map_async() fonksiyonu bir AsyncResult döndürürken, map() fonksiyonu hedef fonksiyondan dönen değerlerin bir yinelenebilirini(recursive) döndürür.
            map_async() fonksiyonu geri dönüş değerleri ve hatalar üzerinde geri arama fonksiyonlarını çalıştırabilirken, map() fonksiyonu geri arama(callback) fonksiyonlarını desteklemez.
            map_async() fonksiyonu, hedef görev fonksiyonlarını, görev yürütülürken çağıranın bloklanmaması gereken süreç havuzunda işlem yapmak için kullanılmalıdır.
            #callbacki de açıkla, nasıl çalışıyor

            map() işlevi, hedef görev işlevlerini işlem havuzuna vererek, çağıranın tüm işlev çağrıları tamamlanana kadar engelleyebileceği veya engellemesi gereken işlemleri gerçekleştirmesini sağlar.

    .. tab:: apply() vs apply_async()
        
        .. code-block:: python

            
            import multiprocessing as mp
            import time

            def square(x):
                #birden fazla argümanlı bir method yaz
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

            apply() fonksiyonu Sonuç hazır olana kadar bloke eder. Bu bloklar göz önüne alındığında, apply_async() paralel olarak iş yapmak için daha uygundur.



.. _OpenMP:

=======
OpenMP
=======
OpenMP bir bilgisayardaki iş paylaştırma sisteminde çalışabilen bir kod oluşturmak için kullanılan bir paralel programlama modelidir. 
Aynı belleği paylaşan işlemciler için taşınabilir ve ölçeklenebilir bir programlama modeli sunar. 

OpenBLAS, MKL, BLAS, LAPACK ve NUMBA gibi HPC sistemlerinde sıklıkla kullanılan kütüphaneler OpenMP'den yararlanarak işlemleri paralelleştirirler.
Bu paralelleştirme işlemini yapmak için, OMP_NUM_THREADS, MKL_NUM_THREADS, OPENBLAS_NUM_THREADS, NUMBA_NUM_THREADS gibi parametreler kullanılabilir.


Paylaşımlı bellek sistemleri 

.. image:: https://nyu-cds.github.io/python-mpi/fig/01-shared-mem.png
   :align: center

`kaynakca <https://nyu-cds.github.io/python-mpi/fig/01-shared-mem.png/>`_



======
Numba
======


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
                import numba as nb
                import time
                @njit(parallel=True)
                def parallel_sum(arr):
                    total = 0.0
                    for i in nb.prange(len(arr)):
                        total += arr[i]
                    return total


                def serial_sum(arr):
                    total = 0.0
                    for i in range(len(arr)):  # nb.prange yerine range kullanıyoruz
                        total += arr[i]
                    return total


                if __name__ == '__main__':
                    data = np.random.rand(100000000)
                    start = time.time()
                
                    result = parallel_sum(data)
                
                
                    print("Sum:", result)
                    end = time.time()
                    print("Ilk fonksiyon " , str(end - start) , " saniye aldı.")

                    start2 = time.time()
                    result_serial = serial_sum(data)
                
                    print("Sum:", result_serial)
                    end2 = time.time()
                    print("Ikinci fonksiyon " , str(end2 - start2) , " saniye aldı.")

    .. group-tab:: numba_python.slurm
        
        .. code-block:: bash

            #!/bin/bash
            #SBATCH -p barbun 
            #SBATCH -N 1
            #SBATCH -n 1
            #SBATCH -c 4
            #SBATCH -A {kulllanıcı_adı}
            #SBATCH -o numba_çıktı.out
            #SBATCH -J numba__work


            eval "$(/truba/sw/centos7.9/lib/anaconda3/2021.11/bin/conda shell.bash hook)"
            conda activate numba_env

            export NUMBA_NUM_THREADS=4
            srun python numba.py

        .. code-block:: bash

            sbatch numba_python.slurm

            işi kuyruğua bu şekilde gönderebilirsiniz.   

    .. group-tab:: numba_python.out

           - Sum: 49998201.82653082
           -  Ilk fonksiyon  **0.5291917324066162**  saniye aldı.
           - Sum: 49998201.826489806
           - Ikinci fonksiyon  **10.636533260345459**  saniye aldı.


.. _OpenMPI:

=========
OpenMPI
=========
MPI, paralel hesaplama ortamında işlemler arasındaki iletişimi ve koordinasyonu sağlamak için yaygın olarak kullanılan bir standarttır. OpenMPI ise paralel işlemleri yönetmek için kullanılan bir MPI aracıdır. Bu paralel işlemler paylaşılan bir bellek üzerinde olmadığı için iletişim kurmaları gerekir. MPI protokolleri tam olarak burada işimize yarar.
Bu dağıtık iletişimde olma hali bazen karmaşıklık yaratabileceği için ağ özelliklerine dikkat edilmelidir. Ağ özellikleri programın performansını etkileyebilir. 
Daha fazla bilgi için `<https://docs.truba.gov.tr/education/openmpi/index.html>`_  adresini ziyaret edebilirsiniz.

.. note:: 
    MPI ve paylaşılan bellek paralelliği aynı uygulama tarafından yapıldığında buna genellikle hibrit paralelleştirme denir. Bu modeli kullanan programlar hem birden fazla görev hem de görev başına birden fazla çekirdek gerektirebilir.

.. _mpi4py:

=======
mpi4py
=======
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

Aşağıda basit bir MPI örneği görebilirsiniz.

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

            module purge
            eval "$(/truba/sw/centos7.9/lib/anaconda3/2021.11/bin/conda shell.bash hook)"
            conda activate fast-mpi4py

            mpirun -np 4 python3 hello_mpi.py

    .. group-tab:: hello_mpi2.slurm

        .. code-block:: bash

            #!/bin/bash
            #SBATCH -p barbun             #İşin çalıştırılacağı kuyruk adı
            #SBATCH -N 4                  #n adet taskın başlatılacağı node sayısı
            #SBATCH -n 4                  #başlatılacak görev sayısı
            #SBATCH -A {kullanıcı_adı}    #İşi kuyruğa gönderen kullanıcını ismi
            #SBATCH -o hello_mpi2         #çıktıların yazılacağı dosya ismi
            #SBATCH -J hello_mpi2         #İşin kuyrukta görülecek adı

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
    
`kaynakça <https://blogonparallelcomputing.files.wordpress.com/2017/04/launchmpi1.png>`_

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

    .. group-tab:: serial.slurm    

        .. code-block:: bash

            #!/bin/bash

            #SBATCH -p barbun 
            #SBATCH -N 1
            #SBATCH -n 1
            #SBATCH -c 4 
            #SBATCH -A bgorgun
            #SBATCH --output=/truba/scratch/serial.out
            #SBATCH -J serial


            eval "$(/truba/sw/centos7.9/lib/anaconda3/2021.11/bin/conda shell.bash hook)"

            srun python3 serial_example.py


        .. code-block:: output
                
                Harcanan sure :  28.698472655989463
                Integral degeri:  24.347402547472264

    .. group-tab:: numba.py

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

    .. group-tab:: numba.slurm

        .. code-block:: bash

            #!/bin/bash
            #SBATCH -p barbun 
            #SBATCH -N 1
            #SBATCH -n 1
            #SBATCH -c 4
            #SBATCH -A bgorgun
            #SBATCH --output=/truba/scratch/{kullanici_adi}/numba.out
            #SBATCH -J numba_work


            eval "$(/truba/sw/centos7.9/lib/anaconda3/2021.11/bin/conda shell.bash hook)"
            conda activate numba_env

            export NUMBA_NUM_THREADS=4
            srun python numba_2d.py


        .. code-block:: output

            Harcanan sure : 0.6155008749919944
            Integral degeri: 24.347402547472264



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
                dx = (x_max-x_min)/ float(n)
                dy = (y_max-y_min) / float(n)


                integral = 0.0
                workload= n/numprocess
                begin = int(workload*processindex)
                end= int(workload*(processindex+1))

                for i in range(begin,end):
                    for j in range(n):
                        x = (x_min + i) * dx
                        y = (y_min + j) * dy
                        integral += f(x,y)*dx*dy

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

        .. group-tab:: 

            .. code-block:: bash
                #!/bin/bash

                #SBATCH -p barbun 
                #SBATCH -N 1
                #SBATCH -n 1
                #SBATCH -c 4 
                #SBATCH -A bgorgun
                #SBATCH --output=/truba/scratch/{kullanici_adi}/multiprocess.out
                #SBATCH -J mp


                eval "$(/truba/sw/centos7.9/lib/anaconda3/2021.11/bin/conda shell.bash hook)"

                srun python3 multiprocess_2d.py


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
            processor_name = MPI.Get_processor_name()  # Node ismini al

            def f(x, y):
                return x * y

            def integrate_2d_mpi(f, x_min, x_max, y_min, y_max, n, nprocs, myrank):

                dx = (x_max - x_min) / float(n)
                dy = (y_max - y_min) / float(n)

                sum = 0.0
                workload = n // nprocs
                begin = int(workload * myrank)
                end = int(workload * (myrank + 1))

                for i in range(begin, end):
                    for j in range(n):
                        x = (x_min + i) * dx
                        y = (y_min + j) * dy
                        sum += f(x, y) * dx * dy
                partial_integrals = sum
                return partial_integrals

            if __name__ == "__main__":
                start = perf_counter()

                p = integrate_2d_mpi(f, 0, math.pi, 0, math.pi, 10000, nprocs, myrank)
                integral = comm.reduce(p, op=MPI.SUM, root=0)

                end = perf_counter()

                if myrank == 0:
                    print(f"Harcanan süre: {end - start}")
                    print(f"Integral değeri: {integral}")
                print(f"Rank {myrank} çalışıyor - Node ismi: {processor_name}")

    .. group-tab::

        .. code-block:: bash
            #!/bin/bash
            #SBATCH -p barbun           
            #SBATCH -N 4                  
            #SBATCH -n 4                  
            #SBATCH -A bgorgun
            #SBATCH --time=00:05:00 
            #SBATCH --output=/truba/scratch/{kullanici_adi}/openmpi.out          
            #SBATCH -J hello_mpi          

            module purge
            eval "$(/truba/sw/centos7.9/lib/anaconda3/2021.11/bin/conda shell.bash hook)"
            conda activate fast-mpi4py

            mpirun -np 4 python3 openmpi.py

        .. code-block:: output

            Rank 3 çalışıyor - Node ismi: barbun116.yonetim
            Rank 2 çalışıyor - Node ismi: barbun115.yonetim
            Rank 1 çalışıyor - Node ismi: barbun91.yonetim
            Rank 0 çalışıyor - Node ismi: barbun90.yonetim

            Harcanan süre: 6.725769966840744
            Integral değeri: 24.347402547473603
            
               




.. image:: /assets/python-howto/images/1d_integral.png
    :align: center
    :width: 180px


.. tabs:: 
    .. group-tab:: Multiprocessing ve MPI Örneği

        .. code-block:: python

            from time import perf_counter
            from multiprocessing import Pool
            from mpi4py import MPI

            comm = MPI.COMM_WORLD
            nprocs = comm.Get_size()
            myrank = comm.Get_rank()
            processor_name = MPI.Get_processor_name()  # Node ismini al

            def trapezoidal_rule(f, a, b, n_sub):
                h = (b - a) / n_sub
                integral = 0.5 * (f(a) + f(b))
                for i in range(1, n_sub):
                    integral += f(a + i * h)
                integral *= h
                return integral

            def f(x):
                return x

            def integrate_1d_multiprocess(sub_interval_start, sub_interval_end, n_sub):
                return trapezoidal_rule(f, sub_interval_start, sub_interval_end, n_sub)

            def main():
                x_min = 0
                x_max = 4
                n_sub = 100000
            
                #mpi icin kullanilacak core sayisi
                num_cores_per_rank = 4  

                start = perf_counter()

                # Her rank, kendi sub intervalini hesaplar.
                width = (x_max - x_min) / nprocs
                sub_interval_start = x_min + myrank * width
                sub_interval_end = x_min + (myrank + 1) * width
                
                print("sub_interval_start", sub_interval_start) 
                print("sub_interval_end", sub_interval_end)
            
                # Paralelleştirme için alt aralıkları berlirledim, pool isleminden once oldu bu.
                sub_width = (sub_interval_end - sub_interval_start) / num_cores_per_rank
                print("sub_width", sub_width)
                sub_intervals = [(sub_interval_start + i * sub_width,
                                sub_interval_start + (i + 1) * sub_width,
                                n_sub // num_cores_per_rank) for i in range(num_cores_per_rank)]
                print("sub intervals",sub_intervals)
                with Pool(processes=num_cores_per_rank) as p:
                    local_results = p.starmap(integrate_1d_multiprocess, sub_intervals)

                local_sum = sum(local_results)
                print(f"Rank {myrank} çalışıyor - Node ismi: {processor_name} {local_sum:.2f} burada hesaplandı")

                # Sonuçları birleştirmek için MPI reduce kullanılır
                global_sum = comm.reduce(local_sum, op=MPI.SUM, root=0)

                end = perf_counter()

                if myrank == 0:
                    print("Toplam integral sonucu:", global_sum)
                    print("Harcanan zaman:", end - start)

            if __name__ == "__main__":
                main()



        
   
    .. group-tab:: SLURM dosyası

        .. code-block:: bash

            #!/bin/bash

            #SBATCH -p barbun
            #SBATCH -N 4
            #SBATCH -n 4
            #SBATCH -c 4
            #SBATCH --cpus-per-task=4
            #SBATCH -J mpi_omp
            #SBATCH -o mpi_omp.out
            #SBATCH --time=00-05:00
            #SBATCH -A {kullanici_adi}

            module purge
            eval "$(/truba/sw/centos7.9/lib/anaconda3/2021.11/bin/conda shell.bash hook)"
            conda activate fast-mpi4py

            mpirun -np 4 python3 mpi_omp.py

        .. code-block:: output

            Rank 3 çalışıyor - Node ismi: barbun42.yonetim 3.50 burada hesaplandı
            Rank 1 çalışıyor - Node ismi: barbun15.yonetim 1.50 burada hesaplandı
            Rank 2 çalışıyor - Node ismi: barbun41.yonetim 2.50 burada hesaplandı
            Rank 0 çalışıyor - Node ismi: barbun8.yonetim 0.50 burada hesaplandı
            Toplam integral sonucu: 8.000000000000002
            Harcanan zaman: 0.07213476859033108


    .. group-tab:: 

        .. code-block:: bash

            #!/bin/bash

            #SBATCH -p barbun
            #SBATCH -N 1
            #SBATCH -n 4
            #SBATCH -c 1
            #SBATCH -J integration1D_multiprocess
            #SBATCH -o multiprocess.out
            #SBATCH --time=00-02:00
            #SBATCH -A bgorgun

            module purge

            srun python3 multiprocess.py

        .. code-block:: output

            Her bir alt processin sonucu [0.4999999999999999, 1.5000000000000004, 2.5, 3.4999999999999996]
            Sonuc: 8.0
            Harcanan zaman: 0.2714084889739752

    