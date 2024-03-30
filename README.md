# qemu network-config

host config:
  #!/bin/sh
  ip link add name br0 type bridge
  ip addr add 192.168.100.1/24 brd + dev br0
  ip link set br0 up
  开启 ip_forward
  sysctl -w net.ipv4.ip_forward=1
  允许对从 br0 流入的数据包进行 FORWARD
  iptables -t filter -A FORWARD -i br0 -j ACCEPT
  iptables -t filter -A FORWARD -o br0 -j ACCEPT
  也可以直接将 filter FORWARD 策略直接设置为 ACCEPT
  iptables -t filter -P FORWARD ACCEPT
  开启 NAT
  iptables -t nat -A POSTROUTING -o enp4s0 -j MASQUERADE
  
add "allow br0" into host's /etc/qemu/bridge.conf

qemu guest config:
  ip addr add 192.168.100.2/24 brd + dev ens3
  ip route add default via 192.168.100.1 dev ens3
qemu /etc/network/interfaces:
  auto eth0
  iface eth0 inet static
	  address 192.168.100.2
	  netmask 255.255.255.0
	  gateway 192.168.100.1

start qemu command:
  sudo qemu-system-x86_64 \
	  -m 2048 \
	  -boot order=d \
	  -append "console=ttyS0 root=/dev/sda earlyprintk=serial net.ifnames=0" \
	  -kernel /home/zq/linux-6.7-rc5/arch/x86/boot/bzImage \
	  -drive file=/home/zq/fs_img/bullseye.img,format=raw \
	  -nic bridge,br=br0,model=e1000,mac=02:76:7d:d7:1e:3f \
	  -nographic \
	  -pidfile vm.pid \
   	  2>&1 | tee vm.log

result:
 `qemu and host can communicate with each other, but qemu can't access internet because "Temporary failure in name resolution" `

