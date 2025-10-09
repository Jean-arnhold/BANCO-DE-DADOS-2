## Considerando as tabelas abaixo (que devem ser criadas por você), crie triggers conforme solicitado a seguir:
1. Criar uma trigger para atualizar o estoque na tabela produtos toda vez que um produto for vendido (neste exercício vender significa inserir na tabela itensvenda).


```sql
DELIMITER $$

CREATE TRIGGER atualizaEstoque
AFTER INSERT ON itensvenda
FOR EACH ROW
BEGIN
    UPDATE produtos
    SET estoque = estoque - NEW.quantidade
    WHERE produto_id = NEW.produto_id;
END$$

DELIMITER ;

```
2. Criar uma trigger para atualizar o estoque na tabela produtos toda vez que um for devolvido (neste exercício devolver significa apagar da tabela itensvenda).

```sql
DELIMITER $$

CREATE OR REPLACE TRIGGER devolveEstoque
AFTER DELETE ON itensvenda
FOR EACH ROW
BEGIN
    UPDATE produtos
    SET estoque = estoque + OLD.quantidade
    WHERE produto_id = OLD.produto_id;
END$$

DELIMITER ;
```

3. Criar uma trigger para atualizar o estoque na tabela produtos toda vez que a quantidade
de um produto vendido for alterado (neste exercício significa alterar a quantidade na
tabela itensvenda).


```sql
DELIMITER $$

CREATE TRIGGER atualiza_estoque_apos_update
BEFORE UPDATE ON itensvenda
FOR EACH ROW
BEGIN
    UPDATE produtos
    SET estoque = estoque + OLD.quantidade - NEW.quantidade
    WHERE produto_id = NEW.produto_id;
END $$

DELIMITER ;
```

## Exercício 2 - triggers 
Considere o banco de dados Floricultura.

1 - Criar uma trigger para atualizar o estoque na tabela Flor toda vez que uma flor for vendida (um novo item for inserido na tabela ItensVenda).

```sql 
DELIMITER $$

CREATE TRIGGER atualiza_estoque_venda
AFTER INSERT ON itensvenda
FOR EACH ROW
BEGIN
    UPDATE flor 
    SET estoque = estoque - NEW.quantidade 
    WHERE idFlor = NEW.idFlor;
END$$

DELIMITER ;
```
2- Criar uma trigger para inserir na tabela ComprasDevolvidas os dados referentes a uma devolução total de uma venda (uma linha da tabela ItensVenda for deletada).

```Sql
DELIMITER $$

CREATE TRIGGER registra_devolucao_total
AFTER DELETE ON itensvenda
FOR EACH ROW
BEGIN
    
     INSERT INTO comprasdevolvidas (idVenda, idFlor, quantidade, dataCompra, dataDevolucao)
     VALUES (OLD.idVenda, OLD.idFlor, OLD.quantidade, OLD.dataCompra, CURDATE());

     UPDATE flor 
     SET estoque = estoque + OLD.quantidade 
     WHERE idFlor = OLD.idFlor;
END$$

DELIMITER ;
```

3 - Criar uma trigger para apagar a linha da tabela de promoção de uma flor quando a mesma for inseridas na tabela ListaCompras.

```Sql
DELIMITER //

CREATE TRIGGER remove_promocao_lista_compras
AFTER INSERT ON listacompras
FOR EACH ROW
BEGIN
    DELETE FROM promocao 
    WHERE idFlor = NEW.idFlor;
END//

DELIMITER ;
```

4 - Criar uma trigger para inserir uma flor na tabela lista de compras toda vez que seu estoque for menor ou igual a quantidade mínima de estoque. A quantidade a ser comprada deve ser a diferença entre estoque máximo e o estoque atual.

```sql
DELIMITER $$

CREATE TRIGGER verifica_estoque_minimo
AFTER UPDATE ON flor
FOR EACH ROW
BEGIN
    DECLARE quantidade_comprar INT;
    
    IF NEW.estoque <= NEW.estoque_min THEN

        SET quantidade_comprar = NEW.estoque_max - NEW.estoque;    
 
        IF NOT EXISTS (SELECT 1 FROM listacompras WHERE idFlor = NEW.idFlor) THEN
            INSERT INTO listacompras (idFlor, quantidade)
            VALUES (NEW.idFlor, quantidade_comprar);
        END IF;
    END IF;
END$$

DELIMITER ;
```