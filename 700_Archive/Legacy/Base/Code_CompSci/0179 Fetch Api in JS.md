# База
Fetch Api - это интерфейс для получения данных с браузера. Нужен для отправки и получения данных с серверов. Обновление профиля пользователя или список продуктов без перезагрузки страницы.Типо 200 - ОК 
## Sending a request 
Promise - нужен для создания запросов с веб браузера. 
fetch() - метод который указывает веб браузерам отправлять запрос на URL адрес.
fetch требует лишь один параметр который url ресурса который ты хочешь заfetch-ить.
let response = fetch(url)
fetch возвращает промис поэтому можно использовать методы then() and catch():
fetch(url)
.then(res(response) => {
// разбирается с ответом
console.log('Succes');
})
.catch(error => {
// разбирается с проблемами
console.log('Failed');
});
fetch всегда успешен,даже когда провалился. Он всегда кидает данные с сайта,даже если его нет. Дабы это избежать нужно встроить ошибку в сам fetch:
fetch(url)
.then(res => { 
	if(res.ok) {
	console.log('Succes')
	}else {
		console.log('Not succeful')
		}
})
Иногда мы хотим удалить,добавить и тд данные. Для этого:
Сейчас мы создаем пользователя
fetch(url)
	method: 'POST',
	headers: {
	'Content-Type': 'application/json'
	}
	body: JSON.stringify({
	name: 'User 1' 
	})
.then(res => { 
	if(res.ok) {
	console.log('Succes')
	}else {
		console.log('Not succeful')
		}
})

tag: #dev 
links:
src: [Learn Fetch API In 6 Minutes - YouTube](https://www.youtube.com/watch?v=cuEtnrL9-H0), [Урок 14. JavaScript. Запросы на сервер. Fetch, XMLHttpRequest (XHR), Ajax - YouTube](https://www.youtube.com/watch?v=eKCD9djJQKc)