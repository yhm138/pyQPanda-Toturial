酉矩阵分解
=====================
目前，量子计算的算法通常用量子线路表示，量子线路包括量子逻辑门操作。
通常，连续的一段量子线路通常包含几十上百个甚至成千上万个量子逻辑门操作，而量子逻辑门数量或单个量子逻辑门操作的量子比特数越多，计算过程越为复杂，导致量子线路的模拟效率较低，且对硬件资源的占用较多。

算法目标
>>>>>>>>>>
----

对于上述问题，有必要对量子线路进行一种等价转换，需要减少量子线路中逻辑门的数量，
同时在此基础上，需要确保转换前后整个量子线路对应的酉矩阵完全相同.

算法概述
>>>>>>>>>>
----

本文介绍的算法是将一个N阶酉矩阵，分解成不超过r = N(N−1)/2个带有少量控制的单量子逻辑门序列，其中N=2^n，分解的产物满足如下等式关系

.. centered:: :math:`U_rU_{r-1}···U_3U_2U_1U=I_N`

从而可以得到原矩阵U的分解结果表示

.. centered:: :math:`U=U_1^{\dagger}U_2^{\dagger}U_3^{\dagger}···U_{r-1}^{\dagger}U_r^{\dagger}`

使用介绍
>>>>>>>>>>>>>>>>
----

pyqpanda中设计了 ``matrix_decompose`` 接口用于进行酉矩阵分解，该接口需要两个参数，
第一个是使用到的所有量子比特，第二个是待分解的酉矩阵，该函数的输出是转换后的量子线路。

实例
>>>>>>>>>>
----

.. _酉矩阵分解示例程序:
以下示例展示了酉矩阵接口的使用方式

    .. code-block:: python
  
        import pyqpanda as pq
        import numpy as np

        if __name__=="__main__":

            machine = pq.CPUQVM()
            machine.init_qvm()
            q = machine.qAlloc_many(2)
            c = machine.cAlloc_many(2)


            prog = pq.QProg()
            prog << pq.H(q[0]) << pq.CNOT(q[0],q[1]) <<pq.RX(q[1],np.pi) <<pq.RY(q[0],np.pi) <<pq.RY(q[1],np.pi) 

            # 获取一个线路的酉矩阵
            source_matrix = pq.get_matrix(prog,True)

            print("source_matrix : ")
            print(source_matrix)

            out_cir = pq.matrix_decompose(q,np.array(source matrix).reshape(4,4))
            circuit_matrix = pq.get_matrix(out_cir)

            print("the decomposed matrix : ")
            print(circuit_matrix)

            source_matrix  = np.round(np.array(source_matrix),3)
            circuit_matrix = np.round(np.array(circuit_matrix),3)

            if np.all(source_matrix == circuit_matrix):
                print('matrix decompose ok !')
            else:
                print('matrix decompose false !')


上述实例运行的结果如下：

    .. code-block:: python

        source_matrix : 
        [(4.329780281177466e-17, +8.659560562354932e-17j), (-4.329780281177466e, -17+0j), (-5.302451562355311e, -33-0.7071067811865475j), (0.7071067811865475j), 
         (0.7071067811865475j), (5.302451562355311e-33, +0.7071067811865475j), (-4.329780281177466e, -17+0j), (-4.329780281177466e-17, -8.659560562354932e-17j), 
         (0.7071067811865475j), (5.302451562355311e-33, -0.7071067811865475j), (-4.329780281177466e, -17+0j), (4.329780281177466e-17, -8.659560562354932e-17j), 
         (4.329780281177466e-17-8.659560562354932e-17j), (4.329780281177466e-17, +0j), (5.302451562355311e-33, -0.7071067811865475j), (-0.7071067811865475j)]
        the decomposed matrix :
        [(4.329780281177466e-17, +8.659560562354932e-17j), (-4.329780281177466e, -17+0j), (-5.302451562355311e, -33-0.7071067811865475j), (0.7071067811865475j), 
         (0.7071067811865475j), (5.302451562355311e-33, +0.7071067811865475j), (-4.329780281177466e, -17+0j), (-4.329780281177466e-17, -8.659560562354932e-17j), 
         (0.7071067811865475j), (5.302451562355311e-33, -0.7071067811865475j), (-4.329780281177466e, -17+0j), (4.329780281177466e-17, -8.659560562354932e-17j), 
         (4.329780281177466e-17-8.659560562354932e-17j), (4.329780281177466e-17, +0j), (5.302451562355311e-33, -0.7071067811865475j), (-0.7071067811865475j)]
        matrix decompose ok !

从输出的结果可以看出，分解前后的矩阵完全相同，对于一个量子比特数目确定的量子系统，
即使分解前的量子线路含有成千上万个量子逻辑门，该接口可以将分解后的量子线路复杂度控制在合理范围之内，
完全不受到分解前量子线路复杂度的影响，

    .. note::

        1. 该接口的输入参数必须为酉矩阵。
        2. 通过将分解的结果数量约束在一个限定范围内，有效减少了量子线路中的量子逻辑门数量，极大地提升了量子算法的模拟效率。
        3. 示例程序中， ``get_matrix`` 接口用于获取一个量子线路对应的矩阵。