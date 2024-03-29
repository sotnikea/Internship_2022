# Условие

Написать программу, в которой 2 объекта-счетчика соединены связью сигнал-слот: при изменении значения одного изменяется значение другого. Интерфейс счетчиков:

~~~C++
class Counter {
public:
    Counter(int startValue);
    int Increment();
    int Decrement();

// ... Место для объявления сигналов и слотов

private:
    int m_count;
}
~~~

# Способ проверки
Необходимо показать пример использования счетчиков на следующем тесте:
- Создаем два счетчика с разным стартовым значением
- Вызываем Increment  для первого счетчика
- Выводим значения обеих счетчиков
- Вызываем Increment для второго счетчика
- Выводим значения обеих счетчиков
- Вызываем Decrement  для первого счетчика
- Выводим значения обеих счетчиков
- Вызываем Decrement для второго счетчика
- Выводим значения обеих счетчиков

# Формат сдачи
К отчету прикрепить файлы кода и скриншот выполнения теста.

# Дополнительные условия
Реализация с GUI необязательна, достаточно чтобы счетчики логгировали изменения и причину в std::out.
Ожидаемый результат: код и вывод программы.
Для разработки лучше всего использовать Qt Creator, он скачивается вместе с фреймворком Qt: https://www.qt.io/download.

# Обзор решения
Проект реализован как консольное приложение Qt и содержит следующие файлы:  
- counter.h
- counter.cpp
- main.cpp

Файл counter.h содержит в себе описание класса-счетчика, соответственно counter.cpp содержит его реализацию.  
В файле main.cpp созданы два экземпляра описанного класса, которые соединяются связью сигнал-слот. Дальнейшая часть файла содержит тест взаимодействия экземпляров счетчиков согласно заданного условия.

Т.к. класс-счетчик должен обеспечивать сигнал-слот связь, он должен содержать библиотеку #include `<QObject>`. Сам класс должен наследоваться от `QObject` и содержать его в полях

~~~C++
class Counter : public QObject
{
    Q_OBJECT
private:

public:

}
~~~
Так как оба счетчика могут как подавать сигнал, так и принимать его, класс должен содержать и сигналы, и слоты.  
Для создания слота подключения используется секция `slots`. В качестве слота создаем функцию принимающую некоторую булевую переменную. Булева переменная играет роль флага, определяющего тип операции над счетчиком. Если правда - инкремент, ложь - декремент:
~~~C++
public slots:
    void FromCounterGet(bool operation);
~~~
Для создания сигнала используется секция signals. В качестве сигнала создаем функцию, которая будет передавать булевое значение, указывающее какой тип операции должен выполнить слушатель сигнала (инкрементирования или декрементирование):
~~~C++
signals:
    void ToCounterSet(bool operation);
~~~
Для того, чтобы после изменения значения счетчика, можно было подать сигнал на такое же изменение другому счетчику, в данных функциях используется `emit` с указанием функции-сигнала, которую нужно запустить.
~~~C++
void Counter::Increment(){
    m_value++;
    emit ToCounterSet(true);
}

void Counter::Decrement(){
    m_value--;
    emit ToCounterSet(false);
}
~~~
Для связи счетчиков используется QObject::connect  
Создаем связь, где первый счетчик будет подавать сигнал, а второй принимать его в своем слоте 
~~~C++
QObject::connect(&c1, &Counter::ToCounterSet, &c2, &Counter::FromCounterGet);
~~~
Теперь создаем связь в обратном направлении
~~~C++
QObject::connect(&c2, &Counter::ToCounterSet, &c1, &Counter::FromCounterGet);
~~~

Формируем набор тестов согласно заданного условия. Результат выполнения тестов:  
~~~
--- START VALUE OF COUNTERS ---
Value of 1st counter: 50
Value of 2nd counter: 100
--- INCREMENT 1ST COUNTER ---
Value of 1st counter: 51
Value of 2nd counter: 101
--- INCREMENT 2ND COUNTER ---
Value of 1st counter: 52
Value of 2nd counter: 102
--- DECREMENT 1ST COUNTER ---
Value of 1st counter: 51
Value of 2nd counter: 101
--- DECREMENT 2ND COUNTER ---
Value of 1st counter: 50
Value of 2nd counter: 100
~~~

Как видим, результат полностью удовлетворяет условию задания.

# Оценка результата
100 из 100  
Реализация не требует исправлений
