**Вариант 13 готов к выполнению.**

Я буду помогать тебе шаг за шагом. Выполняй команды по порядку в своём терминале Linux.

### Шаг 1. Создать корневой каталог
```bash
mkdir variant13
cd variant13
```

### Шаг 2. Создать структуру каталогов
```bash
mkdir -p office/office/docs office/office/table archive
```

### Шаг 3. Создать файлы
```bash
touch office/office/docs/letter.txt
touch office/office/table/data.txt
```

### Шаг 4. Записать содержимое в файлы
```bash
cat > office/office/docs/letter.txt << EOF
To manager
Meeting at 10
Bring report
EOF

cat > office/office/table/data.txt << EOF
Row1
Row2
Row3
EOF
```

### Шаг 5. Скопировать файл
```bash
cp office/office/docs/letter.txt archive/
```

### Шаг 6. Переименовать файл
```bash
mv office/office/table/data.txt office/office/table/table_data.txt
```

### Шаг 7. Найти слово в файле
```bash
grep "Meeting" office/office/docs/letter.txt
```

### Шаг 8. Подсчитать количество строк
```bash
wc -l office/office/table/table_data.txt
```

### Шаг 9. Показать дерево каталогов (итоговая структура)
```bash
tree
```
или, если `tree` не установлен:
```bash
ls -R
```

---

### Проверка результата (покажи преподавателю):
```bash
pwd
cd ~/variant13          # или где у тебя находится variant13
tree                    # или ls -R
find . -type f
cat office/office/docs/letter.txt
cat office/office/table/table_data.txt
cat archive/letter.txt
grep "Meeting" office/office/docs/letter.txt
wc -l office/office/table/table_data.txt
```

Хочешь, я сейчас покажу тебе, как должна выглядеть итоговая структура, или сразу перейдём к следующему шагу?  
Можешь копировать команды по одной и писать «готов» после каждого шага.
