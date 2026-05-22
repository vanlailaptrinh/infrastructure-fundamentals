# KVM / QEMU / Intel VT-x / AMD-V

## 1. What is KVM?

**Reference:** [https://linux-kvm.org/page/Main_Page](https://linux-kvm.org/page/Main_Page)

**KVM (Kernel-based Virtual Machine)**
KVM is the virtualization module built into the Linux kernel, turning Linux into a hypervisor. 
KVM utilizes the hardware virtualization features of the CPU:
- Intel VT-x
- AMD-V

When KVM is enabled, Linux can run multiple Virtual Machines (VMs) such as:
- Ubuntu VM
- Windows Server VM
- CentOS VM
- pfSense VM

Each VM has its own dedicated virtual CPU, RAM, disk, and network interface card. However, KVM primarily handles CPU and memory virtualization at the kernel level. It is not a complete VM management interface on its own.

## 2. What is QEMU?

**Reference:** [https://www.qemu.org/](https://www.qemu.org/)

QEMU is an open-source software used for machine emulation and virtualization.
It can build a complete "virtual computer" including a CPU, RAM, disk, network card, BIOS/UEFI, and I/O devices to run an operating system or programs inside it.

**In Short:**
QEMU is the program that creates and runs the virtual machine.
- When running standalone, it can emulate hardware using software.
- When combined with KVM, it runs VMs with high performance by utilizing the physical CPU.

**Key Takeaway:**
QEMU builds the virtual machine. KVM helps that virtual machine run quickly on the physical CPU.
⇒ QEMU is an open-source machine emulator and virtualizer responsible for creating virtual hardware for a VM, such as the CPU, RAM, disk, network card, and I/O devices. When combined with KVM on Linux, QEMU can run VMs with near-native performance by taking advantage of the CPU's hardware virtualization capabilities.

## 3. What are Intel VT-x / AMD-V?

**Reference:** [https://en.wikipedia.org/wiki/X86_virtualization](https://en.wikipedia.org/wiki/X86_virtualization)

Intel VT-x and AMD-V are the names of the hardware virtualization technologies for x86/x86-64 architecture CPUs from Intel and AMD, respectively.
"They are not software, not a hypervisor, and not a virtual machine."

⇒ x86 virtualization is described as the use of hardware-assisted virtualization capabilities on x86/x86-64 CPUs. Intel calls this technology VT-x, while AMD calls it AMD-V. In Linux, Intel CPUs have the `vmx` flag, and AMD CPUs have the `svm` flag.

**How to check on your personal laptop:**
```bash
egrep -o 'vmx|svm' /proc/cpuinfo | sort -u
```

**Examples:**
- Intel CPU: **vmx**
- AMD CPU: **svm**

![alt text](<images/KVM_QEMU_Intel_VT-x_AMD-V(1).png>)



### 3.1. What do VT-x / AMD-V allow?
They allow the CPU to have a special execution mode for VMs.
For example, with Intel VT-x, the CPU can switch between:
- **Host mode:** Running the Linux host / hypervisor.
- **Guest mode:** Running the operating system inside the VM.

The Guest OS still believes it is running with the highest privileges:
- A Guest Windows/Linux system thinks it is in ring 0.

But in reality, the CPU still protects the host on the outside.
**Simply put:**
The Guest OS has the "illusion" that it is controlling the CPU, but the CPU + KVM still retain actual control for the host.

### 3.2. How does KVM use VT-x / AMD-V?
KVM is a module in the Linux kernel.
KVM does not create virtualization power out of thin air. It uses the CPU's VT-x or AMD-V features to run the VM's vCPU (virtual CPU).

**The exact flow is:**
1. QEMU creates the VM.
2. QEMU creates the vCPU, RAM, virtual disk, and virtual network.
3. QEMU calls `/dev/kvm`.
4. KVM utilizes Intel VT-x / AMD-V.
5. The VM's vCPU is executed on the physical CPU.

**Summary:**
- **VT-x / AMD-V** = Hardware feature of the CPU.
- **KVM** = The component within the Linux kernel that utilizes this feature.
- **QEMU** = The component that builds the complete virtual machine.

### 3.3. The synergy between components
- QEMU creates the vCPU for the VM.
- KVM helps that vCPU run directly on the physical CPU using Intel VT-x / AMD-V.
- The Linux kernel schedules that vCPU onto a physical CPU core.

**4 Basic Layers:**
1. **Proxmox / OpenStack / virt-manager** = VM management layer.
2. **QEMU** = Builds the virtual machine: virtual CPU, virtual RAM, virtual disk, virtual network.
3. **KVM** = Linux kernel module that helps run the VM's vCPU/RAM efficiently.
4. **Intel VT-x / AMD-V** = Hardware features in the physical CPU to support virtualization.

**Key statements to remember:**
QEMU creates the complete virtual machine. KVM uses Intel VT-x or AMD-V to execute the virtual machine's vCPU on the physical CPU with high performance.
**Or:**
QEMU creates the VM and spawns vCPU threads. KVM takes those vCPU threads and uses VT-x/AMD-V to run them on the physical CPU in guest mode. The Linux scheduler decides which physical core the thread runs on.

### 3.4. Some Concepts
- **Driver** = Software that helps the OS communicate with the hardware.
- **VirtIO** = Virtual drivers/devices optimized for VMs.

**Common types of VirtIO:**
- `virtio-net` = High-performance virtual network card.
- `virtio-blk` = Virtual block storage drive.
- `virtio-scsi` = Virtual disk controller.
- `virtio-balloon` = Dynamic RAM management for the VM.
- `virtio-gpu` = Virtual GPU/display.
- `virtio-rng` = Virtual random number generator.

⇒ If the VM is Linux, it typically already has VirtIO drivers built into the kernel.
⇒ If the VM is Windows, you usually need to install additional VirtIO drivers via an ISO file.

- **libvirt** = VM management layer, sitting on top of QEMU/KVM ([https://libvirt.org/](https://libvirt.org/)).
- **virt-manager** = A GUI that uses libvirt to create VMs.

## 4. General Architecture

### 4.1. Overview: VM Creation And Core Components 

![alt text](<images/KVM_QEMU_Intel_VT-x_AMD-V(2).png>)


### 4.2. Execution Flow Of A vCPU (While the VM is running)

![alt text](<images/KVM_QEMU_Intel_VT-x_AMD-V(3).png>)

