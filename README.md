 score = int(input("Введите количество баллов: "))

if score >= 90:
    print("Гриффиндор")
elif score >= 70:
    print("Когтевран")
elif score >= 50:
    print("Пуффендуй")
elif score >= 0:
    print("Слизерин")
else:
    print("Некорректное количество баллов")
