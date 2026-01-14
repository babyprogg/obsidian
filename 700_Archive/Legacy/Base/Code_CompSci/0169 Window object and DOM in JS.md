# Base
С помощью Js мы можем манипулировать DOM HTML. Менять атрибуты,текст и тд.
**Чтобы работать с ДОМ-ом нужно привязаться к хтмл и отслыаться к айди или классам**
*Single element selector*
	1. console.log(document.getElementById('my-form'));
	Можно сделать как переменную 
	2.
	 const form = document.getElementById('my-form');
	console.log(form);
	*3.* console.log(document.querySelector('my-form'));
*Multiple element selector*
	1. console.log(document.querySelectorAll('.item'));
