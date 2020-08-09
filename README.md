# 更新china-dnsmasq

echo "conf-dir=/etc/dnsmasq.d" >> /etc/dnsmasq.conf

wget /etc/dnsmasq.d https://raw.githubusercontent.com/kszym2002/dnsmasq-ps4/master/cn-ipset.conf

wget /etc/dnsmasq.d https://raw.githubusercontent.com/kszym2002/dnsmasq-ps4/master/cn.conf

# 更新mwan3 chnroute

wget https://raw.githubusercontent.com/17mon/china_ip_list/master/china_ip_list.txt -O ./etc/mwan3helper/all_cn.txt
