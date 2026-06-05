# Installation

Get disk information:
```bash
talosctl get disks --insecure --nodes 5.135.136.115
```

```bash
NODE   NAMESPACE   TYPE   ID        VERSION   SIZE     READ ONLY   TRANSPORT   ROTATIONAL   WWID               MODEL                 SERIAL
       runtime     Disk   loop0     2         4.1 kB   true                                                                                                                                                        
       runtime     Disk   loop1     2         102 kB   true                                                                                                                                                        
       runtime     Disk   loop2     2         83 MB    true                                                                                                                                                        
       runtime     Disk   nvme0n1   2         450 GB   false       nvme                     nvme.8086-43565046363332353030394b34353052474e-494e54454c205353445045324d583435304737-00000001   INTEL SSDPE2MX450G7   CVPF6325009K450RGN
       runtime     Disk   nvme1n1   2         450 GB   false       nvme                     nvme.8086-43565046373136323030373634353052474e-494e54454c205353445045324d583435304737-00000001   INTEL SSDPE2MX450G7   CVPF71620076450RGN
```

```bash
talosctl get links --insecure --nodes 5.135.136.115
```

```bash
NODE   NAMESPACE   TYPE         ID        VERSION   ALIAS   TYPE       KIND     HW ADDR                                           OPER STATE   LINK STATE
       network     LinkStatus   bond0     1                 ether      bond     7e:de:92:a1:a6:ce                                 down         false
       network     LinkStatus   dummy0    1                 ether      dummy    56:43:8e:d5:e2:47                                 down         false
       network     LinkStatus   eno1      3                 ether               0c:c4:7a:da:d1:64                                 up           true
       network     LinkStatus   eno2      3                 ether               0c:c4:7a:da:d1:65                                 up           true
       network     LinkStatus   ip6tnl0   1                 tunnel6    ip6tnl   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00   down         false
       network     LinkStatus   lo        2                 loopback            00:00:00:00:00:00                                 unknown      true
       network     LinkStatus   sit0      1                 sit        sit      00:00:00:00                                       down         false
       network     LinkStatus   teql0     1                 void                                                                  down         false
       network     LinkStatus   tunl0     1                 ipip       ipip     00:00:00:00                                       down         false
```

```bash
talosctl version --insecure --nodes 5.135.136.115
```

```bash
Client:
        Tag:         v1.13.2
        SHA:         undefined
        Built:       
        Go version:  go1.26.3
        OS/Arch:     darwin/arm64
Server:
        NODE:        5.135.136.115
        Tag:         v1.13.3
        SHA:         befeda7c
        Built:       
        Go version:  go1.26.3
        OS/Arch:     linux/amd64
        Enabled:     
```

```bash
talosctl get addresses --insecure --nodes 5.135.136.115
```

```bash
NODE   NAMESPACE   TYPE            ID                                 VERSION   ADDRESS                       LINK
       network     AddressStatus   eno1/5.135.136.115/24              1         5.135.136.115/24              eno1
       network     AddressStatus   eno1/fe80::ec4:7aff:feda:d164/64   2         fe80::ec4:7aff:feda:d164/64   eno1
       network     AddressStatus   eno2/fe80::ec4:7aff:feda:d165/64   2         fe80::ec4:7aff:feda:d165/64   eno2
       network     AddressStatus   lo/127.0.0.1/8                     1         127.0.0.1/8                   lo
       network     AddressStatus   lo/::1/128                         1         ::1/128                       lo
```