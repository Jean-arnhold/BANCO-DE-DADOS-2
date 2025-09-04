# LISTA 1

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

# LISTA 2

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

# LISTA 3

## 1-Criar uma função que recebe o CPF do funcionário e retorna o valor do aumento do funcionário. Se a soma de horas trabalhadas em projetos pelo funcionário for maior ou igual a 40, o aumento deve ser de 20%. Caso contrário o aumento deve ser de 5%.

```sql
DELIMITER //

CREATE FUNCTION calcular_aumento(cpf_funcionario VARCHAR(11)) 
RETURNS DECIMAL(5,2)
BEGIN
    DECLARE total_horas INT;
    DECLARE percentual_aumento DECIMAL(5,2);
    
    -- Calcula a soma total de horas trabalhadas em projetos
    SELECT COALESCE(SUM(te.Horas), 0) INTO total_horas
    FROM trabalha_em te
    INNER JOIN funcionario f ON te.Fcpf = f.Cpf
    WHERE f.cpf = cpf_funcionario;
    
    IF total_horas >= 40 THEN
        SET percentual_aumento = 20.00;
    ELSE
        SET percentual_aumento = 5.00;
    END IF;
    
    RETURN percentual_aumento;
END //

DELIMITER ;
```

## 2-Fazer uma consulta para mostrar o uso da função que retorne o nome do funcionário e seu novo salário.

```sql
SELECT 
    f.Pnome,
    f.cpf,
    f.salario AS salario_atual,
    calcular_aumento(f.cpf) AS percentual_aumento,
    (f.salario * calcular_aumento(f.cpf) / 100) AS valor_aumento,
    (f.salario + (f.salario * calcular_aumento(f.cpf) / 100)) AS novo_salario
FROM funcionario f
ORDER BY percentual_aumento DESC, f.Pnome;
```

## 3-Criar uma função que recebe três 3 números de projeto como parâmetro e retorna o projeto com mais funcionários trabalhando nele.

```sql
DELIMITER //

CREATE OR REPLACE FUNCTION projeto_mais_funcionarios(projeto1 INT, projeto2 INT, projeto3 INT) 
RETURNS INT
READS SQL DATA
BEGIN
    DECLARE projeto_maior INT;
    DECLARE count1 INT;
    DECLARE count2 INT;
    DECLARE count3 INT;
    

    SELECT COUNT(DISTINCT trabalha_em.Fcpf) INTO count1
    FROM trabalha_em
    WHERE Pnr = projeto1;
    
    
    SELECT COUNT(DISTINCT trabalha_em.Fcpf) INTO count2
    FROM trabalha_em
    WHERE Pnr = projeto2;
    

    SELECT COUNT(DISTINCT trabalha_em.Fcpf) INTO count3
    FROM trabalha_em
    WHERE Pnr = projeto3;
    

    IF count1 >= count2 AND count1 >= count3 THEN
        SET projeto_maior = projeto1;
    ELSEIF count2 >= count1 AND count2 >= count3 THEN
        SET projeto_maior = projeto2;
    ELSE
        SET projeto_maior = projeto3;
    END IF;
    
    RETURN projeto_maior;
END //

DELIMITER ;
```

## 4-Fazer uma consulta para mostrar o uso da função.

```sql
SELECT projeto_mais_funcionarios(1, 2, 3) AS projeto_maior;
```