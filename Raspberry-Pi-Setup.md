### General setup

apt install etckeeper
apt install cron-apt
vi /etc/exim4/update-exim4.conf.conf
# Set:
# dc_eximconfig_configtype='satellite'
# dc_readhost='netfuture.ch'
# dc_smarthost='smtp.netfuture.ch'
update-exim4.conf

### Networking setup

# Also enabled wireless LAN for test:
vi /etc/wpa_supplicant/wpa_supplicant.conf

### GPS setup

raspi-config
# Interfacing options→Serial:
# - Disable serial login shell
# - Keep serial port hardware enabled
apt install gpsd gpsd-clients python-gps
# Rasbperries before 3B use /dev/ttyAMA0, 3B /dev/ttyS0
# On modern OSes, /dev/serial0 symlinks to the correct one
ln -s ttyS0 /dev/gps0
vi /etc/default/gps # set GPSDEVICE to /dev/gps0
systemctl restart gpsd
# Verify with `gpsmon` or `cgps -s`

### PPS setup

# Maybe can use `git clone https://github.com/flok99/rpi_gpio_ntpd.git` sometime in the future…
# For now, it remains:
git clone https://github.com/MarcelWaldvogel/rpi_gpio_ntpd.git
cd rpi_gpio_ntpd
git checkout dont-accidentally-create-shm
make install
# Verify with `./rpi_gpio_ntp -f -N 1 -g 4 -d` (when daemon is not running)
# Note that PPS (rpi_gpio_ntpd) uses the closest full second from the system
# clock as the time fed through SHM to ntpd when a pulse occurs.

### NTP setup

apt install ntpd pps-tools
vi /etc/ntp.conf
# Delete unused pool machines, add some reliable servers
cat >> /etc/ntp.conf << EOF
# Time from GPS (because of serial link off by about 0.5 s)
# For me, using 127.127.28.0 did not work
server 127.127.46.0 iburst minpoll 4
fudge 127.127.46.0 time1 0.573
# Precision second pulse (requires accurate time from other source)
server 127.127.28.1 prefer iburst minpoll 4
fudge 127.127.28.1 refid GPSp
EOF
systemctl restart ntp
# Verify with `ntpq -p`: Should list both `GPSD` and `GPSp` refids;
# after a few seconds, they should also show values

