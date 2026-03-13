// Вариант 1
x = 3.11;
y = log(1.5 * x);           // log = ln в Scilab
z = atan(cos(x^2));

disp("Вариант 1:");
disp("y = " + string(y));
disp("z = " + string(z));

// Вариант 2 (с параметром a)
a = 2.5;
y2 = log(14.5 * x);
z2 = atan(cos(a * x^2));

disp("Вариант 2:");
disp("y = " + string(y2));
disp("z = " + string(z2));
