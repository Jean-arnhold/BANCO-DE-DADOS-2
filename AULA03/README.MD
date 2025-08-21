# aula 03
## Criação de Colunas Calculadas

```sql
CREATE TABLE exemplo1 (
  ID int PRIMARY KEY AUTO_INCREMENT,
  valor1 int NOT NULL,
  valor2 int NOT NULL,
  valor3 int GENERATED ALWAYS AS (valor1 * valor2) VIRTUAL
);
```

## Atividades

### 1.  Crie uma tabela item_venda com as seguintes colunas: id(AUTO_INCREMENT e PRIMARY KEY), produto_id, nota_id, quantidade, preco_unitario e valor_total onde: valor_total: deve ser uma coluna que calcula automaticamente o valor total do item considerando a quantidade vendida e o preço do item. O valor deve ser armazenado. 

```sql
CREATE TABLE item_venda (
    id int PRIMARY KEY AUTO_INCREMENT,
    produto_id int NOT NULL,
    nota_id int NOT NULL,
    quantidade int NOT NULL,
    preco_unitario DOUBLE NOT NULL,
    valor_total int GENERATED ALWAYS AS (quantidade * preco_unitario) STORED
    );
``` 
### 2.Crie uma tabela funcionarios com as seguintes colunas: id(AUTO_INCREMENT e PRIMARY KEY), nome, data_nascimento, data_admissao, idade onde: idade: deve ser uma coluna que calcula automaticamente a idade do funcionário. O valor deve não deve ser armazenado.

```sql
CREATE TABLE funcionarios (
    id int PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(30) NOT NULL,
    data_nascimento DATE NOT NULL,
    data_admissao DATE NOT NULL,
    idade int GENERATED ALWAYS AS (DATEDIFF(data_nascimento, CURRENT_DATE())) VIRTUAL
    );
``` 

### 3.  Crie uma tabela produtos com as seguintes colunas: id(AUTO_INCREMENT e PRIMARY KEY), nome, custo, margem_lucro e preco_venda onde: preco_venda: deve ser uma coluna que calcula automaticamente o preço de venda de um produto, adicionando uma margem de lucro ao custo. O valor deve ser armazenado. 


```sql
CREATE TABLE produtos (
    id int PRIMARY KEY AUTO_INCREMENT,
    nome VARCHAR(30) NOT NULL,
    custo double NOT NULL,
    margem_lucro double NOT NULL,
    preco_venda double GENERATED ALWAYS AS (custo * (1 + margem_lucro)) STORED
    );
```

## Atividade Views

```sql
CREATE VIEW viewConsolidade AS SELECT

	pessoa.pais as Pais,
    "PF" AS Tipo,
    COUNT(pessoa.pessoa_id) as Quantidade
FROM
	pessoa
INNER JOIN pessoa_fisica ON pessoa_fisica.pessoa_id = pessoa.pessoa_id
GROUP BY pessoa.pais;

CREATE USER 'appGerente'@'localhost' 
IDENTIFIED BY 'senha_segura';

GRANT SELECT on viewconsolidada TO 'appGerente'@'localhost';
``` 