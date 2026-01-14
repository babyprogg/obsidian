# Базовое понятие
*Это просто парные данные. Типо 
человек и его возраст,имя и тд*
## Код
const person = {
	firstName: 'Rocky',
	lastName: 'Balboa',
	age: 30,
	hobbies: ['box','running','winning'],
	address: {
	street: '50 main st',
	city:'Boston',
	state:'MA'
	}
}
1. Можно получать нужную нам инфу. Например,только имя и фамилию то:
	   console.log(person.firtsName, person.lastName);
2.  Если нам нужно какое либо данное внутри объекта то:
	console.log(person.hobbies[1]);
	console.log(person.adress.city);
3. Деструктуризация 
	const{ firstName, lastName,address: {city}} = person;
4. Если нужно что то добавить то:
	person.email = 'rocky@gmail.com';
	