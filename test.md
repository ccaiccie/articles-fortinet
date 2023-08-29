Diagram:
 
![image](https://github.com/ccaiccie/articles-fortinet/assets/20251042/9814959a-e6b3-40a6-b779-31fa9a73db65)

 

The configuration for that requirement in FortiGate is mentioned here.
 
FortiGate port2 is the public-facing interface which connected to the remote device over the internet and port3 is connected internal device.
 
VIP and corresponding firewall policy configuration is needed. So that, it can forward the IKE packets from the remote device to the internal device.
 
IKE service config.
 
Fortigate # config firewall service custom
 
Fortigate (custom) # edit "IKE"
 
Fortigate (IKE) # show
    config firewall service custom
        edit "IKE"
            set category "Tunneling"
            set udp-portrange 500 4500
        next
end
 
Fortigate (IKE) #
 
VIP config.
 
Fortigate # config firewall vip
 
Fortigate (vip) # edit "To-internal"
 
Fortigate (To-internal) # show
    config firewall vip
        edit "To-FW4"
            set uuid 8b3e2fa6-4350-51ee-1149-0f96fd9c92a1
            set service "IKE"
            set extip 172.16.1.2
            set mappedip "172.16.2.2"
            set extintf "port2"
        next
    end
 
Fortigate (To-internal) # 
 
Firewall policy config.
 
Fortigate # config firewall policy
 
Fortigate (policy) # edit "2"
 
Fortigate (2) # show
    config firewall policy
        edit 2
            set name "To-internal"
            set uuid a5f530ec-4350-51ee-9e54-ea52191495e2
            set srcintf "port2"
            set dstintf "port3"
            set action accept
            set srcaddr "all"
            set dstaddr "To-internal"
            set schedule "always"
            set service "IKE"
            set logtraffic all
        next
    end
 
Fortigate (2) # 
 
The second policy is required to pass the traffic from the LAN interface (Internal device) to the WAN interface (Remote device). In that case, a regular LAN-WAN firewall policy is enough:
 
Fortigate # config firewall policy
 
Fortigate (policy) # edit "1"
 
Fortigate (1) # show
    config firewall policy
        edit 1
            set name "Internet"
            set uuid 669f8e24-4350-51ee-ed99-fea711a87459
            set srcintf "port3"
            set dstintf "port2"
            set action accept
            set srcaddr "all"
            set dstaddr "all"
            set schedule "always"
            set service "ALL"
            set logtraffic all
            set nat enable
        next
    end
 
Fortigate (1) #
 
Now, both devices should be able to talk to each other for the IPSec negotiation. As FortiGate is NATTING the traffic, the negotiation should take over UDP port 4500.
