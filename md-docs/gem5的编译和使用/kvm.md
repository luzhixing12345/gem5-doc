
# kvm

gem5 in kvm: [gem5 using_kvm](https://www.gem5.org/documentation/general_docs/using_kvm/)

检查处理器是否支持硬件虚拟化

```bash
egrep -c '(vmx|svm)' /proc/cpuinfo
```

安装依赖

```bash
sudo apt-get install qemu-kvm libvirt-daemon-system libvirt-clients bridge-utils cpu-checker
```

```bash
sudo adduser `id -un` libvirt
sudo adduser `id -un` kvm
```

```bash
scons build/X86/gem5.opt -j`nproc`
./build/X86/gem5.opt configs/example/gem5_library/x86-ubuntu-run-with-kvm.py
```

## 参考

- [gem5 using_kvm](https://www.gem5.org/documentation/general_docs/using_kvm/)
- [phoenixnap ubuntu-install-kvm](https://phoenixnap.com/kb/ubuntu-install-kvm)