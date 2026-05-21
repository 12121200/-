mkdir -p \~/shell_scripts && cd \~/shell_scripts

cat > hello_user.sh << 'EOF'
#!/bin/bash
echo "Привет, $1!"
EOF
chmod +x hello_user.sh

bash -x hello_user.sh Студент
./hello_user.sh Студент

cat > greet.sh << 'EOF'
#!/bin/bash
echo "Здравствуйте, $1!"
name=$1
echo "Переменная: $name"
EOF
chmod +x greet.sh
./greet.sh Иван

cat > check_dir.sh << 'EOF'
#!/bin/bash
[ -d \~/reports ] || mkdir \~/reports && echo "Папка создана"
EOF
chmod +x check_dir.sh
./check_dir.sh

cat > reports.sh << 'EOF'
#!/bin/bash
mkdir -p \~/reports
for i in {1..5}; do
  echo "Отчёт $i" > \~/reports/report$i.txt
done
ls \~/reports
EOF
chmod +x reports.sh
./reports.sh

less \~/.bashrc
# или
nano \~/.bashrc
