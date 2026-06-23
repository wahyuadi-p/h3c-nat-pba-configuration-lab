# H3C CGNAT Lab - PBA Dynamic & PBA Deterministic

Repository ini berisi dokumentasi dan konfigurasi lab **H3C CGNAT (Carrier Grade NAT)** menggunakan metode **Port Block Allocation (PBA)**.

Implementasi yang tersedia:

- PBA Dynamic
- PBA Deterministic

Lab ini mensimulasikan jaringan ISP sederhana menggunakan:

- H3C BRAS sebagai PPPoE Server
- H3C Router sebagai CGNAT Gateway
- PPPoE Client sebagai Subscriber
- Public NAT Pool sebagai IPv4 Translation

---

## Topology

![Topology](topology.png)


```
                    Internet / Cloud
                          |
                          |
                    NAT-FAKE-PUB
                    H3C CGNAT
                          |
                          |
                    H3C-BRAS-1
                    PPPoE Server
                          |
                          |
                    PPPoE Client
                  /       |        \
                 /        |         \
             Client1   Client2    Client3
```

---

# Network Design

## Subscriber Network

Subscriber mendapatkan IP Private melalui PPPoE:

```
100.64.0.0/24
```

Example:

```
100.64.0.2
100.64.0.3
100.64.0.4
```

IP tersebut akan ditranslasikan oleh CGNAT menuju Public IP.

---

## Public NAT Pool

Public IP yang digunakan:

```
192.200.100.1 - 192.200.100.254
```

---

# PPPoE Service

PPPoE Server berjalan pada H3C BRAS.

Feature:

- PPP Authentication
- IP Address Assignment
- DNS Distribution
- Subscriber Management


Virtual Template:

```
Virtual-Template1
```

Authentication:

```
CHAP
PAP
```

---

# Port Block Allocation (PBA)

PBA merupakan metode NAT dimana setiap subscriber mendapatkan blok port tertentu.

Keuntungan:

- Mengurangi NAT session table
- Menghemat resource router
- Memudahkan tracking subscriber
- Cocok untuk implementasi ISP CGNAT

---

# PBA Dynamic

## Description

PBA Dynamic melakukan alokasi port secara otomatis berdasarkan kebutuhan subscriber.

Ketika subscriber membuat koneksi:

```
Subscriber
     |
100.64.x.x
     |
CGNAT
     |
Dynamic Port Allocation
     |
Public IP
     |
Internet
```

---

## Dynamic Configuration

NAT outbound:

```
nat outbound 3000 address-group 1
```

ACL subscriber:

```
acl advanced 3000

rule 0 permit ip source 100.64.0.0 0.0.0.255
```

NAT Pool:

```
nat address-group 1

address 192.200.100.1 192.200.100.254
```

Port Block:

```
port-block block-size 256
```

---

## Example Dynamic Allocation


Subscriber:

```
100.64.0.10
```

Mendapatkan:

```
Public IP:
192.200.100.10

Port:
2048-2303
```

Subscriber lain dapat memperoleh block berbeda secara dinamis.

---

# PBA Deterministic

## Description

PBA Deterministic menggunakan mapping yang tetap antara:

```
Private IP Subscriber
        |
        |
Public IP + Port Block
```

Berbeda dengan dynamic:

- Mapping selalu sama
- Lebih mudah dilakukan logging
- Cocok untuk kebutuhan ISP


---

## Deterministic Flow


```
Subscriber

100.64.0.10

       |

CGNAT

       |

192.200.100.10

Port Block 4096-4351

       |

Internet
```

---

## Deterministic Configuration


Interface NAT:

```
interface GigabitEthernet1/0

nat outbound port-block-group 1
```


Port Block Group:

```
nat port-block-group 1

local-ip-address 100.64.0.0 100.64.0.254

global-ip-pool 192.200.100.1 192.200.100.254

port-range 2048 65535
```

---

# Comparison

| Feature | PBA Dynamic | PBA Deterministic |
|---|---|---|
| Allocation | Dynamic | Fixed |
| Mapping | Berubah | Tetap |
| Logging | Membutuhkan NAT Log | Lebih mudah |
| Tracking User | Medium | High |
| ISP CGNAT | Support | Recommended |
| Resource Usage | Efficient | Efficient |

---

# Repository Structure

```
H3C-CGNAT-PBA
|
|-- README.md
|
|-- topology.png
|
|-- PBA-Dynamic
|      |
|      |-- h3c-pba-dynamic.cfg
|
|-- PBA-Deterministic
       |
       |-- h3c-pba-deterministic.cfg
```

---

# Device Information

Tested:

```
H3C Comware 7

Version:
7.1.059 ESS 0322
```

---

# Verification Command

Check PPPoE User:

```
display ppp access-user
```

Check NAT Session:

```
display nat session
```

Check NAT Mapping:

```
display nat port-block-group
```

Check Interface:

```
display interface brief
```

---

# Use Case

Project ini dapat digunakan untuk:

- ISP Broadband Network
- FTTH Network
- CGNAT Implementation
- IPv4 Conservation
- Network Engineering Lab
- H3C NAT Testing


---

# Disclaimer

Konfigurasi ini dibuat untuk:

- Lab environment
- Learning
- Testing

Untuk implementasi production perlu menyesuaikan:

- Hardware capability
- Subscriber scale
- Logging requirement
- NAT capacity


---

# Author

Network Engineering Lab

H3C | CGNAT | ISP Network
