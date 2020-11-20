# lab 06

### Gabriel de Sousa Pereira
RA: 216194

## Tarefa de Cypher e o FDA Adverse Event Reporting System (FAERS)

## Exercício 1

Escreva uma sentença em Cypher que crie o medicamento de nome `Metamizole`, código no DrugBank `DB04817`.

### Resolução
~~~cypher
CREATE (:Drug {drugbank: "DB04817", name: "Metamizole"})
~~~

## Exercício 2

Considerando que a `Dipyrone` e `Metamizole` são o mesmo medicamento com nomes diferentes, crie uma aresta com o rótulo `:SameAs` que ligue os dois.

### Resolução
~~~cypher
MATCH (d1:Drug {name:"Dipyrone"})
MATCH (d2:Drug {name:"Metamizole"})
CREATE (d1)-[:SameAs]->(d2)
~~~

## Exercício 3

Use o `DELETE` para excluir o relacionamento que você criou (apenas ele).

### Resolução
~~~cypher
MATCH (d1:Drug {name:"Dipyrone"})
MATCH (d2:Drug {name:"Metamizole"})
MATCH (d1)-[r:SameAs]->(d2)
DELETE r
~~~

## Exercício 4

Faça a projeção em relação a Patologia, ou seja, conecte patologias que são tratadas pela mesma droga.

### Resolução
~~~cypher
MATCH (p1:Pathology)<-[a]-(d:Drug)-[b]->(p2:Pathology)
MERGE (p1)<-[r:Relates]->(p2)
ON CREATE SET r.weight=1
ON MATCH SET r.weight=r.weight+1
~~~

## Exercício 5

Construa um grafo ligando os medicamentos aos efeitos colaterais (com pesos associados) a partir dos registros das pessoas, ou seja, se uma pessoa usa um medicamento e ela teve um efeito colateral, o medicamento deve ser ligado ao efeito colateral.

### Resolução
~~~cypher
//Primeiro gero a relação entre pessoas e medicamentos, evitando repetições
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/drug-use.csv' AS line
MERGE (p:Person {code: line.idperson})
MERGE (d:Drug {code: line.codedrug})
MERGE (p)-[:Takes]->(d)

//Depois relaciono as pessoas com a ocorrêcia de efeitos colaterais
LOAD CSV WITH HEADERS FROM 'https://raw.githubusercontent.com/santanche/lab2learn/master/data/faers-2017/sideeffect.csv' AS line
MERGE (p:Person {code: line.idPerson})
MERGE (s:SideEffect {code: line.codePathology})
MERGE (s)-[:Affects]->(p)

//Desta forma, traço a relação direta entre medicamento e efeito colateral
MATCH (s)-[:Affects]->(p)-[:Takes]->(d)
MERGE (d)-[c:Causes]->(s)
ON CREATE SET c.weight=1
ON MATCH SET c.weight=c.weight+1
~~~

## Exercício 6

Que tipo de análise interessante pode ser feita com esse grafo?

Proponha um tipo de análise e escreva uma sentença em Cypher que realize a análise.

### Resolução
~~~cypher
//Atráves do grafo, é possível identificar os medicamentos cujo usuários mais apresentaram efeitos colaterais:
MATCH (d)-[c:Causes]->(s)
WHERE c.weight>30
RETURN d,s

//Também é possível realizar as projeções, em relação aos medicamentos:
MATCH (s1:SideEffect)<-[a]-(d:Drug)-[b]->(s2:SideEffect)
MERGE (s1)<-[r:Relates]->(s2)
ON CREATE SET r.weight=1
ON MATCH SET r.weight=r.weight+1

//e em relação aos efeitos colaterais:
MATCH (d1:Drug)->[a]-(s:SideEffect)-[b]<-(d2:Drug)
MERGE (d1)->[r:Relates]<-(d2)
ON CREATE SET r.weight=1
ON MATCH SET r.weight=r.weight+1
~~~
