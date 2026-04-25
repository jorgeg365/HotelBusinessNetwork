# Detailed Build Steps (From Session)

This guide documents the final working implementation sequence used in the Packet Tracer lab.

## 1. Device Inventory

- CORE router: Cisco 2911
- OFFICE router: Cisco 2811 (CME host)
- REST router: Cisco 1941
- GUEST router: Cisco 1941
- ISP router: Cisco 1941
- SW-OFFICE: Cisco 2960
- SW-REST: Cisco 2960
- SW-GUEST: Cisco 2960
- Endpoints: departmental PCs, restaurant POS PCs, guest devices, and IP phones

## 2. Logical Design

### Department VLANs

- VLAN 10: Front Desk, 192.168.10.0/24
- VLAN 20: IT, 192.168.20.0/24
- VLAN 30: Guest Data, 192.168.30.0/24
- VLAN 40: Restaurant 1 POS, 192.168.40.0/24
- VLAN 50: Restaurant 2 POS, 192.168.50.0/24
- VLAN 70: Sales and Marketing, 192.168.70.0/24
- VLAN 80: Accounting, 192.168.80.0/24
- VLAN 90: HR, 192.168.90.0/24

### Voice VLANs

- VLAN 150: Office Voice, 192.168.150.0/24
- VLAN 160: Restaurant Voice, 192.168.160.0/24
- VLAN 170: Guest-Area Voice, 192.168.170.0/24

## 3. Core Router Links

- CORE <-> OFFICE
- CORE <-> REST
- CORE <-> GUEST
- CORE <-> ISP

The exact physical interface names can vary depending on installed modules in Packet Tracer.

## 4. Branch Router-on-a-Stick

Configure subinterfaces on branch routers for their local VLANs.

### OFFICE router

- Data subinterfaces for VLAN 10, 20, 70, 80, 90
- Voice subinterface for VLAN 150

### REST router

- Data subinterfaces for VLAN 40, 50
- Voice subinterface for VLAN 160

### GUEST router

- Data subinterface for VLAN 30
- Voice subinterface for VLAN 170

## 5. OSPF

Enable OSPF process 1 on CORE, OFFICE, REST, and GUEST.

Advertise:

- Inter-router transit networks
- Local data VLAN networks
- Voice VLAN networks

Verify with:

- show ip ospf neighbor
- show ip route

## 6. DHCP

Create DHCP pools on each router for locally attached VLANs.

Voice pools must include Option 150 pointing to CME:

- option 150 ip 192.168.150.1

Verify with:

- show ip dhcp binding

## 7. Switch Configuration

### Common

- Create required VLANs
- Configure uplink to router as trunk on fa0/24

### Office switch ports

- Phone access ports: data VLAN + voice VLAN 150
- Department PCs mapped to their VLANs

### Restaurant switch ports

- POS ports mapped to VLAN 40/50
- Phone ports configured with voice VLAN 160

### Guest switch ports

- Guest data ports mapped to VLAN 30
- Phone ports configured with voice VLAN 170

## 8. CME (VoIP) on OFFICE 2811

Configure telephony service and DNs.

- ip source-address 192.168.150.1 port 2000
- max-ephones and max-dn increased as needed

Extension example set used:

- 1001, 1002 (Office)
- 1003, 1004 (Restaurant)
- 1005, 1006 (Guest area)

Map DNs to ephones and validate registration:

- show ephone
- show ephone-dn

## 9. Routing to ISP and Return Paths

Configure upstream routeing so internal networks can reach ISP and receive return traffic.

When full NAT behavior was constrained by module/interface simulator behavior, explicit return routes on ISP were used for internal ranges.

## 10. Validation Sequence

1. Verify interface status (show ip interface brief)
2. Verify OSPF neighbors and learned routes
3. Verify DHCP address assignment per VLAN
4. Verify endpoint to gateway pings
5. Verify inter-VLAN and inter-site pings
6. Verify ISP reachability from office, restaurant, and guest endpoints
7. Verify all phones register and complete calls across sites

## 11. Troubleshooting Patterns Used

- Wrong cable type can hide expected interfaces in Packet Tracer UI
- Voice phones require switchport voice vlan on access port
- Phones stuck on Configuring CM List usually indicate missing reachability or DHCP Option 150 issue
- Duplicate or overlapping voice subnets across separate routed domains breaks registration
- Unregistered ephone with 0.0.0.0 points to DHCP/VLAN path issue before CME mapping

## 12. Outcome

Completed a multi-site hotel business network with:

- Enterprise VLAN segmentation
- OSPF dynamic routing
- Multi-site VoIP on centralized CME
- Department/POS/guest separation
- End-to-end tested cross-site phone communication
