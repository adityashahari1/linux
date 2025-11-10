CMPE283 Assignment 

Steps followed - 
## How to Reproduce
Steps to rebuild and test the instrumentation:

1. Clone and configure kernel
   ```bash
   git clone https://github.com/<your-username>/linux.git
   cd linux
   yes "" | make olddefconfig
   make -j"$(nproc)"
   sudo make modules_install
   sudo make install
   sudo reboot

2. Edit arch/x86/kvm/svm/svm.c to add exit counters and printk lines.

3. Rebuild only KVM:
make -j"$(nproc)" M=arch/x86/kvm
sudo make M=arch/x86/kvm modules_install
sudo modprobe -r kvm_amd kvm || true
sudo modprobe kvm_amd

4. View logs:
sudo dmesg | grep CMPE283 | tail

> Assumes Debian/Ubuntu outer VM and AMD host (SVM path). If using Intel on a different machine, the analogous file is `arch/x86/kvm/vmx/vmx.c` and the module is `kvm_intel`.



Answers

1) Detailed steps
Forked Linux, cloned to outer VM, configured kernel.
Edited arch/x86/kvm/svm/svm.c to add counters & printk.
Rebuilt only KVM modules and reloaded them.
Verified successful build and clean dmesg output.
Attempted dummy VM to confirm instrumentation.

2) Exit frequency
Exits come in bursts, not constant.
High during boot (CPUID/MSR), I/O setup, and memory mapping; then mostly HLT while idle.
A full guest boot is roughly 50k–500k exits depending on devices and workload.

3) Most vs least frequent exits
Most: CPUID (0x072), MSR Read/Write (0x07C), IOIO (0x07B), HLT (0x078).
Least: NMI (0x040), SMI (0x041), Nested Page Fault (0x400).
