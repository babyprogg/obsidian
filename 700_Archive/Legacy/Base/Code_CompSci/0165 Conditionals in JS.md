# Basics
const x = 10;
if(x == 10){
	console.log('x is 10');
} else if(x > 10){
	console.log('x is greater than 10')
} else {
console.log('x is less than 10');
}

|| - or/или. Требует только одно из условий верным/true
const x = 5;
const y = 11;
if(x > 5 || y > 10) {
	console.log(x is greater than 5 or y greater than 10)
}

&& - и/and. Требует все условия верным/true.
const x = 6;
const y = 11;
if(x > 5 && y > 10) {
	console.log(x is greater than 5 or y greater than 10)
}

? - then/после
: - else/так же
const = 10;

const color = x > 10 ? 'red' : 'blue';
console.log(color);


const = 10;

const color = x > 10 ? 'red' : 'blue';
console.log(color);
switch(color){
case 'red':
	console.log('color is read');
	break;
case 'blue':
	console.log('color is blue');
	break;
default:
	console.log('color is not red or blue');
	break;
}

