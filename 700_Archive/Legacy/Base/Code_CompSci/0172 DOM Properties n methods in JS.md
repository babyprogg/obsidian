Узлом может быть что угодно,но чаще всего это html class or id

# Accessor properties
- textContent - текст заданный node(узлу)
- nodeType - integer показывающий тип узла
- nodeName - string показывающий имя узла
- nodeValue - A string representing the value of the node(хз)
- parentNode - отсылка к родителю узла к которому мы обращаемся
- firstChild - NodeList хранящий в себе всех "детей" данного узла
- lastChild - отсылка к последнему ребенка
- nextSibling - следущий "родственник"
- previousSibling - предедущий родственник
- ownerDocument - отсылка к документу хранящий узел
## Methods
- appendChild() - 
	 добавляет узел к данному узлу как последнего ребенка
	 syntax: ==node.appendChild(childNode)==
- insertBefore() - 
	добавляет узел к данному узлу как предедущего ребенка
	syntax
	==node.insertBefore(newNode,childNode)==
	node - узел в который мы хотим добавить
	newNode - новый узел который мы добавляем
	childNode - это референс узла который мы хотим добавить
- replaceChild() - 
	заменить ребенка данного узла другим узлом
	syntax:
	node.replaceChild(newNode,oldNode)
- removeChilds() - 
	убрать ребенка из данного узла
	syntax:
	node.removeChild(childNode)
- cloneChild() - копия данного узла

