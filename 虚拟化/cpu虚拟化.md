
<img src="https://github.com/Yongli-Lisa/Linux-Notes1/blob/aec10522017a25c190e8c4e13270457d97184cb6/Img/%E8%99%9A%E6%8B%9F%E5%8C%96/cpu%E8%99%9A%E6%8B%9F%E5%8C%961.PNG" width="700px">  
<img src="https://github.com/Yongli-Lisa/Linux-Notes1/blob/aec10522017a25c190e8c4e13270457d97184cb6/Img/%E8%99%9A%E6%8B%9F%E5%8C%96/cpu%E8%99%9A%E6%8B%9F%E5%8C%962.PNG" width="800px">

**CPU虚拟化步骤：**  
1. 初始化所有的 Module   
3. 解析 qemu 的命令     
5. 初始化 machine    
7. 初始化块设备    
9. 初始化计算虚拟化的加速模式    
11. 初始化网络设备    
13. CPU 虚拟化  
    

**CPU虚拟化过程：**  
<img src="https://github.com/Yongli-Lisa/Linux-Notes1/blob/aec10522017a25c190e8c4e13270457d97184cb6/Img/%E8%99%9A%E6%8B%9F%E5%8C%96/MachineClass.PNG" width="800px">  
<img src="https://github.com/Yongli-Lisa/Linux-Notes1/blob/aec10522017a25c190e8c4e13270457d97184cb6/Img/%E8%99%9A%E6%8B%9F%E5%8C%96/CPU%20%E7%9A%84%E8%99%9A%E6%8B%9F%E5%8C%96%E8%BF%87%E7%A8%8B.png" width="1500px">  



首先，我们要定义 CPU 这种类型的 TypeInfo 和 TypeImpl、继承关系，并且声明它的类初始化函数。  
在 qemu 的 main 函数中调用 MachineClass 的 init 函数，这个函数既会初始化 CPU，也会初始化内存。  
CPU 初始化的时候，会调用 pc_new_cpu 创建一个虚拟 CPU，它会调用 CPU 这个类的初始化函数。  
每一个虚拟 CPU 会调用 qemu_thread_create 创建一个线程，线程的执行函数为 qemu_kvm_cpu_thread_fn。  
在虚拟 CPU 对应的线程执行函数中，我们先是调用 kvm_vm_ioctl(KVM_CREATE_VCPU)，在内核的 KVM 里面，创建一个结构 struct vcpu_vmx，表示这个虚拟 CPU。在这个结构里面，有一个 VMCS，用于保存当前虚拟机 CPU 的运行时的状态，用于状态切换。  
在虚拟 CPU 对应的线程执行函数中，我们接着调用 kvm_vcpu_ioctl(KVM_RUN)，在内核的 KVM 里面运行这个虚拟机 CPU。运行的方式是保存宿主机的寄存器，加载客户机的寄存器，然后调用 __ex(ASM_VMX_VMLAUNCH) 或者 __ex(ASM_VMX_VMRESUME)，进入客户机模式运行。一旦退出客户机模式，就会保存客户机寄存器，加载宿主机寄存器，进入宿主机模式运行，并且会记录退出虚拟机模式的原因。大部分的原因是等待 I/O，因而宿主机调用 kvm_handle_io 进行处理。
