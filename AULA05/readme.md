## LISTA 1


## Exercício 1: Soma dos Números: Crie uma função que aceita um número inteiro positivo N como parâmetro e retorna a soma dos números inteiros de 1 a N usando um laço WHILE

## Exercício 2: Tabuada: Crie uma função que aceita um número inteiro N como parâmetro e gera a tabuada desse número de 1 a 10 usando um laço REPEAT (o retorno da função deve ser um VARCHAR de tamanho suficiente para mostrar os resultados das multiplicações.)


## Exercício 3: Contagem Regressiva: Crie uma função que aceita um número inteiro positivo N como parâmetro e exibe uma contagem regressiva de N até 1 usando um laço LOOP.


```sql

1.

DELIMITER $

CREATE FUNCTION SomaNumeros(N INT) 
RETURNS INT
BEGIN
    DECLARE soma INT DEFAULT 0;
    DECLARE contador INT DEFAULT 1;

    WHILE contador <= N DO
        SET soma = soma + contador;
        SET contador = contador + 1;
    END WHILE;

    RETURN soma;
END $

DELIMITER ;
----------------------------------------
2.

DELIMITER $

CREATE FUNCTION tabuada(N INT) 
RETURNS VARCHAR(255)
BEGIN
    DECLARE resultado VARCHAR(255) DEFAULT '';
    DECLARE contador INT DEFAULT 1;

    REPEAT
        SET resultado = CONCAT(resultado, N, ' x ', contador, ' = ', N * contador, '\n');
        SET contador = contador + 1;
    UNTIL contador > 10
    END REPEAT;

    RETURN resultado;
END $

DELIMITER ;

----------------------------------------
3.

DELIMITER $

CREATE FUNCTION contagemRegressiva(N INT) 
RETURNS VARCHAR(255)
BEGIN
    DECLARE resultado VARCHAR(255) DEFAULT '';
    DECLARE contador INT;

    SET contador = N;

    loop_contagem: LOOP
        IF contador < 1 THEN
            LEAVE loop_contagem;
        END IF;
        SET resultado = CONCAT(resultado, contador, ' ');
        SET contador = contador - 1;
    END LOOP loop_contagem;

    RETURN TRIM(resultado);
END $

DELIMITER ;

```


## Exercício 4: Geração de Relatório: Crie uma função que gera um relatório com base em uma tabela de vendas. A tabela de vendas tem colunas id, data, valor. A função deve iterar pelas vendas em um período específico e calcular o total de vendas para cada dia. O relatório deve listar a data e o total de vendas para cada dia durante o período.
Tabela e dados exemplo abaixo:

```sql

CREATE TABLE vendas (
 id INT AUTO_INCREMENT PRIMARY KEY,
 data DATE NOT NULL,
 valor DECIMAL(10, 2) NOT NULL
);
INSERT INTO vendas (data, valor) VALUES
 ('2023-09-01', 100.50),
 ('2023-09-01', 75.25),
 ('2023-09-02', 200.00),
 ('2023-09-02', 50.75),
 ('2023-09-03', 150.20),
 ('2023-09-03', 85.30),
 ('2023-09-03', 120.75);

 ```
## Resposta
 ```sql

DELIMITER $

CREATE FUNCTION relatorio_vendas(data_inicial DATE, data_final DATE)
RETURNS TEXT
BEGIN
    DECLARE resultado TEXT DEFAULT '';
    DECLARE data_atual DATE;
    DECLARE total DECIMAL(10, 2);

    SET data_atual = data_inicial;

    WHILE data_atual <= data_final DO
        SELECT SUM(valor) INTO total FROM vendas WHERE data = data_atual;
        SET resultado = CONCAT(resultado, 'Data: ', data_atual, ', Total de Vendas: ', total, '\n');
        SET data_atual = DATE_ADD(data_atual, INTERVAL 1 DAY);
    END WHILE;

    RETURN resultado;
END;
$

DELIMITER ;

Uso da função:
SELECT relatorio_vendas('2023-09-01', '2023-09-03'); -- Substitua as datas pelo intervalo desejado

 ```