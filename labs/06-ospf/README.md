# OSPF filtering

![](scheme.png)

## Description

- R14 and R15 in area 0 - backbone;
- R12 and R13 in area 10 must have all routes plus default route;
- R19 in area 101 and get only default route;
- R20 in area 102 and get all of the routes except routes to area 101;
- SW4 and SW5 (area 10) are also participate in routing, so ospf process will be running on both switches

### OSPF on R14 and R15 (area 0)
```
router ospf 1
 router-id 14.14.14.14
 area 10 stub
 area 101 stub no-summary
 network 10.10.15.0 0.0.0.3 area 101
 network 10.10.15.4 0.0.0.3 area 10                          
 network 10.10.15.8 0.0.0.3 area 10
 network 10.16.0.0 0.0.0.3 area 0
 network 50.50.23.0 0.0.0.255 area 0
```
```
router ospf 1
 router-id 15.15.15.15
 area 10 stub
 area 102 stub
 area 102 filter-list prefix no_area101_prefixes in
 network 10.10.15.12 0.0.0.3 area 10
 network 10.10.15.16 0.0.0.3 area 10
 network 10.10.15.20 0.0.0.3 area 102
 network 10.16.0.0 0.0.0.3 area 0
 network 60.60.154.0 0.0.0.255 area 0
!
!
no ip http server
!
ip route 0.0.0.0 0.0.0.0 60.60.154.1
!
!
ip prefix-list no_area101_prefixes seq 5 deny 10.10.15.0/30
```

### R14 routing table
```
R14#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 50.50.23.1 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 15 subnets, 2 masks
O        10.10.15.12/30 [110/20] via 10.10.15.6, 05:46:24, Ethernet0/0
O        10.10.15.16/30 [110/20] via 10.10.15.10, 05:46:24, Ethernet0/1
O IA     10.10.15.20/30 [110/20] via 10.16.0.2, 00:08:20, Ethernet1/0
O        10.20.0.12/30 [110/20] via 10.10.15.6, 05:46:34, Ethernet0/0
O        10.30.0.12/30 [110/20] via 10.10.15.6, 05:46:34, Ethernet0/0
O        10.40.0.12/30 [110/20] via 10.10.15.10, 05:46:24, Ethernet0/1
O        10.50.0.12/30 [110/20] via 10.10.15.10, 05:46:24, Ethernet0/1
      60.0.0.0/24 is subnetted, 1 subnets
O        60.60.154.0 [110/20] via 10.16.0.2, 05:46:39, Ethernet1/0
```
### R15 routing table
```
R15#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 60.60.154.1 to network 0.0.0.0

      10.0.0.0/8 is variably subnetted, 15 subnets, 2 masks
O IA     10.10.15.0/30 [110/20] via 10.16.0.1, 00:10:00, Ethernet1/0
O        10.10.15.4/30 [110/20] via 10.10.15.14, 00:10:00, Ethernet0/1
O        10.10.15.8/30 [110/20] via 10.10.15.18, 00:10:00, Ethernet0/0
O        10.20.0.12/30 [110/20] via 10.10.15.14, 00:10:00, Ethernet0/1
O        10.30.0.12/30 [110/20] via 10.10.15.14, 00:10:00, Ethernet0/1
O        10.40.0.12/30 [110/20] via 10.10.15.18, 00:10:00, Ethernet0/0
O        10.50.0.12/30 [110/20] via 10.10.15.18, 00:10:00, Ethernet0/0
      50.0.0.0/24 is subnetted, 1 subnets
O        50.50.23.0 [110/20] via 10.16.0.1, 00:10:00, Ethernet1/0
```

### OSPF on R12 and R13 (area 10)
```
router ospf 1
 router-id 12.12.12.12
 area 10 stub
 network 10.10.15.4 0.0.0.3 area 10
 network 10.10.15.12 0.0.0.3 area 10
 network 10.20.0.12 0.0.0.3 area 10
 network 10.30.0.12 0.0.0.3 area 10
```
```
router ospf 1
 router-id 13.13.13.13
 area 10 stub
 network 10.10.15.8 0.0.0.3 area 10
 network 10.10.15.16 0.0.0.3 area 10
 network 10.40.0.12 0.0.0.3 area 10
 network 10.50.0.12 0.0.0.3 area 10
```

### R12 routing table
```
R12#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 10.10.15.13 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 10.10.15.13, 05:55:31, Ethernet0/3
                [110/11] via 10.10.15.5, 05:55:36, Ethernet0/2
      10.0.0.0/8 is variably subnetted, 15 subnets, 2 masks
O IA     10.10.15.0/30 [110/20] via 10.10.15.5, 05:55:36, Ethernet0/2
O        10.10.15.8/30 [110/20] via 10.10.15.5, 05:55:36, Ethernet0/2
O        10.10.15.16/30 [110/20] via 10.10.15.13, 05:55:31, Ethernet0/3
O IA     10.10.15.20/30 [110/20] via 10.10.15.13, 00:17:23, Ethernet0/3
O IA     10.16.0.0/30 [110/20] via 10.10.15.13, 05:55:31, Ethernet0/3
                      [110/20] via 10.10.15.5, 05:55:36, Ethernet0/2
O        10.40.0.12/30 [110/30] via 10.10.15.13, 05:55:31, Ethernet0/3
                       [110/30] via 10.10.15.5, 05:55:31, Ethernet0/2
O        10.50.0.12/30 [110/30] via 10.10.15.13, 05:55:31, Ethernet0/3
                       [110/30] via 10.10.15.5, 05:55:31, Ethernet0/2
      50.0.0.0/24 is subnetted, 1 subnets
O IA     50.50.23.0 [110/20] via 10.10.15.5, 05:55:36, Ethernet0/2
      60.0.0.0/24 is subnetted, 1 subnets
O IA     60.60.154.0 [110/20] via 10.10.15.13, 05:55:31, Ethernet0/3
```

### R13 routing table
```
R13#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 10.10.15.17 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 10.10.15.17, 05:56:50, Ethernet0/2
                [110/11] via 10.10.15.9, 05:56:45, Ethernet0/3
      10.0.0.0/8 is variably subnetted, 15 subnets, 2 masks
O IA     10.10.15.0/30 [110/20] via 10.10.15.9, 05:56:45, Ethernet0/3
O        10.10.15.4/30 [110/20] via 10.10.15.9, 05:56:45, Ethernet0/3
O        10.10.15.12/30 [110/20] via 10.10.15.17, 05:56:50, Ethernet0/2
O IA     10.10.15.20/30 [110/20] via 10.10.15.17, 00:18:40, Ethernet0/2
O IA     10.16.0.0/30 [110/20] via 10.10.15.17, 05:56:50, Ethernet0/2
                      [110/20] via 10.10.15.9, 05:56:45, Ethernet0/3
O        10.20.0.12/30 [110/30] via 10.10.15.17, 05:56:50, Ethernet0/2
                       [110/30] via 10.10.15.9, 05:56:45, Ethernet0/3
O        10.30.0.12/30 [110/30] via 10.10.15.17, 05:56:50, Ethernet0/2
                       [110/30] via 10.10.15.9, 05:56:45, Ethernet0/3
      50.0.0.0/24 is subnetted, 1 subnets
O IA     50.50.23.0 [110/20] via 10.10.15.9, 05:56:45, Ethernet0/3
      60.0.0.0/24 is subnetted, 1 subnets
O IA     60.60.154.0 [110/20] via 10.10.15.17, 05:56:50, Ethernet0/2
```

### OSPF on SW4-L3 and SW5-L3 (area 10)
```
router ospf 1
 router-id 4.4.4.4
 area 10 stub
 network 10.20.0.12 0.0.0.3 area 10
 network 10.50.0.12 0.0.0.3 area 10
```
```
router ospf 1
 router-id 5.5.5.5
 area 10 stub
 network 10.30.0.12 0.0.0.3 area 10
 network 10.40.0.12 0.0.0.3 area 10
```

### SW4 routing table
```
SW4#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 10.50.0.13 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/21] via 10.50.0.13, 00:00:32, Ethernet1/1
                [110/21] via 10.20.0.14, 00:00:32, Ethernet1/0
      10.0.0.0/8 is variably subnetted, 13 subnets, 2 masks
O IA     10.10.15.0/30 [110/30] via 10.50.0.13, 00:00:32, Ethernet1/1
                       [110/30] via 10.20.0.14, 00:00:32, Ethernet1/0
O        10.10.15.4/30 [110/20] via 10.20.0.14, 00:00:32, Ethernet1/0
O        10.10.15.8/30 [110/20] via 10.50.0.13, 00:00:32, Ethernet1/1
O        10.10.15.12/30 [110/20] via 10.20.0.14, 00:00:32, Ethernet1/0
O        10.10.15.16/30 [110/20] via 10.50.0.13, 00:00:32, Ethernet1/1
O IA     10.10.15.20/30 [110/30] via 10.50.0.13, 00:00:32, Ethernet1/1
                        [110/30] via 10.20.0.14, 00:00:32, Ethernet1/0
O IA     10.16.0.0/30 [110/30] via 10.50.0.13, 00:00:32, Ethernet1/1
                      [110/30] via 10.20.0.14, 00:00:32, Ethernet1/0
O        10.30.0.12/30 [110/20] via 10.20.0.14, 00:00:32, Ethernet1/0
O        10.40.0.12/30 [110/20] via 10.50.0.13, 00:00:32, Ethernet1/1
      50.0.0.0/24 is subnetted, 1 subnets
O IA     50.50.23.0 [110/30] via 10.50.0.13, 00:00:32, Ethernet1/1
                    [110/30] via 10.20.0.14, 00:00:32, Ethernet1/0
      60.0.0.0/24 is subnetted, 1 subnets
O IA     60.60.154.0 [110/30] via 10.50.0.13, 00:00:32, Ethernet1/1
                     [110/30] via 10.20.0.14, 00:00:32, Ethernet1/0
```

### SW5 routing table
```
SW5#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 10.40.0.13 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/21] via 10.40.0.13, 00:01:17, Ethernet1/0
                [110/21] via 10.30.0.14, 00:01:17, Ethernet1/1
      10.0.0.0/8 is variably subnetted, 13 subnets, 2 masks
O IA     10.10.15.0/30 [110/30] via 10.40.0.13, 00:01:17, Ethernet1/0
                       [110/30] via 10.30.0.14, 00:01:17, Ethernet1/1
O        10.10.15.4/30 [110/20] via 10.30.0.14, 00:01:17, Ethernet1/1
O        10.10.15.8/30 [110/20] via 10.40.0.13, 00:01:17, Ethernet1/0
O        10.10.15.12/30 [110/20] via 10.30.0.14, 00:01:17, Ethernet1/1
O        10.10.15.16/30 [110/20] via 10.40.0.13, 00:01:17, Ethernet1/0
O IA     10.10.15.20/30 [110/30] via 10.40.0.13, 00:01:17, Ethernet1/0
                        [110/30] via 10.30.0.14, 00:01:17, Ethernet1/1
O IA     10.16.0.0/30 [110/30] via 10.40.0.13, 00:01:17, Ethernet1/0
                      [110/30] via 10.30.0.14, 00:01:17, Ethernet1/1
O        10.20.0.12/30 [110/20] via 10.30.0.14, 00:01:17, Ethernet1/1
O        10.50.0.12/30 [110/20] via 10.40.0.13, 00:01:17, Ethernet1/0
      50.0.0.0/24 is subnetted, 1 subnets
O IA     50.50.23.0 [110/30] via 10.40.0.13, 00:01:17, Ethernet1/0
                    [110/30] via 10.30.0.14, 00:01:17, Ethernet1/1
      60.0.0.0/24 is subnetted, 1 subnets
O IA     60.60.154.0 [110/30] via 10.40.0.13, 00:01:17, Ethernet1/0
                     [110/30] via 10.30.0.14, 00:01:17, Ethernet1/1
```

### OSPF on R19 (area 101) and R20 (area 102)
```
router ospf 1
 router-id 19.19.19.19
 area 101 stub
 network 10.10.15.0 0.0.0.3 area 101
```
```
router ospf 1
 router-id 20.20.20.20
 area 102 stub
 network 10.10.15.20 0.0.0.3 area 102
```

### R19 routing table
```
R19#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

Gateway of last resort is 10.10.15.2 to network 0.0.0.0

O*IA  0.0.0.0/0 [110/11] via 10.10.15.2, 01:56:09, Ethernet0/0
```

### R20 routing table
```
R20#sh ip route ospf
Codes: L - local, C - connected, S - static, R - RIP, M - mobile, B - BGP
       D - EIGRP, EX - EIGRP external, O - OSPF, IA - OSPF inter area
       N1 - OSPF NSSA external type 1, N2 - OSPF NSSA external type 2
       E1 - OSPF external type 1, E2 - OSPF external type 2
       i - IS-IS, su - IS-IS summary, L1 - IS-IS level-1, L2 - IS-IS level-2
       ia - IS-IS inter area, * - candidate default, U - per-user static route
       o - ODR, P - periodic downloaded static route, H - NHRP, l - LISP
       + - replicated route, % - next hop override

O*IA  0.0.0.0/0 [110/11] via 10.10.15.21, 00:02:20, Ethernet0/0
      10.0.0.0/8 is variably subnetted, 12 subnets, 2 masks
O IA     10.10.15.4/30 [110/30] via 10.10.15.21, 00:01:25, Ethernet0/0
O IA     10.10.15.8/30 [110/30] via 10.10.15.21, 00:01:25, Ethernet0/0
O IA     10.10.15.12/30 [110/20] via 10.10.15.21, 00:01:25, Ethernet0/0
O IA     10.10.15.16/30 [110/20] via 10.10.15.21, 00:01:25, Ethernet0/0
O IA     10.16.0.0/30 [110/20] via 10.10.15.21, 00:01:25, Ethernet0/0
O IA     10.20.0.12/30 [110/30] via 10.10.15.21, 00:01:25, Ethernet0/0
O IA     10.30.0.12/30 [110/30] via 10.10.15.21, 00:01:25, Ethernet0/0
O IA     10.40.0.12/30 [110/30] via 10.10.15.21, 00:01:25, Ethernet0/0
```