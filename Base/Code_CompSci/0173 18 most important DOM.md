1. Adding elements 
	Есть два append
	append(string) && appendChild(elements like divs,spans etc)
2. Creating elements
	Сперва говорим что мы собираемся добавить createElement('div')
	Потом мы создаем конст 
	const div = document.createElement('div')
	Потом наполняем пустой див
	div.innerText = "Hello world"
	теперь мы можем добавить его через аппенд
3. Modifying element text
	div.textContent = "Hello world"
	и
	div.innerText = "Hello World"
	Они делают одно и тоже но в чем же разница 
	innerText = принтует как в  HTML
	textContent = принтует вообще все как в HTML с пробелами,extra space и тд
4. Modifying HTML element 
	div.innerHTML = "<strong>Hello World</strong>"
	Можно сделать чуть больше но не иметь проблем в будущем. Это так создать консты с таким хтмл
	const strong = document.createElement('strong')
	strong.innerText = 'Hello World 2'
	div.append(strong)
5. Removing Elements from DOM
	div.removeChild(spanHi)
	spanHi это конст с хтмл айди
	Есть еще другой способ как div.remove(spanHi)
	Но самый легкий способ это 
	spanHi.remove()
6. Modifying Element Attributes
	Если нам нужно получить значение из хтмл элементов то делаем так
	console.log(spanHi.getAttribute("id(значение которые мы хотим получить)")) 
	Но есть более легкий способ это
	console.log(spanHi.title)
	Можно еще изменять название значений 
	console.log(spanHi.setAttribute("title", "sdfsdfsd")
	Аналогично,можно сделать легче
	spanHi.id = 'sdfsdfs')
	Еще можно удалить атрибуты
	spanHi.removeAttribute("id")
7. Modifying Data Attributes 
	console.log(spanHi.dataset)
	Можно так получать доступ к дате в хтмл 
	Еще можно добавлять дату через JS 
	spanHi.dataset.newName = "hi"
8. Modifying  Element Classes 
	Можно добавить класс 
	spanHi.classList.add("new-class")
	Можно и удалить класс
	spanHi.classList.remove("hi1")
	Можно еще удалить класс или добавить его если не существует
	spanHi.classList.toggle("hi3")
	Можно еще управлять им 
	spanHi.classList.toggle("hi3", false) удалить 
	spanHi.classList.toggle("hi3", true) добавить
9. Modifying Element Style
	Можно использовать любой стайл цсс 
	spanHi.style.color = "red"
	А с тэгами с дефизом(style-background) по другому
	spanHi.style.backgroundColor = "red"


tag: #айти 
links: 
src: [Learn DOM Manipulation In 18 Minutes - YouTube](https://www.youtube.com/watch?v=y17RuWkWdn8&t=61s)