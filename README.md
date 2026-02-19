Команды 

ssh team01@192.168.10.180

Team02! пароль

FLAG1
Создайте report.txt (3 строки) и получите флаг:

printf "HOST: $(hostname)\nOS: $(cat /etc/os-release | grep PRETTY_NAME | cut -d= -f2-)\nIP: $(hostname -I | awk '{print $1}')\n" > report.txt

quest-check1

FLAG2 (поиск)
find ~/lab -name "flag2.txt" -type f -print -exec cat {} \;

FLAG3 (права)
chmod 644 ~/vault/flag3.txt
cat ~/vault/flag3.txt

FLAG4 (в логах)
grep "CODE:" ~/logs/system.log

FLAG5 (архив)
mkdir -p ~/unpack
tar -xzf ~/pack.tar.gz -C ~/unpack
cat ~/unpack/flag5.txt
