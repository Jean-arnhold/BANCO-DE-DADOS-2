
## Exercício 1 - Procedure simples
Crie uma procedure chamada aumenta_salario que receba dois parâmetros: id_func (INT) e
percentual (DECIMAL). A procedure deve atualizar o salário do funcionário informado e retornar
o novo salário.
Tabelas necessárias
CREATE TABLE funcionarios (
 id INT PRIMARY KEY AUTO_INCREMENT,
 nome VARCHAR(100),
 salario DECIMAL(10,2)
);
INSERT INTO funcionarios (nome, salario) VALUES
('Ana', 2500.00),
('Bruno', 3000.00),
('Carla', 4000.00);



```sql
DELIMITER $
CREATE PROCEDURE aumenta_salario(
    IN id_func INT,
    IN percentual DECIMAL(5,2)
)
BEGIN
    UPDATE
        funcionarios
    SET
        funcionarios.salario =
            funcionarios.salario * (1 + (percentual / 100))
        
    WHERE
        funcionarios.id = id_func;

END$
DELIMITER
;

CALL aumenta_salario(2, 10);
```

## Exercício 1B- Procedure simples com variável de saída Crie uma procedure chamada aumenta_salario_v2 que receba dois parâmetros: id_func (INT) e percentual (DECIMAL). A procedure deve atualizar o salário do funcionário informado e retornar uma variável de saída com o texto “Salário atualizado para ” concatenado com o novo salário.

```sql 
DELIMITER $

CREATE PROCEDURE aumenta_salario_v2(
    IN id_func INT,
    IN percentual DECIMAL(5,2),
    OUT saida DECIMAL(10, 2)
)
BEGIN
    UPDATE funcionarios 
    SET salario = salario * (1 + percentual/100)
    WHERE id = id_func;
    
    SELECT salario
    INTO saida
    FROM funcionarios
    WHERE id = id_func;
END$

DELIMITER ;


CALL aumenta_salario_v2(2, 10, @saida);
SELECT @saida;
```


## Exercício 2 - Procedure simples 2 (ajuste por setor + log) Crie uma procedure chamada ajustar_salarios_setor que recebe id_setor e percentual, atualiza os salários de todos os funcionários do setor, registra o ajuste em log e retorna os novos salários.
Tabelas necessárias
CREATE TABLE setores (
 id INT PRIMARY KEY AUTO_INCREMENT,
 nome VARCHAR(100)
);
CREATE TABLE funcionarios (
 id INT PRIMARY KEY AUTO_INCREMENT,
 nome VARCHAR(100),
 salario DECIMAL(10,2),
 setor_id INT,
 FOREIGN KEY (setor_id) REFERENCES setores(id)
);
CREATE TABLE ajustes_salariais (
 id_ajuste INT PRIMARY KEY AUTO_INCREMENT,
 setor_id INT,
 percentual_aplicado DECIMAL(5,2),
 data_ajuste DATETIME
);
INSERT INTO setores (nome) VALUES ('TI'), ('RH'), ('Vendas');
INSERT INTO funcionarios (nome, salario, setor_id) VALUES
('Ana', 2500.00, 1),
('Bruno', 3000.00, 1),
('Carla', 4000.00, 2),
('Diego', 3500.00, 3);

```sql
DELIMITER $

CREATE PROCEDURE ajustar_salarios_setor(
    IN id_setor INT,
    IN percentual DECIMAL(5,2)
)
BEGIN
    DECLARE data_atual DATETIME;
    SET data_atual = NOW();
       
    UPDATE funcionarios 
    SET salario = salario * (1 + percentual/100)
    WHERE setor_id = id_setor;    

    INSERT INTO ajustes_salariais (setor_id, percentual_aplicado, data_ajuste)
    VALUES (id_setor, percentual, data_atual);
    
    SELECT id, nome, salario AS novo_salario
    FROM funcionarios
    WHERE setor_id = id_setor;
END$

DELIMITER ;

CALL ajustar_salarios_setor(1,500);
                           
```

## Exercício 3 - Procedure com laço WHILE Crie uma procedure chamada inserir_numeros(qtd INT) que insere valores de 1 até qtd na tabela numeros usando um laço WHILE.
Tabelas necessárias
CREATE TABLE numeros (
 id INT PRIMARY KEY AUTO_INCREMENT,
 valor INT
);


```sql
DELIMITER $

CREATE PROCEDURE inserir_numeros(
    IN qtd INT
)
BEGIN
    DECLARE I INT;
    SET I = 1;
   	calculo:WHILE (I<=qtd) DO
	INSERT INTO numeros(numeros.valor)
VALUES(I) ; 
 SET I = I +1;
 END WHILE calculo;
   
END$

DELIMITER ;

CALL inserir_numeros(5);
```

## Exercício 4 - Cursor 1 (listar por setor) Crie uma procedure listar_funcionarios_cursor(p_setor_id) que percorre via cursor os funcionários do setor e exibe nome e salário.

```sql 
DELIMITER $

CREATE OR REPLACE PROCEDURE listar_funcionarios_cursor(IN p_setor_id INT)
BEGIN
    DECLARE v_nome VARCHAR(100);
    DECLARE v_salario DECIMAL(10,2);
    DECLARE v_fim INT DEFAULT 0;
    
  
    DECLARE cur_funcionarios CURSOR FOR
    SELECT nome, salario
    FROM funcionarios
    WHERE setor_id = p_setor_id;
    
   
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET v_fim = 1;
    
   
    CREATE TEMPORARY TABLE temp_resultados (
        nome VARCHAR(100),
        salario DECIMAL(10,2)
    );
    
    
    OPEN cur_funcionarios;
    
    
    loop_cursor: LOOP
        FETCH cur_funcionarios INTO v_nome, v_salario;
        
        IF v_fim = 1 THEN
            LEAVE loop_cursor;
        END IF;
        
      
        INSERT INTO temp_resultados VALUES (v_nome, v_salario);
    END LOOP loop_cursor;
    
    
    CLOSE cur_funcionarios;
    
    
    SELECT * FROM temp_resultados;
    
   
    DROP TEMPORARY TABLE temp_resultados;
END$

DELIMITER ;

CALL listar_funcionarios_cursor(1);
```

## Exercício 5 – Cursor 2 (ajuste por faixas + log) Crie uma procedure ajustar_salarios_cursor(p_setor_id) que percorre via cursor os funcionários do setor e aplica reajuste salarial como segue: < 3000: +10%; 3000–5000: +5%; > 5000: sem ajuste. Registre mudanças em ajustes_cursor_log.

```sql
DELIMITER $$

CREATE PROCEDURE ajustar_salarios_cursor(p_setor_id INT)
BEGIN
 
    DECLARE v_id INT;
    DECLARE v_salario DECIMAL(10,2);
    DECLARE v_novo_salario DECIMAL(10,2);
    DECLARE done INT DEFAULT 0;

    DECLARE cur CURSOR FOR
        SELECT id, salario
        FROM funcionarios
        WHERE setor_id = p_setor_id;

    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

    OPEN cur;

    read_loop: LOOP
        FETCH cur INTO v_id, v_salario;
        IF done = 1 THEN
            LEAVE read_loop;
        END IF;


        IF v_salario < 3000 THEN
            SET v_novo_salario = v_salario * 1.10;
        ELSEIF v_salario BETWEEN 3000 AND 5000 THEN
            SET v_novo_salario = v_salario * 1.05;
        ELSE
            SET v_novo_salario = v_salario;
        END IF;

        IF v_novo_salario <> v_salario THEN
            UPDATE funcionarios
            SET salario = v_novo_salario
            WHERE id = v_id;

            INSERT INTO ajustes_cursor_log (id_func, salario_antigo, salario_novo, data_ajuste)
            VALUES (v_id, v_salario, v_novo_salario, NOW());
        END IF;
    END LOOP;

    CLOSE cur;
END$$

DELIMITER ;

CALL ajustar_salarios_cursor(1);


```