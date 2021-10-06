Лекция 3. Функции высшего порядка
=================================

Значения языка Scheme:

* Числа: 1, 1.0, 6.022e23, 1/3...
* Строки: "Scheme"
* Логический тип: #t, #f
* Литерный (character) тип: #\a #\newline ...
* Символьный (symbol) тип: 'x, 'sin...
* ...
* **Процедурный тип**

    (lambda (аргументы) выражение)

Конструкция lambda создаёт безымянную
процедуру. Эту процедуру можно вызвать:

    ((lambda (x y) (+ x y)) 10 13)
    ;         ^--- формальные параметры
    ;фактические параметры --^

При вызове процедуры создаются новые
переменные, соответствующие формальным
параметрам и они связываются с фактическими
параметрами.

    (define f
      (lambda (x y) (+ x y)))
    (f 10 13)

Синтаксический сахар:

    (define f (lambda (парам) выраж))

эквивалентно

    (define (f парам) выраж)

Передача процедуры как параметра:

    (define (g f)
      (f 10 13))

    (g (lambda (x y) (+ x y)))
    (g +)

Возврат процедуры из процедуры

    (define (select n)
      (if (> n 0)
          (lambda (x y) (+ x y))
          (lambda (x y) (- x y))))

    ((select +1) 10 13)
    ((select -1) 100 50)




Управляющие конструкции языка Scheme
====================================

Конструкции let, let* и letrec
------------------------------

    (let ((var1 expr1)
          (var2 expr2)
          ...
          (varN exprN))
      выражение)

В теле let-выражения можно использовать
переменные var1...varN. Выражения expr1...exprN
могут вычисляться в произвольном порядке -
порядок их вычисления не определён.

НО! Внутри expr1...exprN нельзя использовать
var1...varN.

    (let ((x (+ y z)))
       (* x x))


    (let* ((var1 expr1)
           (var2 expr2)
           ...
           (varN exprN))
      body)

Переменную varK можно использовать не только
в теле let*, но и в exprM, где M > K.

Выражения expr1...exprN вычисляются
_последовательно._


     (letrec ((var1 expr1)
              (var2 expr2)
              ...
              (varN exprN))
      body)

Внутри любого exprK можно использовать любую
переменную из var1...varN.

Все эти конструкции являются синтаксическим
сахаром. Для примера:

    (let ((var1 expr1)
          (var2 expr2)
          ...
          (varN exprN))
      выражение)

эквивалентна

    ((lambda (var1 ... varN)
       выражение)
     expr1 ... exprN)


### "let с рекурсией"

    (let proc-name ((var1 expr1)
                    (var2 expr2)
                    ...
                    (varN exprN))
      body)

Внутри тела let-выражения можно вызывать
процедуру proc-name, передавая ей N параметров.

Это выражение эквивалентно

    (letrec ((proc-name
              (lambda (var1 ... varN)
                body)))
      (proc-name expr1 ... exprN))


Рекурсия, итерация и хвостовая рекурсия
=======================================

N! = 1*2*3*...*(N-1)*N

0! = 1
N! = N * (N-1)!

Рекурсия - делим задачу на меньшие подзадачи,
подобные исходной.

Итерация - задача делится на некоторое
количество одинаковых подзадач, одинаковых
шагов, приближающих к цели.

Как итерацию выразить через рекурсию?

Итерация: пока цель не достигнута, повторять
шаг вычисления.

Рекурсия:
* Цель достигнута?
  * Да - прекратить вычисления, вернуть
    результат.
  * Нет - выполнить один шаг вычисления,
    выполнить рекурсивный вызов.

Факториал в терминах итерации:

    int fact(int N) {
      int res = 1;
      int i = 1;
      while (i <= N) {
        res = res * i;
        i = i + 1;
      }
      return res;
    }

* Для цикла заводим вспомогательную процедуру.
* Переменные цикла становятся параметрами
  процедуры.
* Тело цикла превращается в рекурсивный
  вызов.
* Инициализация переменных цикла становится
  вызовом рекурсивной процедуры.

    (define (fact N)
      (define (loop i res)
        (if (<= i N)
            (loop (+ i 1)
                  (* res i))
            res))
      (loop 1 1))

    (define (fact N)
      (let loop ((i 1)
                 (res 1))
        (if (<= i N)
            (loop (+ i 1)
                  (* res i))
            res)))

Хвостовая рекурсия
------------------

Хвостовой вызов - вызов, который является
последним, результат этого вызова становится
результатом работы функции.

    (define (f x y z)
      (if (a)
          (b x (c y))
          (d (if (e)
                 (g)
                 (h)))))

(Вызовы b и d - хвостовые)

В языке Scheme заложена оптимизация хвостового
вызова, т.н. оптимизация хвостовой рекурсии.
Фрейм стека (см. лекцию про продолжения)
вызывающей процедуры замещается фреймом стека
вызываемой процедуры.

Если хвостовой вызов является рекурсивным,
фреймы стека не накапливаются.

**Хвостовая рекурсия в языке Scheme эквивалента
итерации** по вычислительным затратам.

Рекурсивный факториал:

    (define (fact N)
      (if (> N 0)
          (* (fact (- N 1)) N)
          1))

Итеративный факториал:

    (define (fact N)
      (define (loop i res)
        (if (<= i N)
            (loop (+ i 1)
                  (* res i))
            res))
      (loop 1 1))


Оптимизация хвостовой рекурсии изнутри:

    int loop(int N, int i, int res) {
      if (i <= N) {
        loop(N, i + 1, res * i);
      } else {
        return res;
      }
    }

    int loop(int N, int i, int res) {
    LOOP:
      if (i <= N) {
        res = res * i;
        i = i + 1;
        goto LOOP;
      } else {
        return res;
      }
    }