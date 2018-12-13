# Dicas TypeScript / Javascript (ES6) ...

### Remover elementos de um Indexable Type

const themes: {[ theme: string ]: string} = {
	'default-theme': 'Padrão',
	'dark-theme': 'Escuro'
};

Remover um elemento:
delete themes['default-theme'];
console.log(themes);

Remover um elemento sem mudar o original:
const allThemesExceptDefault = { ...themes };
delete allThemesExceptDefault['default-theme'];
console.log(themes);
console.log(allThemesExceptDefault);

### Juntar dois ou mais arrays usando Array Spread (ES6)
const arr1 = [1,2,3]
const arr2 = [4,5,6]
const arr3 = [...arr1, ...arr2] //arr3 ==> [1,2,3,4,5,6]

