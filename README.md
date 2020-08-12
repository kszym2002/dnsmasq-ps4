# 添加 gituhub 路由
route add -host 151.101.0.133 dev WG接口名称

# 更新 china-dnsmasq

echo "conf-dir=/etc/dnsmasq.d" >> /etc/dnsmasq.conf

wget /etc/dnsmasq.d https://github.com/kszym2002/dnsmasq-ps4/raw/master/cn-ipset.conf

wget /etc/dnsmasq.d https://github.com/kszym2002/dnsmasq-ps4/raw/master/cn.conf

/etc/init.d/dnsmasq restart

# 更新mwan3 chnroute

rm -rf /etc/mwan3helper/all_cn.txt

wget https://raw.githubusercontent.com/17mon/china_ip_list/master/china_ip_list.txt -O /etc/mwan3helper/all_cn.txt

重启mwan3helper


# 不使用mwan3  开机启动脚本
cat > /etc/init.d/chnroute_wireguard <<EOF
#!/bin/sh /etc/rc.common

START=99

start() {
    ip route add 8.8.8.8 dev WG接口名称

    cat /etc/iproute2/rt_tables | grep wgproxy
    if [ \$? -eq 0 ];then
        echo "wgproxy 已存在"
    else
        echo "200 wgproxy" >> /etc/iproute2/rt_tables
    fi

    iptables -t mangle -N fwmark
    iptables -t mangle -C OUTPUT -j fwmark || iptables -t mangle -A OUTPUT -j fwmark
    iptables -t mangle -C PREROUTING -j fwmark || iptables -t mangle -A PREROUTING -j fwmark

    # 本地ip不走代理，非常重要，不然路由器都进不去
    iptables -t mangle -A fwmark -d 0.0.0.0/8 -j RETURN
    iptables -t mangle -A fwmark -d 10.0.0.0/8 -j RETURN
    iptables -t mangle -A fwmark -d 127.0.0.0/8 -j RETURN
    iptables -t mangle -A fwmark -d 169.254.0.0/16 -j RETURN
    iptables -t mangle -A fwmark -d 172.16.0.0/12 -j RETURN
    iptables -t mangle -A fwmark -d 192.168.0.0/16 -j RETURN
    iptables -t mangle -A fwmark -d 224.0.0.0/4 -j RETURN
    iptables -t mangle -A fwmark -d 240.0.0.0/4 -j RETURN

    # 服务器ip/域名不走代理，修改为你自己的服务器地址，非常重要，启动脚本这里也要改
    iptables -t mangle -A fwmark -d 服务器ip -j RETURN

    iptables -t mangle -A fwmark -m set ! --match-set chnroute dst -j MARK --set-mark 0xffff
    ip rule add fwmark 0xffff table wgproxy
    ip route add default dev WG接口名称 table wgproxy
    iptables -I FORWARD -o WG接口名称 -j ACCEPT
    iptables -t nat -I POSTROUTING -o WG接口名称 -j MASQUERADE
}
EOF

# 赋予执行权限

chmod +x /etc/init.d/chnroute_wireguard

# 开机启动

/etc/init.d/chnroute_wireguard enable
