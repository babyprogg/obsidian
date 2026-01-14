# Basics
// Constructor function 
function  Person(firstName , lastName , dob){
this.firstName = firstName;
this.lastName = lastName;
this.dob = dob;
}

// Instantiate object
const person1 = new Person('John', 'Doe', '4-3-1980');
const person2 = new Person('Rocky', 'Balboa','1-5-1979');
console.log(person1);

## This
*this* - это ключевое слово относящиеся к объекту 
The `this` keyword refers to different objects depending on how it is used:
	In an object method, `this` refers to the **object**
	Alone, `this` refers to the **global object**.
	In a function, `this` refers to the **global object**
	In a function, in strict mode, `this` is `undefined`.
	In an event, `this` refers to the **element** that received the event.
	Methods like `call()`, `apply()`, and `bind()` can refer `this` to **any object**.
*this* - отсылается к обладателю функции.
В примере сверху this отсылает к person который обладает firstName.
### Prototypes
Это функции которые не будут показываться(ресерч)
Person.prototype.getBirthYear = function(){
return this.dob.getFullYear();
}