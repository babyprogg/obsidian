# Basic code
- For 
	for(let i = 0; i <= 10; i++){
	console.log(`For Loop Number ${i}`);
	}
	for(начало;условие;шаг){тело.действие которое будет лупится}
- While
	let i = 0 
	while(i <=10){
	console.log(`While Loop Number ${i}`);
	i++
	}
## Looping through arrays
for(let todo(*можно как угодно назвать*) of todos){
console.log(todo.text);
}
### Higher order array methods
- forEach(сквозь массив)
- 
		todos.forEach(function(todo){
		console.log(todo.text);
		});
- map(массив из массива)
- 
		const todoText = todos.map(function(todo){
		return todo.text;
		});
		console.log(todoText);
- filter(массив из условия)
- 
		const todoCompleted = todos.filter(function(todo){
		return todo.isCompleted === true;
		}).map(function(todo){
		return todo.text;
		})
		console.log(todoCompleted);