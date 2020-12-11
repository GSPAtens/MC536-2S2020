# lab 08

### Gabriel de Sousa Pereira
RA: 216194

## PubChem e DRON com XQuery/XPath

## Questão 1
Construa uma comando SELECT que retorne dados equivalentes a este XPath
~~~xquery
//individuo[idade>20]/@nome
~~~

### Resolução
~~~xquery
SELECT nome
FROM individuo
WHERE idade > 20
~~~

## Questão 2
Qual a outra maneira de escrever esta query sem o where?

~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
 
for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~
### Resolução
~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')
 
for $i in ($fichariodoc//individuo[idade>17])
return {data($i/@nome)}
~~~

## Questão 3
Escreva uma consulta SQL equivalente ao XQuery:
~~~xquery
let $fichariodoc := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/xml/fichario.xml')

for $i in ($fichariodoc//individuo)
where $i[idade>17]
return {data($i/@nome)}
~~~

### Resolução
~~~sql
SELECT nome
FROM individuo
WHERE idade > 17
~~~

## Questão 4
Retorne quantas publicações são posteriores ao ano de 2011.

### Resolução
~~~xquery
let $publicacoes := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')

return count($publicacoes//[years>2011])
~~~

## Questão 5
Retorne a categoria cujo `<label>` em inglês seja 'e-Science Domain'.

### Resolução
~~~xquery
let $publicacoes := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')

return data($pubs//categories/category[label/@lang="en-US" and label="e-Science Domain"]/@key)
~~~

## Questão 6
Retorne as publicações associadas à categoria cujo `<label>` em inglês seja 'e-Science Domain'. A associação entre o label e a key da categoria deve ser feita na consulta.

### Resolução
~~~xquery
let $publicacoes := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/publications/publications.xml')

let $categoria := data($publicacoes//categories/category[label/@lang="en-US" and label="e-Science Domain"]/@key)
return $publicacoes//publication[key=$categoria]
~~~

## Tarefas com DRON e PubChem

## Questão 1

Liste o nome de todas as classificações que estão apenas dois níveis imediatamente abaixo da raiz.

### Resolução
~~~xquery
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')
let $deep2 := $dron/drug/drug/drug
for $d in ($deep2)
return {data($d/@name), '&#xa;'}
~~~

## Questão 2

Apresente todas as classificações de um componente a sua escolha (diferente de `Acetylsalicylic Acid`) que estejam hierarquicamente dois níveis acima. Note que no exemplo dado em sala foi considerado um nível hierárquico acima.

### Resolução
Exemplo para METHENAMINE
~~~xquery
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')//drug

for $d in ($dron)
where ($d/drug/drug/@name) = "METHENAMINE"
let $drug := data($d/@name)
group by $drug
order by $drug
return {$drug, '&#xa;'}
~~~

## Questão 3

### Questão 3.1

Liste todos os códigos ChEBI dos componentes disponíveis.

#### Resolução
~~~xquery
let $data := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')

for $drug in ($data/PC-DataSet/InformationList/Information)
let $ChEBI := substring(data($drug/Synonym[1]),7)
group by $ChEBI
return {$ChEBI, '&#xa;'}
~~~

### Questão 3.2

Liste todos os códigos ChEBI e primeiro nome (sinônimo) de cada um dos componentes disponíveis.

#### Resolução
~~~xquery
let $data := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')

for $drug in ($data/PC-DataSet/InformationList/Information)
let $ChEBI := substring(data($drug/Synonym[1]),7)
let $name := data($drug/Synonym[2])
return {"ChEBI: ", $ChEBI, '&#xa;', "Nome: ", $name, '&#xa;', '&#xa;'}
~~~

### Questão 3.3

Para cada código ChEBI, liste os sinônimos e o nome que aparece para o mesmo componente no DRON (se existir). Para isso faça um JOIN com o DRON.

#### Resolução
~~~xquery
let $pubchem := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/pubchem/pubchem-chebi-synonyms.xml')/PC-DataSet
let $dron := doc('https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017-dron/dron.xml')

for $pub in ($pubchem/InformationList/Information),
    $dr in ($dron//drug)
where $dr/@id = concat("http://purl.obolibrary.org/obo/CHEBI_", substring($pub/Synonym[1]/text(), 7))
return {data($dr/@id),'&#xa;'}
~~~
