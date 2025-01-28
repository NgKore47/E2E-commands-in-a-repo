# E2E-commands-in-a-repo

## USRP(Software Defined Radios)

**Build Command**
```shell
sudo ./build_oai -w USRP --ninja --nrUE --gNB --build-lib "nrscope" -C
```

### B210

Run:
```shell
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.fr1.106PRB.usrpb210.conf --sa -E --continuous-tx
```

### N310

Run:
```shell
sudo ./nr-softmodem -O ../../n310_78_106_2x2.conf --sa --usrp-tx-thread-config 1
```

### X410

Run:
```shell
sudo ./nr-softmodem -O ../../3_x4xx.conf --sa --usrp-tx-thread-config 1
```


## Linuxptp

### Liteon + X710

```shell
sudo ./ptp4l -i ens2f0 -m -H -2 -s -f configs/c3.cfg
sudo ./phc2sys -w -m -s ens2f0 -R 8 -f configs/c3.cfg
```
### Liteon + e810

```shell
sudo ./ptp4l -i ens2f0 -m -H -2 -s -f configs/mod_c1_e810.cfg
sudo ./phc2sys -w -m -s ens2f0 -R 8 -f configs/mod_c1_e810.cfg
```
### Benetel + x710

```shell
sudo ./ptp4l -i ens2f0 -m -H -2 -s -f configs/c3.cfg
sudo ./phc2sys -w -m -s ens2f0 -R 8 -f configs/c3.cfg
```

### Bentel + e810

```shell
sudo ./ptp4l -i ens2f0 -m -H -2 -s -f configs/benetel_e810.cfg
sudo ./phc2sys -w -m -s ens2f0 -R 8 -f configs/benetel_e810.cfg

# OR

sudo ./ptp4l -f ./configs/benetel_e810.cfg -2 -i enp24s0f0 -m
sudo ./phc2sys -f ./configs/benetel_e810.cfg -s enp24s0f0 -w -m -R 8

## O-RU(Commercial Radios)

**Build Command**
```shell
./build_oai --gNB --ninja -t oran_fhlib_5g --cmake-opt -Dxran_LOCATION=$HOME/phy/fhi_lib/lib
```

### Run:

```shell
sudo ./nr-softmodem -O gnb.sa.band78.273prb.fhi72.4x4-liteon.conf --sa --thread-pool 3,5,7,9,11,12,13,14
```


## Additional Information


## For 1 Node system(18  cores)

### CPU configuration
```shell
intel_iommu=on iommu=pt mitigations=off mce=off idle=poll hugepagesz=1G hugepages=40 hugepagesz=2M hugepages=0 default_hugepagesz=1G selinux=0 enforcing=0 nmi_watchdog=0 softlockup_panic=0 audit=0 skew_tick=1 rcu_nocb_poll kthread_cpus=14-17 skew_tick=1 isolcpus=managed_irq,domain,0-13 nohz_full=0-13 rcu_nocbs=0-13 intel_pstate=disable nosoftlockup tsc=nowatchdog
```
### nr-softmodem

```shell
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.273prb.fhi72.4x4-liteon.conf --sa --thread-pool 3,5,7,9,11,12,13
```
### tuned-adm

```shell
seven@seven:~$ cat /etc/tuned/realtime-variables.conf 
# Examples:
# isolated_cores=2,4-7
isolated_cores=0-13
#
#
# Uncomment the 'isolate_managed_irq=Y' bellow if you want to move kernel
# managed IRQs out of isolated cores. Note that this requires kernel
# support. Please only specify this parameter if you are sure that the
# kernel supports it.
#
# isolate_managed_irq=Y
```

## For 2 Node system(56 Cores)

### CPU configuration
```shell
mitigations=off usbcore.autosuspend=-1 intel_iommu=on intel_iommu=pt selinux=0 enforcing=0 nmi_watchdog=0 softlockup_panic=0 audit=0 skew_tick=1 isolcpus=managed_irq,domain,0,2,4,6,8,10,12,14,16 nohz_full=0,2,4,6,8,10,12,14,16 rcu_nocbs=0,2,4,6,8,10,12,14,16 rcu_nocb_poll intel_pstate=disable nosoftlockup cgroup_disable=memory mce=off hugepagesz=1G hugepages=40 hugepagesz=2M hugepages=0 default_hugepagesz=1G isolcpus=managed_irq,domain,0,2,4,6,8,10,12,14 kthread_cpus=18-35 intel_pstate=disable nosoftlockup tsc=reliable
```
### nr-softmodem

```shell
sudo ./nr-softmodem -O ../../../targets/PROJECTS/GENERIC-NR-5GC/CONF/gnb.sa.band78.273prb.fhi72.4x4-liteon.conf --sa --thread-pool 30,31,32,33,34,35,36,37
```
### tuned-adm

```shell
# Examples:
isolated_cores=0,2,4,6,8,10,12,14,16
# isolated_cores=2-23
#
#
# Uncomment the 'isolate_managed_irq=Y' bellow if you want to move kernel
# managed IRQs out of isolated cores. Note that this requires kernel
# support. Please only specify this parameter if you are sure that the
# kernel supports it.
#
# isolate_managed_irq=Y
```

```
## Network Interface

### Liteon + x710

```shell
echo "0" > /sys/class/net/ens2f0/device/sriov_numvfs
echo "2" > /sys/class/net/ens2f0/device/sriov_numvfs
sudo ip link set ens2f0 vf 0 mac 00:11:22:33:44:99 vlan 564 spoofchk off trust on 
sudo ip link set ens2f0 vf 1 mac 00:11:22:33:44:99 vlan 564 spoofchk off trust on
modprobe vfio_pci

dpdk-devbind.py -s
dpdk-devbind.py --bind vfio-pci 0000:8a:02.0 0000:8a:02.1

sudo ifconfig ens2f0 mtu 1500
sudo ethtool -G ens2f0 rx 4096 tx 4096
```

### Liteon + e810

```shell
set -xe

sudo ethtool -G enp24s0f0 rx 4096
sudo ethtool -G enp24s0f0 tx 4096
sudo ifconfig enp24s0f0 mtu 1500

sudo modprobe -r iavf
sudo modprobe iavf
sudo sh -c 'echo 0 > /sys/class/net/enp24s0f0/device/sriov_numvfs'
sudo sh -c 'echo 2 > /sys/class/net/enp24s0f0/device/sriov_numvfs'
sudo ip link set enp24s0f0 vf 0 mac 00:11:22:33:44:66 vlan 564 qos 0 spoofchk off mtu 1500
sudo ip link set enp24s0f0 vf 1 mac 00:11:22:33:44:67 vlan 564 qos 0 spoofchk off mtu 1500

sudo /usr/local/bin/dpdk-devbind.py --unbind 0000:18:01.0
sudo /usr/local/bin/dpdk-devbind.py --unbind 0000:18:01.1
sudo modprobe vfio-pci
sudo /usr/local/bin/dpdk-devbind.py --bind vfio-pci 0000:18:01.0
sudo /usr/local/bin/dpdk-devbind.py --bind vfio-pci 0000:18:01.1
sudo /usr/local/bin/dpdk-devbind.py -s
```

### Multiple Liteon

```shell
echo "0" > /sys/class/net/ens2f0/device/sriov_numvfs
echo "4" > /sys/class/net/ens2f0/device/sriov_numvfs
sudo ip link set ens2f0 vf 0 mac 00:11:22:33:44:99 vlan 564 spoofchk off trust on 
sudo ip link set ens2f0 vf 1 mac 00:11:22:33:44:99 vlan 564 spoofchk off trust on
sudo ip link set ens2f0 vf 2 mac 00:11:22:33:44:99 vlan 564 spoofchk off trust on
sudo ip link set ens2f0 vf 3 mac 00:11:22:33:44:99 vlan 564 spoofchk off trust on
modprobe vfio_pci

dpdk-devbind.py -s
dpdk-devbind.py --bind vfio-pci 0000:8a:02.0 0000:8a:02.1 0000:8a:02.2 0000:8a:02.3

sudo ifconfig ens2f0 mtu 1500
sudo ethtool -G ens2f0 rx 4096 tx 4096
```

### Benetel + e810

```shell
set -x

sudo ethtool -G enp24s0f0 rx 4096
sudo ethtool -G enp24s0f0 tx 4096
sudo ifconfig enp24s0f0 mtu 9216

sudo modprobe -r iavf
sudo modprobe iavf
sudo sh -c 'echo 0 > /sys/class/net/enp24s0f0/device/sriov_numvfs'
sudo sh -c 'echo 2 > /sys/class/net/enp24s0f0/device/sriov_numvfs'
sudo ip link set enp24s0f0 vf 0 mac 00:11:22:33:44:66 vlan 3 qos 0 spoofchk off mtu 9216
sudo ip link set enp24s0f0 vf 1 mac 00:11:22:33:44:67 vlan 3 qos 0 spoofchk off mtu 9216

sudo /usr/local/bin/dpdk-devbind.py --unbind 0000:18:01.0
sudo /usr/local/bin/dpdk-devbind.py --unbind 0000:18:01.1
sudo modprobe vfio-pci
sudo /usr/local/bin/dpdk-devbind.py --bind vfio-pci 0000:18:01.0
sudo /usr/local/bin/dpdk-devbind.py --bind vfio-pci 0000:18:01.1
```
### CPU Power
`super user`
```shell
for ((i=0; i<$(nproc --all); i++)); do sudo cpufreq-set -c $i --min 3000000 --max 3100000 -g performance; done
```

