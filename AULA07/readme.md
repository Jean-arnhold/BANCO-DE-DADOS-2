## 1 Criar um procedimento que atualize os salários dos funcionários de um departamento de acordo com o especificado na chamada do procedimento. O procedimento deverá receber como parâmetro de entrada: -nome do departamento -faixa de aumento Internamente o procedimento deve considerar a faixa de aumento da seguinte forma:
Se faixa de aumento:
"A" - então aumentar salário em 10%
"B" - então aumentar salário em 20%
"C" - então aumentar salário em 30%
O procedimento deverá ter como saída:
-número de funcionários do departamento que tiveram o salário atualizado e a soma total dos
salários do departamento após a atualização.


```sql
DELIMITER $
CREATE PROCEDURE atualizaSalario(IN nome_departamento VARCHAR(25), IN faixa_aumento CHAR(1), OUT saida VARCHAR(200))
BEGIN
	DECLARE numDepto int;
	DECLARE percentual_aumento double;
	DECLARE soma_salarios_atuais DECIMAL(10,2);
   	DECLARE soma_salarios_novos DECIMAL(10,2);
   	DECLARE registros_atualizados int;
    
	SELECT Dnumero INTO numDepto FROM departamento WHERE Dnome = nome_departamento;
	SET percentual_aumento = 0.00;		
	
	CASE faixa_aumento
	   WHEN "A" THEN 
		SET percentual_aumento = 0.10;
	   WHEN "B" THEN 	
		SET percentual_aumento = 0.20;        	
	   WHEN "C" THEN 
		SET percentual_aumento = 0.30;				
	END CASE;

	SELECT COUNT(cpf) INTO registros_atualizados FROM funcionario WHERE Dnr = numDepto;
	SELECT SUM(salario) INTO soma_salarios_atuais FROM funcionario WHERE Dnr = numDepto;
	
	UPDATE funcionario SET salario = salario * (1+percentual_aumento) WHERE Dnr = numDepto;
	
	SELECT SUM(salario) INTO soma_salarios_novos FROM funcionario WHERE Dnr = numDepto;
	
	
	SET saida = CONCAT("Departamento: ", nome_departamento, " | Funcionários atualizados: ", registros_atualizados, " | Aumento: ", percentual_aumento*100,"% | Soma antiga dos salários: ", soma_salarios_atuais, " | Soma atual dos salários: ", soma_salarios_novos);
END $$
DELIMITER ;


#TESTANDO
call atualizaSalario("Pesquisa","A", @saida);
SELECT @saida;
```

## 2 Criar uma tabela chamada funcionarios_projetos para armazenar o cpf, o nome completo do funcionário (em uma única coluna), a quantidade de projetos e a quantidade de horas que cada funcionário trabalha. Criar um procedimento para manter atualizados os dados da tabela funcionarios_projetos baseado nos dados das outras tabelas do banco. Cuidar para não duplicar os dados na tabela funcionarios_projetos quando o procedimento for executado mais de uma vez.

```sql
DELIMITER $$

CREATE PROCEDURE atualizar_funcionarios_projetos()
BEGIN

    DECLARE v_cpf VARCHAR(11);
    DECLARE v_nome VARCHAR(100);
    DECLARE v_sobrenome VARCHAR(100);
    DECLARE v_nome_completo VARCHAR(150);
    DECLARE v_qtd_projetos INT;
    DECLARE v_qtd_horas INT;
    DECLARE done INT DEFAULT 0;


    DECLARE cur CURSOR FOR
        SELECT Cpf, Pnome, Unome FROM funcionario;

   
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

   
    TRUNCATE TABLE funcionarios_projetos;

    
    OPEN cur;

    read_loop: LOOP
        FETCH cur INTO v_cpf, v_nome, v_sobrenome;
        IF done = 1 THEN
            LEAVE read_loop;
        END IF;
       
        SET v_nome_completo = CONCAT(v_nome, ' ', v_sobrenome);

       
        SELECT COUNT(DISTINCT te.Pnr), COALESCE(SUM(te.Horas), 0)
        INTO v_qtd_projetos, v_qtd_horas
        FROM trabalha_em te 
        WHERE te.Fcpf = v_cpf;

        INSERT INTO funcionarios_projetos (cpf, nomeCompleto, qtdProj, qtdHoras)
        VALUES (v_cpf, v_nome_completo, v_qtd_projetos, v_qtd_horas);
    END LOOP;

    CLOSE cur;
END$$

DELIMITER ;

CALL atualizar_funcionarios_projetos();

SELECT * FROM funcionarios_projetos;
```

## 3 Crie uma procedure para retornar concatenados os produtos que tenham mais de certa quantidade em estoque. 
Entrada: quantidade a ser verificada. 
Saída: lista concatenada de nome de produtos com mais do que a quantidade informada

```sql 
DELIMITER $$

CREATE PROCEDURE listar_produtos_concatenados(IN p_qtd INT, OUT p_lista TEXT)
BEGIN

    DECLARE v_desc VARCHAR(100);
    DECLARE done INT DEFAULT 0;
    DECLARE cur CURSOR FOR
        SELECT descricao FROM produto WHERE estoque > p_qtd;
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = 1;

 
    SET p_lista = '';

    OPEN cur;

    read_loop: LOOP
        FETCH cur INTO v_desc;
        IF done = 1 THEN
            LEAVE read_loop;
        END IF;

        IF p_lista = '' THEN
            SET p_lista = v_desc;
        ELSE
            SET p_lista = CONCAT(p_lista, ', ', v_desc);
        END IF;
    END LOOP;

    CLOSE cur;
END$$

DELIMITER ;

CALL listar_produtos_concatenados(100, @lista);
SELECT @lista;
```