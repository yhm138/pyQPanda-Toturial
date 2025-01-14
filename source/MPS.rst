.. _张量网络量子电路模拟器:

张量网络量子虚拟机
=================
----

对于一个 :math:`N` 个量子比特的自旋体系，对应的希尔伯特空间维数为 :math:`2^{N}` 。

对于该复杂系统的状态演化，传统的全振幅模拟器将其看做一个有 :math:`2^{N}` 个元素的一维向量。

然而从张量网络的角度来看，整个系统量子态的系数对应 :math:`2^{N}` 维张量（即N阶张量，即有 :math:`N` 个指标，每个指标的维数是2），量子操作算符的系数为 :math:`2^{2N}` 维张量（ :math:`2N` 阶张量，即有个 :math:`2N` 指标，每个指标的维数是2），我们可以用如下图形来表示量子态：

.. image:: images/state.png
   :align: center  

当量子系统的自旋个数增加时，量子态系数的个数随指数增加，称为指数墙问题，这一障碍限制了传统全振幅模拟器的最大模拟自旋数和模拟性能。

但是可通过张量网络处理这一问题，从而绕过指数墙障碍，在张量网络中，我们对量子系统的模拟，包括量子逻辑门操作和测量操作，均可以通过对于张量的缩并与分解来实现。矩阵乘积态是张量网络中最常用的表示形式，在多线性代数中称为张量列或TT（Tensor-Train），示意图如下。

.. image:: images/MPS.png
   :align: center  

将量子态分解成等式右边的表示形式，对于量子线路中部分量子逻辑门操作，可以将全局问题转化为局部的张量处理问题，从而有效地降低了时间复杂度和空间复杂度。

应用场景
>>>>>>>>>>>>>>>>
----

在量子电路的模拟方法中，选择合适的模拟后端非常重要，不同量子线路模拟器的适用场所如下：

     ``全振幅量子虚拟机`` ：全振幅模拟器可以同时模拟和存储量子态的全部振幅，但受限于机器的内存条件，量子比特达到50位已是极限，适合低比特高深度的量子线路，比如低比特下的谷歌随机量子线路以及需要获取全部模拟结果的场景等。
    
     ``部分振幅量子虚拟机`` ：部分振幅模拟器依赖于其他模拟器提供的低比特量子线路振幅模拟结果，能模拟更高的比特数量，但能模拟的深度降低，通常用于获取量子态振幅的部分子集模拟结果。
    
     ``单振幅量子虚拟机`` ：单振幅模拟器能模拟更高的量子比特线路图，同时模拟的性能较高，不会随着量子比特数目增加呈指数型增长，但随着线路深度增加，模拟性能急剧下降，同时难以模拟多控制门也是其缺点，该模拟器适用于高比特低深度的量子线路模拟，通常用于快速地模拟获得单个量子态振幅结果。
     
     ``张量网络量子虚拟机`` ：张量网络模拟器与单振幅类似，与单振幅对比，可以模拟多控制门，同时在深度较高的线路模拟上存在性能优势。
    
     ``量子云虚拟机`` ：量子云虚拟机可以将任务提交在远程高性能计算集群上运行，突破本地硬件性能限制，同时支持在真实的量子芯片上运行量子算法。

使用介绍
>>>>>>>>>>>>>>>>
----

``pyqpanda`` 中可以通过 ``MPSQVM`` 类实现用张量网络模拟量子电路。和许多其他模拟器的使用方法一样，都具有相同的量子虚拟机接口，比如下述简单的使用示例代码:

    .. code-block:: python

        from numpy import pi
        from pyqpanda import *

        # 构建量子虚拟机
        qvm = MPSQVM()

        # 初始化操作
        qvm.set_configure(64, 64)
        qvm.init_qvm()

        q = qvm.qAlloc_many(10)
        c = qvm.cAlloc_many(10)

        # 构建量子程序
        prog = QProg()
        prog << hadamard_circuit(q)\
            << CZ(q[2], q[4])\
            << CZ(q[3], q[7])\
            << CNOT(q[0], q[1])\
            << Measure(q[0], c[0])\
            << Measure(q[1], c[1])\
            << Measure(q[2], c[2])\
            << Measure(q[3], c[3])

        # 量子程序运行100次，并返回测量结果
        result = qvm.run_with_configuration(prog, c, 100)

        # 打印量子态在量子程序多次运行结果中出现的次数
        print(result)

        qvm.finalize()

完整示例代码
>>>>>>>>>>
----

.. _张量网络虚拟机示例程序:
以下示例展示了张量网络模拟器计算部分接口的使用方式

    .. code-block:: python

        from numpy import pi
        from pyqpanda import *

        qvm = MPSQVM()
        qvm.set_configure(64, 64)
        qvm.init_qvm()

        q = qvm.qAlloc_many(10)
        c = qvm.cAlloc_many(10)

        prog = QProg()
        prog << hadamard_circuit(q)\
            << CZ(q[2], q[4])\
            << CZ(q[3], q[7])\
            << CNOT(q[0], q[1])\
            << CZ(q[3], q[7])\
            << CZ(q[0], q[4])\
            << RY(q[7], pi / 2)\
            << RX(q[8], pi / 2)\
            << RX(q[9], pi / 2)\
            << CR(q[0], q[1], pi)\
            << CR(q[2], q[3], pi)\
            << RY(q[4], pi / 2)\
            << RZ(q[5], pi / 4)\
            << Measure(q[0], c[0])\
            << Measure(q[1], c[1])\
            << Measure(q[2], c[2])

        # Monte Carlo采样模拟接口
        result0 = qvm.run_with_configuration(prog, c, 100)

        # 概率测量接口
        result1 = qvm.prob_run_dict(prog, [q[0], q[1], q[2]], -1)

        print(result0)
        print(result1)

        qvm.finalize()

    上述代码中 ``run_with_configuration`` 与 ``prob_run_dict`` 接口分别用于Monte Carlo采样模拟和概率测量，他们分别输出模拟采样的结果和对应振幅的概率，上述程序的计算结果如下

    .. code-block:: python

        # Monte Carlo 采样模拟结果
        {'0000000000': 7, 
         '0000000001': 12, 
         '0000000010': 13, 
         '0000000011': 10, 
         '0000000100': 16, 
         '0000000101': 14, 
         '0000000110': 12, 
         '0000000111': 16}

        # 概率测量结果
        {'000': 0.12499999999999194, 
         '001': 0.12499999999999185, 
         '010': 0.12499999999999194, 
         '011': 0.124999999999992, 
         '100': 0.12499999999999198, 
         '101': 0.12499999999999194, 
         '110': 0.12499999999999198, 
         '111': 0.12499999999999208}