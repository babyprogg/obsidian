# Код
const todos = [
	{ 
		id:1,
		text:'Study an anatomy of dragonfly',
		isCompleted: true
	},
	{ 
		id:2,
		text:'Meeting with friends',
		isCompleted: true
	},
	{ 
		id:3,
		text:'Buy a purple Buggati',
		isCompleted: false
	}
];

## Конвертация в Json
const todoJSON = JSON.stringify(todos);
console.log(todoJSON);
