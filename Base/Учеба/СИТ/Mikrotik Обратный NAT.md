![[Pasted image 20260206125328.png]]
Пример

1. Делаем DST-NAT
	1. src. Address - внутренняя сеть/24 (пример 192.168.0.0/24)
	2. Dst. Address - белый ip
	3. Protocol - tcp
	4. dst. Port  - порт который указывается в адресе при подключении
	5. Action - dst-nat
	6. To Addresses - адрес внутренний устройства
	7. To Ports - в нашем случае 3389 порт RDP
2. Делаем masquerade
	1. src. Address - внутренняя сеть/24
	2. Dst. Address - адрес внутренний устройства
	3. Protocol - tcp
	4. Action - masquerade
профит