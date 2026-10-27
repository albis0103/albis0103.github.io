Hypervisor: VM monitor
- Type - 1: $\text{HardWare}\rightarrow \text{Hypervisor}\rightarrow\text{Guest OS}$
	- 直接操作硬體 $\in \text{kernel mode}$

- Type - 2:$\text{HardWare}\rightarrow \text{Host OS}\rightarrow \text{Hypervisor}\rightarrow\text{Guest OS}$ 
	- 需透過OS $\in \text{user mode}$

### KVM(Kernel - Based Virtual Machine)
: Linux-Kernel 內建虛擬化，屬於Type-1 Hypervisor，需搭配QEMU實現CPU Virtualization（讓Linux Kernel暫時切換給Guest OS當Hypervisor，把CPU切進VMX Non-Root模式），Guest OS指令直接給CPU硬體執行

note：硬體虛擬化
VMX Root Operation(Host端)
- Ring 0: Host Kernel(ex. cpu scheduler, memory management, **KVM modulem** , ..)
- Ring 3: Host application(user mode ex. QEMU)
VMX(Non - Root Operation)
- Ring 0: Guest OS kernel
- Ring 3: Guest application


### QEMU(Quick Emulator)
：完整模擬電腦硬體環境（CPU, 記憶體, I/O,..）讓Guest OS誤以為在真實硬體（Guest OS下I / O時發出Trap, 交給QEMU $\in \text{user space}$ ）。

Process
QEMU(User mode, Ring 3)
1. `open('dev/kvm')`
2. `ioctl(fd, KVM_CREATE_VM)`
3. `ioctl(fd, KVM_RUN)`
Linux Kernel(Ring 0)
4. KVM receive `ioctl()` 
5. 執行對應操作