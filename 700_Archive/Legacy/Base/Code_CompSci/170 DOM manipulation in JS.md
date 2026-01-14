# Зачем
"Зачем нам это когда есть цсс?" Ответ: Это динамичнее. Можно изменить составляющее хтмл одной строчкой кода.
## Basic
const ul = document.querySelector('.items');
ul.remove();
**Это только один из многих**
ul.lastElementChild.remove();
*Это меняет изначальный текст на другой*
ul.firstElementChild.textContent = 'Hello';
ul.children[1].innerText = 'Brad';
ul.lastElementChild.innerHTMl = '<h1.>Hello</h1>;
*Игры со стайлом*
const btn = document.querySelector('.btn');
btn.style.background = 'red';