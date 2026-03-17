# TRex EMU — DHCP Test Lab

Bu repo, **Cisco CSR1000v** yönlendiricisi ile **TRex v3.08 EMU (Emulation)** kullanılarak gerçekleştirilen DHCP adres dağıtım testini belgeler.

---

## 🖥️ Topoloji

```
[TRex Port 0] <-----> [CSR1K GigabitEthernet2]
  DHCP İstemciler          99.1.1.1/24 (DHCP Server)
```

---

## ⚙️ Ortam Gereksinimleri

| Bileşen | Versiyon |
|---------|----------|
| TRex | v3.08 |
| Cisco CSR1000v | IOS-XE |
| Python | Python 3 |

---

## 🔧 Cisco CSR1000v Yapılandırması

### DHCP Pool

```ios
ip dhcp pool 1
 network 99.1.1.0 255.255.255.0
 domain-name muji.com
 dns-server 172.16.1.103 172.16.2.10
 lease 30
```

### GigabitEthernet2 Arayüzü

```ios
interface GigabitEthernet2
 ip address 99.1.1.1 255.255.255.0
 load-interval 30
 negotiation auto
 ipv6 address 2001:DB8:1::1/64
 ipv6 enable
 arp timeout 70
 no mop enabled
 no mop sysid
```

---

## 🚀 TRex Başlatma

```bash
cd /v3.08
./trex-console --emu
```

---

## 📋 Test Adımları

### 1. EMU Profili Yükleme

10 istemci içeren DHCP profili yükle:

```
trex> emu_load_profile -f emu/simple_dhcp.py -t --ns 1 --clients 10
```

### 2. İstemci Durumunu Görüntüleme

```
trex> emu_show_all
```

DHCP'den IP alan 10 istemci görüntülenmeli, her birinde `arp`, `dhcp`, `icmp`, `igmp` plugin'leri aktif olmalıdır:

| MAC | IPv4 | Default GW | Plugins |
|-----|------|------------|---------|
| 00:00:00:70:00:01 | 99.1.1.2  | 99.1.1.1 | arp, dhcp, icmp, igmp |
| 00:00:00:70:00:02 | 99.1.1.3  | 99.1.1.1 | arp, dhcp, icmp, igmp |
| 00:00:00:70:00:03 | 99.1.1.4  | 99.1.1.1 | arp, dhcp, icmp, igmp |
| 00:00:00:70:00:04 | 99.1.1.5  | 99.1.1.1 | arp, dhcp, icmp, igmp |
| 00:00:00:70:00:05 | 99.1.1.6  | 99.1.1.1 | arp, dhcp, icmp, igmp |
| 00:00:00:70:00:06 | 99.1.1.7  | 99.1.1.1 | arp, dhcp, icmp, igmp |
| 00:00:00:70:00:07 | 99.1.1.8  | 99.1.1.1 | arp, dhcp, icmp, igmp |
| 00:00:00:70:00:08 | 99.1.1.9  | 99.1.1.1 | arp, dhcp, icmp, igmp |
| 00:00:00:70:00:09 | 99.1.1.10 | 99.1.1.1 | arp, dhcp, icmp, igmp |
| 00:00:00:70:00:0a | 99.1.1.11 | 99.1.1.1 | arp, dhcp, icmp, igmp |

### 3. DHCP Sayaçlarını Doğrulama

Belirli bir istemcinin DHCP handshake sayaçlarını kontrol et:

```
trex> emu_dhcp_show_counters -p 0 --mac 00:00:00:70:00:01
```

Başarılı bir DORA (Discover → Offer → Request → Ack) akışında beklenen çıktı:

```
    name      | value 
--------------+------
pktTxDiscover | 1     
pktRxOffer    | 1     
pktTxRequest  | 1     
pktRxAck      | 1     
pktRxNotify   | 1     
```

---

## 📡 DHCP DORA Akışı

```
İstemci                        CSR1000v (DHCP Server)
   |                                    |
   |-------- DHCP Discover ----------->|   (pktTxDiscover)
   |<-------- DHCP Offer --------------|   (pktRxOffer)
   |-------- DHCP Request ------------>|   (pktTxRequest)
   |<-------- DHCP Ack ----------------|   (pktRxAck)
   |<-------- Notify ------------------|   (pktRxNotify)
   |                                    |
```

---

## 📊 DHCP Sayaç Referansı

| Sayaç | Yön | Açıklama |
|-------|-----|----------|
| `pktTxDiscover` | İstemci → Sunucu | IP isteği yayını |
| `pktRxOffer` | Sunucu → İstemci | Sunucunun IP teklifi |
| `pktTxRequest` | İstemci → Sunucu | Teklifin kabulü |
| `pktRxAck` | Sunucu → İstemci | IP atamasının onayı |
| `pktRxNotify` | Sunucu → İstemci | Lease bildirim mesajı |

---

## 🔍 Faydalı Komutlar

| Komut | Açıklama |
|-------|----------|
| `emu_show_all` | Tüm namespace ve istemci bilgilerini göster |
| `emu_dhcp_show_counters -p 0 --mac <MAC>` | Belirtilen istemcinin DHCP sayaçlarını göster |

---

## 📝 Notlar

- DHCP pool lease süresi **30 gün** olarak ayarlanmıştır.
- DNS sunucuları: `172.16.1.103` ve `172.16.2.10`
- Domain adı: `muji.com`
- Tüm istemciler `99.1.1.1` adresini default gateway olarak kullanmaktadır.
- `pktRxNotify` sayacı, lease bildiriminin başarıyla alındığını gösterir.
