## LISTA 1

## 1-Criar uma função que recebe a data de nascimento do funcionário e retorna a idade do funcionário. Dica: procurar funções de data(date) do MySQL pode ajudar.

```sql
CREATE FUNCTION mostrar_idade(datanasc date)
RETURNS INT
RETURN YEAR(FROM_DAYS(DATEDIFF(CURRENT_DATE, Datanasc)))
```

## 2-Fazer um select que retorne o nome e a idade de todos os funcionário demostrando o uso da função.

```sql
SELECT funcionario.Pnome, 
mostrar_idade(funcionario.Datanasc) as idade
FROM funcionario;
```

## 3-Criar uma função que recebe o CPF do funcionário e retorna o número de dependentes do funcionário.


```sql
CREATE OR REPLACE FUNCTION dependentes_funcionario (cpf BIGINT)
RETURNS INT
RETURN(
   
   SELECT COUNT(dependente.Fcpf) as dependentes
   FROM
    dependente
   WHERE dependente.Fcpf = cpf
);
```

## 4-Fazer um select que retorne o nome e o número de dependentes de um funcionário informado na função.

```sql
SELECT funcionario.Pnome, 
dependentes_funcionario(funcionario.Cpf) as dependentes 
FROM funcionario
WHERE funcionario.Cpf = 12345678966;
```

## 5-Fazer um select que retorne o nome e o número de dependentes de todos os funcionários.

```sql 
SELECT funcionario.Pnome, 
dependentes_funcionario(funcionario.Cpf) as dependentes 
FROM funcionario;
```

## LISTA 2

## 1. Crie a função TotalHoras para retornar o total de horas que um funcionário trabalha. O parâmetro de entrada da função deve ser o cpf do funcionário. A função deve retornar o total de horas que o funcionário trabalha (tabela trabalha_em).


```sql
CREATE OR REPLACE FUNCTION TotalHoras (cpf_funcionario BIGINT)
RETURNS INT
RETURN(
SELECT funcionario.Pnome, SUM(trabalha_em.Horas)
FROM trabalha_em
WHERE funcionario.Cpf = 12345678966);
```

## 2. Crie a função TotalHorasProjetos para retornar o total de horas e o número de projetos que um funcionário trabalha. O parâmetro de entrada da função deve ser o cpf do funcionário. A função deve retornar de forma concatenada o total de horas que o funcionário trabalha e a quantidade de projetos que o funcionário trabalha. Exemplo de retorno da função: '10h - 2 projetos'.

```sql
CREATE OR REPLACE FUNCTION TotalHorasProjetos(cpf_funcionario BIGINT)
RETURNS VARCHAR(55)
RETURN(
SELECT
CONCAT_WS( " - ", 
          CONCAT(SUM(trabalha_em.Horas), "h"),
          CONCAT_WS(" ", COUNT(trabalha_em.Pnr), "projetos") 
)
FROM trabalha_em
WHERE trabalha_em.Fcpf = 12345678966);
```

## 3. Crie a função TotalFuncionários para retornar a quantidade de funcionários que trabalham em um determinado projeto. O parâmetro de entrada da função deve ser o nome do projeto. A função deve retornar a quantidade de funcionários que trabalha no projeto indicado.


```sql
CREATE OR REPLACE FUNCTION TotalFuncionarios (nome_projeto VARCHAR(20))
RETURNS INT 
RETURN(
SELECT COUNT(trabalha_em.Pnr)                      
FROM 
	trabalha_em
JOIN
	projeto ON projeto.Projnumero = trabalha_em.Pnr
WHERE
	projeto.Projnome = nome_projeto);
```

## 4. Crie a função MediaSalDep para retornar a média salarial dos funcionários de um departamento. O parâmetro de entrada da função deve ser o nome do departamento. A função deve retornar a média salarial dos funcionários que trabalham no departamento indicado.

```sql
CREATE OR REPLACE FUNCTION TotalFuncionarios (nome_departamento VARCHAR(25))
RETURNS DECIMAL
RETURN(
SELECT AVG(funcionario.Salario)                    
FROM 
	funcionario
JOIN
	departamento ON departamento.Dnumero = funcionario.Dnr
WHERE
	departamento.Dnome= nome_departamento);
```

