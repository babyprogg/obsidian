# Explaination
Смотри, если нам скажут вычислить сабаррей в аррее то по проводить итерации на каждый индекс это ужасно, медленно и неэффективно потому что мы проводим через итерацию один элемент дважды.
Поэтому используем слайдинг уиндоу техник.
Суть заключается в том чтобы якобы слайдить по каждому индексу отнимая пердыдущий индекс и добавляя последний. Например: 
k = 3
[1,2,3,4,5,6]
сперва итерация пойдет по первым трем индексам(1,2,3)
но мы отнимаем один и добавляем 4. 
iteration1 = 1+2+3=>6 
iteration2 = 6-1+4=>9
## Kind of Problems
- The problem will be based on an array, string, or list data structure.
- You need to find the subrange in this array or string that should provide the longest, shortest, or target values.
- A classic problem: to find the largest/smallest sum of given k (for example, three) consecutive numbers in an array.


src: [JS | 98% | Sliding window | With exlanation - Longest Substring Without Repeating Characters - LeetCode](https://leetcode.com/problems/longest-substring-without-repeating-characters/solutions/2694302/js-98-sliding-window-with-exlanation/)
links: 