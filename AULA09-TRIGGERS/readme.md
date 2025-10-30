## Atividade 2 - triggers
Considere uma tabela de pedidos de clientes em que seja necessário limitar que um cliente faça no máximo 5 pedidos por dia. Crie um trigger que, antes de inserir um novo pedido na tabela pedidos, verifique se o cliente já fez o máximo de 5 pedidos no mesmo dia e, se sim, recuse a operação.
Estrutura de Tabelas:
-- Tabela clientes
CREATE TABLE clientes (
id INT AUTO_INCREMENT PRIMARY KEY,
nome VARCHAR(40),
cidade VARCHAR(40)
);
-- Tabela pedidos
CREATE TABLE pedidos (
id INT AUTO_INCREMENT PRIMARY KEY,
cliente_id INT,
data_pedido DATE,
valor_total DECIMAL(10, 2),
FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);

```sql
CREATE TRIGGER limitar_pedidos_por_dia
BEFORE INSERT ON pedidos
FOR EACH ROW
BEGIN
    DECLARE total_pedidos INT;

    SELECT COUNT(*) INTO total_pedidos
    FROM pedidos
    WHERE cliente_id = NEW.cliente_id
      AND data_pedido = NEW.data_pedido;

    IF total_pedidos >= 5 THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Limite de 5 pedidos por cliente por dia atingido.';
    END IF;
END;
```

```sql
Teste

INSERT INTO clientes (nome, cidade)
VALUES ('João', 'Curitiba');

-- Faz 5 pedidos no mesmo dia
INSERT INTO pedidos (cliente_id, data_pedido, valor_total)
VALUES (1, '2025-10-29', 100.00),
       (1, '2025-10-29', 200.00),
       (1, '2025-10-29', 150.00),
       (1, '2025-10-29', 80.00),
       (1, '2025-10-29', 50.00);

-- 6º pedido -> deve falhar
INSERT INTO pedidos (cliente_id, data_pedido, valor_total)
VALUES (1, '2025-10-29', 60.00);

```


```sql
Solução professor


DELIMITER $$

CREATE TRIGGER limite_pedidos_por_dia
BEFORE INSERT ON pedidos
FOR EACH ROW
BEGIN
    -- Verifica quantos pedidos o cliente já fez no mesmo dia
    DECLARE pedidos_do_dia INT;
    
    SELECT COUNT(*) INTO pedidos_do_dia
    FROM pedidos
    WHERE cliente_id = NEW.cliente_id
    AND data_pedido = NEW.data_pedido;
    
    -- Se já fez 5 ou mais pedidos, gera um erro
    IF pedidos_do_dia >= 5 THEN
        SIGNAL SQLSTATE '45000' 
        SET MESSAGE_TEXT = 'O cliente já atingiu o limite de 5 pedidos por dia.';
    END IF;
END$$

DELIMITER ;
```

## Atividade 5 – triggers
# O IFRS-Campus Feliz está organizando um evento com palestras e oficinas ao longo de 5 dias.
Os participantes (incluindo alunos, docentes e técnicos administrativos) podem se inscrever
em diferentes atividades oferecidas durante o evento, mas existem três restrições importantes:
1. Cada participante pertence a uma categoria (por exemplo, "ensino médio",
"graduação", "pós-graduação", "docente", "técnico administrativo"), e o limite
diário de inscrições é definido pela categoria do participante. O limite de inscrições
diárias por categoria está armazenado na tabela categorias.
2. Cada atividade tem um limite máximo de vagas, que deve ser respeitado.
3. Em cada dia, pode haver um limite máximo de participantes inscritos de cada
categoria, controlado pela tabela categorias, que define quantos participantes de uma
categoria podem se inscrever em atividades no mesmo dia.
Você deve criar uma trigger que, antes de permitir que o participante se inscreva em uma nova
atividade na tabela inscricoes, verifique três condições:
 Se o participante já atingiu o limite diário de inscrições permitido pela sua categoria.
 Se o limite de vagas para a atividade já foi preenchido.
 Se o número total de inscrições feitas por participantes da mesma categoria no mesmo
dia já atingiu o limite diário definido para essa categoria.
Caso qualquer uma dessas condições seja violada, a operação deve ser bloqueada e uma
mensagem de erro personalizada deve ser exibida usando SIGNAL SQLSTATE.
Estrutura das Tabelas:
-- Tabela categorias (Define o limite diário de inscrições e limite de
participantes por dia)
CREATE TABLE categorias (
 id INT AUTO_INCREMENT PRIMARY KEY,
 nome_categoria VARCHAR(40),
 limite_diario INT, -- Limite de inscrições diárias por
participante da categoria
 limite_participantes_diario INT -- Limite de participantes da
categoria que podem se inscrever por dia
);
-- Tabela participantes (Guarda informações sobre os participantes e a
qual categoria pertencem)
CREATE TABLE participantes (
 id INT AUTO_INCREMENT PRIMARY KEY,
 nome VARCHAR(40),
 categoria_id INT, -- Relaciona o participante a uma categoria
 FOREIGN KEY (categoria_id) REFERENCES categorias(id)
);
-- Tabela atividades (Contém as palestras e oficinas do evento, com o
limite de vagas)
CREATE TABLE atividades (
 id INT AUTO_INCREMENT PRIMARY KEY,
 nome_atividade VARCHAR(100),
 data_atividade DATE,
 limite_vagas INT -- Limite máximo de vagas para a atividade
);
-- Tabela inscricoes (Registra as inscrições dos participantes nas
atividades)
CREATE TABLE inscricoes (
 id INT AUTO_INCREMENT PRIMARY KEY,
 participante_id INT,
 atividade_id INT,
 data_inscricao DATE,
 FOREIGN KEY (participante_id) REFERENCES participantes(id),
 FOREIGN KEY (atividade_id) REFERENCES atividades(id)
);
Dados no banco - descrição:
 Atividades: Cada dia tem 5 atividades diferentes com limites variados de vagas.
 Participantes: 10 participantes de cada categoria foram inseridos e distribuídos ao
longo de 5 dias de evento.
 Inscrições: Há pelo menos 20 inscrições por dia, distribuídas entre as diferentes
atividades e categorias. Cada atividade tem 4 inscrições.


```sql

DELIMITER $$

CREATE TRIGGER verificar_restricoes_inscricao
BEFORE INSERT ON inscricoes
FOR EACH ROW
BEGIN
    DECLARE categoria_participante INT;
    DECLARE limite_diario_participante INT;
    DECLARE limite_participantes_categoria INT;
    DECLARE limite_vagas_atividade INT;
    DECLARE qtd_participante INT;
    DECLARE qtd_vagas INT;
    DECLARE qtd_categoria_dia INT;

    SELECT p.categoria_id, c.limite_diario, c.limite_participantes_diario
    INTO categoria_participante, limite_diario_participante, limite_participantes_categoria
    FROM participantes p
    JOIN categorias c ON p.categoria_id = c.id
    WHERE p.id = NEW.participante_id;

    SELECT COUNT(*) INTO qtd_participante
    FROM inscricoes
    WHERE participante_id = NEW.participante_id
      AND data_inscricao = NEW.data_inscricao;

    IF qtd_participante >= limite_diario_participante THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Erro: participante atingiu o limite diário de inscrições para sua categoria.';
    END IF;

    SELECT limite_vagas INTO limite_vagas_atividade
    FROM atividades
    WHERE id = NEW.atividade_id;

    SELECT COUNT(*) INTO qtd_vagas
    FROM inscricoes
    WHERE atividade_id = NEW.atividade_id;

    IF qtd_vagas >= limite_vagas_atividade THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Erro: limite de vagas da atividade atingido.';
    END IF;

    SELECT COUNT(*) INTO qtd_categoria_dia
    FROM inscricoes i
    JOIN participantes p ON i.participante_id = p.id
    WHERE p.categoria_id = categoria_participante
      AND i.data_inscricao = NEW.data_inscricao;

    IF qtd_categoria_dia >= limite_participantes_categoria THEN
        SIGNAL SQLSTATE '45000'
        SET MESSAGE_TEXT = 'Erro: limite diário de participantes dessa categoria atingido para o dia.';
    END IF;

END$$

DELIMITER ;

```