# Basics
function - блок кода для выполнения конкретной задачи.
*Пример кода*
	function addNums(num1 , num2) {
	console.log(num1 + num2);
	}
	addNums(5,4);
*Можно задать дефолт значения так:*
	function addNums(num1 = 1 , num2 = 1)
	но они будут сбиваться когда ты добавишь значения в addNums. Тоесть в нашу функцию.
*Во многом мы хотим что то за return-ить*
	function addNums(num1 , num2) {
	return(num1 + num2);
	}
	console.log(addNums(5,4));
*Перенос эту функцию в функцию стрелки(arrow)*
	const addNums = (num1 = 1, num2 = 1) =>console.log(num1 + num2);
	Чем примечателен этот метод? Тем,что можно это написать в одну линию и избавиться от курли брейсес. 
Еще можно делать так:
const addNums = num1 =>num1 + 5;
console.log(addNums(5));
**Таким методом можно делать не только с функциями,но и массивами(мейби со всем но хз,гугли)**
Примерно так:
todos.forEach((todo) => console.log(todo));

