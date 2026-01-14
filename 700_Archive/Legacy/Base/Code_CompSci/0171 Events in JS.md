# Wha
Это вещи которые происходят с хтмл элементами. Когда джаваскрипт используется в хтмл,то джаваскрипт реагирует на эти эвентыю
## Base
const btn = document.querySelector('.btn');
btn.addEventListener('click', (e) => {
e.preventDefault();
console.log(e);
});
*Теперь с айди плюс игра со стилем*
onst btn = document.querySelector('.btn');
btn.addEventListener('click', (e) => {
document.querySelector('#my-form').style.background = '#ccc';
});