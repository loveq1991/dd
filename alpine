#!/bin/bash
#
# Author: lovekk <admin AT lovekk.org>
# Blog: https://www.lovekk.org/
# Demo: https://16mb.tw/
#

clear
printf "
#######################################################################
#                  16MB Typecho for Alpine Linux v1.0                 #
#       For more information please visit https://www.lovekk.org      #
#                  Demo please visit https://16mb.tw                  #
#######################################################################
"

# Check if user is root
[ $(id -u) != "0" ] && { echo "Error: You must be root to run this script"; exit 1; }

# Get network card
link=$(ls /sys/class/net)
linkStr=''
for i in ${link}; do
    linkStr=${linkStr}","${i}
done
devStr=${linkStr#*,}

# check virutal
while :; do echo
    read -p "Please select your virutal(lxc|openvz): " sys
    if echo {lxc openvz} | grep -w ${sys} &> /dev/null; then
        break
    else
        echo "input error! Please only input 'lxc' or 'openvz'"
    fi
done

# check dev
while :; do echo
    read -p "Please select network card(${devStr}): " dev
    if echo "${link}" | grep -w ${dev} &> /dev/null; then
        break
    else
        echo "input error! Please only input ${devStr}"
    fi
done

# check os
if [ -e /etc/redhat-release ]; then
    yum install wget xz unzip -y
elif [ -n "$(grep 'bian' /etc/issue)" -o "$(lsb_release -is 2>/dev/null)" == "Debian" ]; then
    apt-get install wget xz-utils unzip -y
elif [ -n "$(grep 'Ubuntu' /etc/issue)" -o "$(lsb_release -is 2>/dev/null)" == "Ubuntu" -o -n "$(grep 'Linux Mint' /etc/issue)" ]; then
    apt-get install wget xz-utils unzip -y
else
  echo "${CFAILURE}Does not support this OS, Please contact the author! ${CEND}"
  kill -9 $$
fi

cd /
# download alpine
mkdir -p /lovekk
path=$(wget -O- http://images.linuxcontainers.org/meta/1.0/index-system | grep -v edge | awk '-F;' '($1=="alpine" && $3=="amd64") {print $NF}' | tail -1)
wget --no-check-certificate http://images.linuxcontainers.org${path}rootfs.tar.xz -O ~/rootfs.tar.xz
xz -d ~/rootfs.tar.xz
tar xf ~/rootfs.tar -C /lovekk

# set parameter
sed -i 's/rc_sys="lxc"/rc_sys="${sys}"/' /lovekk/etc/rc.conf

# set network
cat > /lovekk/etc/network/interfaces << EOF
auto lo
iface lo inet loopback

auto $dev
iface $dev inet dhcp

hostname \$(hostname)
EOF
rm -f /lovekk/etc/resolv.conf
cp /etc/resolv.conf /lovekk/etc/resolv.conf

# set passwd
sed -i '/^root:/d' /lovekk/etc/shadow
grep '^root:' /etc/shadow >> /lovekk/etc/shadow

# delete old os
/lovekk/lib/ld-musl-x86_64.so.1 /lovekk/bin/busybox rm -rf `/lovekk/lib/ld-musl-x86_64.so.1 /lovekk/bin/busybox ls / | grep -v "lovekk"`

# copy alpine
/lovekk/lib/ld-musl-x86_64.so.1 /lovekk/bin/busybox cp -a /lovekk/* /

# set export
export PATH="/usr/sbin:/usr/bin:/sbin:/bin"
rm -rf /lovekk

# os update
cat > /etc/apk/repositories << EOF
https://mirrors.ustc.edu.cn/alpine/latest-stable/main
https://mirrors.ustc.edu.cn/alpine/latest-stable/community
EOF
apk update

# install ssh
apk add openssh
echo PermitRootLogin yes >> /etc/ssh/sshd_config
