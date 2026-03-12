/ip firewall filter
%% KNOCKER %%
add chain=input action=add-src-to-address-list protocol=icmp address-list=knockstage1 address-list-timeout=15s packet-size=302 log=no log-prefix=""
add chain=input action=add-src-to-address-list protocol=icmp src-address-list=knockstage1 address-list=knockstage2 address-list-timeout=15s packet-size=83 log=no log-prefix=""
add chain=input action=add-src-to-address-list protocol=icmp src-address-list=knockstage2 address-list=knocksafe address-list-timeout=1h packet-size=900 log=no log-prefix=""
%% PPTP forward %%
add action=accept chain=forward port=1723 protocol=tcp
%% drop dns request from WAN %%
add action=drop chain=input comment="drop dns" dst-port=53 in-interface-list=\
    WAN protocol=udp
%% L2TP IPSec access %%    
add action=accept chain=input in-interface-list=WAN port=\
    1701,500,4500 protocol=udp
   %% allow ipsec-esp input (to mikrotik)%% 
add chain=input action=accept protocol=ipsec-esp
%% allow gre-tunnel forward mikrotik %%
add action=accept chain=forward in-interface-list=WAN protocol=gre
%% allow gre-tunnel input (to mikrotik)%% 
add action=accept chain=input in-interface-list=WAN protocol=gre
add action=accept chain=input dst-port=1723 in-interface-list=WAN protocol=\
    tcp
dd action=accept chain=input in-interface-list=WAN protocol=icmp src-address=185.216.16.162
add action=accept chain=input in-interface-list=!WAN protocol=icmp
add action=accept chain=input connection-state=established in-interface-list=\
    WAN
add action=accept chain=input connection-state=related in-interface-list=WAN
add action=accept chain=forward in-interface-list=LAN
add action=drop chain=input connection-nat-state=!srcnat,dstnat \
    connection-state=new in-interface-list=WAN
add action=drop chain=forward connection-nat-state=!srcnat,dstnat \
    connection-state=new in-interface-list=WAN
add action=drop chain=input connection-state=invalid in-interface-list=WAN
add action=drop chain=forward connection-state=invalid in-interface-list=WAN
add action=accept chain=input dst-port=8291 protocol=tcp src-address=185.216.16.162 place-before 0
