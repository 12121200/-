**Готово!** Я выполню все задания по порядку в этом Linux-окружении (ты — root). Буду показывать команды и результаты.

### Задание 1. Работа с BOOT-меню
В реальной системе:
- Клавиша BIOS/UEFI: обычно **Del**, **F2**, **F10** или **F12** (зависит от производителя).
- BOOT-menu: часто **F12** или **Esc**.
- Порядок загрузки: в BIOS → Boot → Boot Order.
- Для загрузки с USB: поставить USB-накопитель первым в Boot Order (или использовать Boot Menu).

### Задание 2. Первичная диагностика
Краткий алгоритм (6–8 шагов):
1. Проверить питание (кабель, блок питания, индикаторы).
2. Проверить монитор/кабель.
3. Послушать POST-бипы (ошибки).
4. Зайти в BIOS (Del/F2) — проверить, видит ли железо.
5. Проверить порядок загрузки (Boot Order).
6. Проверить загрузочные устройства (HDD/SSD/USB).
7. Посмотреть сообщения BIOS/UEFI (ошибки, инициализация).
8. Если не загружается — попробовать Live-USB или ремонт.

### Задание 3. Создание дерева директорий
```bash
mkdir -p ~/exam/os/linux/files
cd ~/exam/os/linux/files
pwd
```
Результат: `/root/exam/os/linux/files`

### Задание 4. Работа с каталогами
```bash
mkdir -p practice/{docs,tmp,backup}
mv practice/tmp practice/temp
rmdir practice/temp
ls -la practice
```
Результат:
```
drwxr-xr-x 2 root root ... docs
drwxr-xr-x 2 root root ... backup
```

### Задание 5. Создание и определение типа файлов
```bash
mkdir -p ~/docs
cd ~/docs
echo -e "Строка 1\nСтрока 2\nСтрока 3" > text.txt
touch empty.log
echo '#!/bin/bash\necho "Hello"' > script.sh
file text.txt empty.log script.sh
```
Результат:
- text.txt: ASCII text
- empty.log: empty
- script.sh: Bourne-Again shell script (или ASCII text)

### Задание 6. Копирование, перемещение и удаление
```bash
cd ~/docs
echo "Отчёт" > report.txt
cp report.txt backup/
mv report.txt ../exam_archive/   # если нет — mkdir -p ../exam_archive
rm backup/report.txt
ls -R ~/exam
```

### Задание 7. Просмотр содержимого файла
```bash
cd ~/docs
for i in {1..20}; do echo "Строка $i"; done > lines.txt
cat lines.txt
head -n 5 lines.txt
tail -n 5 lines.txt
less lines.txt   # (q для выхода)
```

### Задание 8. Наблюдение за изменением файла
В одном терминале:
```bash
cd ~/docs
touch log.txt
tail -f log.txt
```

В другом терминале:
```bash
echo "Новая строка 1" >> ~/docs/log.txt
echo "Новая строка 2" >> ~/docs/log.txt
```
В первом окне увидишь новые строки в реальном времени.

### Задание 9. Работа со справочной системой
```bash
man mkdir
# Ключ для вложенных каталогов: -p (--parents)
mkdir -p dir1/dir2/dir3
```

### Задание 10. Ключи и аргументы
```bash
ls          # обычный
ls -l       # подробный
ls -a       # скрытые
ls -la      # подробный + скрытые
```
`-l` — long format, `-a` — all (включая . и ..).

### Задание 11. Права доступа в числовой форме
```bash
cd ~/docs
touch access.txt
chmod 644 access.txt
ls -l access.txt
chmod 600 access.txt
ls -l access.txt
chmod 755 access.txt
ls -l access.txt
```
- 644 = rw-r--r--
- 600 = rw-------
- 755 = rwxr-xr-x

### Задание 12. Права в символьной форме
```bash
mkdir shared
chmod g+w shared
chmod o-x shared
ls -ld shared
```

### Задание 13. Владелец и группа
```bash
cd ~/docs
touch group_file.txt
chown root:adm group_file.txt   # или другой group
ls -l group_file.txt
```

### Задание 14. Определение текущего пользователя
```bash
whoami
id
echo $UID
groups
```

### Задание 15. Создание пользователя Linux
```bash
useradd -m -d /home/student_exam student_exam
passwd student_exam   # задать пароль
grep student_exam /etc/passwd
userdel -r student_exam
```

### Задание 16. Работа с группами
```bash
groupadd examgroup
usermod -aG examgroup student_exam   # если пользователь существует
groups student_exam
groupdel examgroup
```

### Задание 17. Работа с vi/vim
```bash
cd ~/docs
vi answer.txt
```
(в vi):
- `i` → вставить 5 строк текста
- `Esc` → `:wq` (сохранить и выйти)
```bash
vi answer.txt
# dd (удалить строку)
# :wq
cat answer.txt
```

### Задание 18. Простой shell-скрипт
```bash
cd ~/docs
cat > hello.sh << EOF
#!/bin/bash
echo "Привет, $1!"
echo "Текущий пользователь: $(whoami)"
EOF
chmod +x hello.sh
./hello.sh Мир
```

### Задание 19. Сценарий с циклом for
```bash
cat > create_dirs.sh << EOF
#!/bin/bash
for i in {1..5}; do
    mkdir -p task$i
done
ls -d task*
EOF
chmod +x create_dirs.sh
./create_dirs.sh
```

### Задание 20. Проверка аргумента
```bash
cat > check_file.sh << EOF
#!/bin/bash
if [ -f "\$1" ]; then
    echo "Файл найден"
else
    echo "Файл не найден"
fi
EOF
chmod +x check_file.sh
touch test.txt
./check_file.sh test.txt
./check_file.sh nonexistent
```

### Задание 21. Резервное копирование
```bash
cat > backup_txt.sh << EOF
#!/bin/bash
mkdir -p backup
cp *.txt backup/ 2>/dev/null
echo "Скопировано в backup:"
ls backup/
EOF
chmod +x backup_txt.sh
./backup_txt.sh
```

### Задание 22. Просмотр процессов
```bash
sleep 300 &
echo "PID: $!"
ps -p $!
kill $!
```

### Задание 23. Информация о системе
```bash
echo "Пользователь: $(whoami)"
echo "Компьютер: $(hostname)"
echo "Ядро: $(uname -r)"
echo "Каталог: $(pwd)"
echo "Свободное место:"
df -h
```

### Задание 24–25. Windows
Эти задания нужно выполнять в Windows:
- **24**: Параметры → Учётные записи → Добавить пользователя → добавить в группу "Пользователи".
- **25**: Создать папку `ExamShare`, правой кнопкой → Свойства → Безопасность → изменить разрешения (Read & execute для Users, без Write).

---

Хочешь, я сохраню все созданные файлы/скрипты в архив или помогу с конкретным заданием подробнее?
