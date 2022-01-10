# Прототипы и наследование

Прототипное наследование в JavaScript одна из ключевых особенностей языка, отличающая его от большинаства других языков.

Каждый объект это набор пар ключ значение и в каждом объект есть одно скрытое свойство `[[Prototype]]` которое либо равно null либо ссылается на другой объект, который назвается прототипом.

Когда мы обращаемся к свойству объекта, JavaScript будет искать это свойство в самом объекте и если не найдет его там, то будет искать его в прототипе объекта. Такой механизм называется протоnипным наследованием. Эта особенность позволяет для множества объект записаывать свойства не в каждом объекте занимая память, а записать его в одном объекте, назначив его прототипом для объектов потомков.

Свойство `[[Prototype]]` является внутренним и скрытым, но его можно задать несколькими способами, один из них использовать `__proto__`.

```js
    let animal = {
        walk() {
            return "Walk";
        }
    };

    let cat = {
        sleep() {
            return "Sleep"
        }
    };

    cat.__proto__ = animal;

    cat.sleep();
    cat.walk();

    // Sleep
    // Walk
```

Свойство `__proto__` является геттером и сеттером для внутреннего [слота](https://262.ecma-international.org/6.0/#sec-object-internal-methods-and-internal-slots) `[[Prototype]]` и находится в `Object.prototype`. Он существует по историческим причинам, в современном стандарте языка его заменяют функции `Object.getPrototypeOf`/`Object.setPrototypeOf`, которые также получают/устанавливают прототип.

![](https://github.com/6rost9/turbo-octo-umbrella/blob/main/images/object_prototype.png?raw=true)

### Только для чтения

Прототип используется только для чтения свойств, операции записи или удлаения свойств работают с объектом напряимую.

```js
    let animal = {
        walk() {
            return "Walk";
        }
    };

    let cat = {
        sleep() {
            return "Sleep"
        }
    };

    cat.__proto__ = animal;
    cat.walk = function() {
        return "Own Walk";
    }

    cat.walk();

    // Own Walk
```

### Значение `this`

Пртотипы не влияют на `this`, что позволяет создавать объект с можеством методов, от которого можно наследоваться. Созданные объекты смогут использовать методы родителя изменяя свое состояние, а не состояние родительсокго объекта.

```js
    let animal = {
        sleep() {
            this.isSleeping = true;
        }
    };

    let cat = {
        name: "White Cat"
    };

    cat.__proto__ = animal;
    cat.sleep();

    cat.isSleeping;
    animal.isSleeping;

    // true
    // undefined
```

### Цикл `for..in`

Цикл `for..in` проходит не только по собственным, но и по унаследованным свойствам объекта.

```js
    let animal = {
        etas: true
    };

    let cat = {
        sleep: true
    };

    cat.__proto__ = animal;

    for(let prop in cat) {
        return prop;
    }; 

    // sleep
    // etas
```

Почти все остальные методы, получающие ключи/значения, такие как Object.keys, Object.values и другие – игнорируют унаследованные свойства.

### Функции

Функции в JavaScript тоже объекты, у которых есть 2 свойства: 
- `[[Prototype]]` ссылается на `Function.prototype`
- `prototype` которое содержит: `constructor` ссылающийся на функцию и `[[Prototype]]` ссылающийся на `Object.prototype`.

![](https://github.com/6rost9/turbo-octo-umbrella/blob/main/images/function_prototype.png?raw=true)

### Оператор `new`

Когда мы вызываем функцию с помощью оператора `new` создается новый объект, который передается функции в качестве `this`, сама функция вызывается, и из функции не явно вернется созданный объект. Эти свойства дают возможность использовать функции в качестве конструктора, что позволяет создавать множество  объектов с одним набором свойств.

```js
    function Person(firstName, lastName, born) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.born = born;
    }

    const Ivan = new Person('Ivan', 'Petrov', 1991);
```
Допустим мы хотим расширить объект методом который будет возвращать возраст, мы можем создать метод в объекте `Person`, но таким образом мы получим метод во всех созданных объектах.

```js
    function Person(firstName, lastName, born) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.born = born;

    this.age = function() {
            const now = new Date();

            return now.getFullYear() - this.born;
        }
    }

    const Ivan = new Person('Ivan', 'Petrov', 1991);
    Ivan.age();
    // 31
```

Мы можем вынести метод `age` в прототип `Person` таким образом созданные объекты не будут иметь собственного свойства `age`, но оно будет доступно через прототип

```js
function Person(firstName, lastName, born) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.born = born;
}

Person.prototype.age = function() {
    const now = new Date();

    return now.getFullYear() - this.born;
}

const Ivan = new Person('Ivan', 'Petrov', 1991);
Ivan.age();
// 31
```
![](https://github.com/6rost9/turbo-octo-umbrella/blob/main/images/person_prototype.png?raw=true)

F.prototype используется только в момент вызова new F() и присваивается в качестве свойства `[[Prototype]]` нового объекта. После этого F.prototype и новый объект ничего не связывает.

После создания F.prototype может измениться, и новые объекты, созданные с помощью new F(), будут иметь другой объект в качестве `[[Prototype]]`, но уже существующие объекты сохранят старый.

```js
function Person(firstName, lastName, born) {
    this.firstName = firstName;
    this.lastName = lastName;
    this.born = born;
}

Person.prototype.age = function() {
    const now = new Date();

    return now.getFullYear() - this.born;
}

const Ivan = new Person('Ivan', 'Petrov', 1991);
Ivan.age();
// 31

Person.prototype.age = function () {
    return `born in ${this.born}`;
}

const Vasily = new Person('Vasily', 'Smironv', 1987);
Vasily.age();
// born in 1987
```

### Классы

В ES6 добавлена возможность создания классов, что от части «синтаксический сахар» для прототипного наследования. Так можно переписать пример вынеся функцию констуртор в `constructor` класса, а наследованные через `prototype` методы описать методами класса. Важно помнить, что методы класса не будут являться собственными свойствами объекта, они будут складываться в `[[Prototype]]`.

```js
class Person {
    constructor(firstName, lastName, born) {
        this.firstName = firstName;
        this.lastName = lastName;
        this.born = born;
    }

    age() {
        const now = new Date();
    
        return now.getFullYear() - this.born;
    }    
}

const Vasily = new Person('Vasily', 'Smironv', 1987);
Vasily.age();
// 35
```
![](https://github.com/6rost9/turbo-octo-umbrella/blob/main/images/person_class.png?raw=true)