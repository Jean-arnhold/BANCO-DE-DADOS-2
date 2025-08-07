## Exercício 3
Você é o responsável pelo banco de dados de uma empresa e recebeu uma tabela antiga populada com alguns dados de exemplo. Porém, no sistema atual da empresa as tabelas foram projetadas considerando o processo de normalização. Como responsável, você precisa definir um conjunto de comandos SQL para inserir os dados da tabela recebida nas tabelas do sistema atual garantindo a integridade dos dados. Considere que:

a. Considere que você recebeu nesta atividade a tabela antiga com apenas parte dos dados. Quando seus comandos SQL forem executados na tabela com todos os dados, os comandos devem funcionar corretamente. Tabela antiga: nota_antiga.

b. Você recebeu uma cópia sem dados da estrutura das tabelas atuais do sistema. Porém, seus comandos SQLs devem considerar que já existam dados nas tabelas e que estes não podem ser duplicados nem perdidos. Note que em algumas tabelas do sistema atual os atributos chaves são AUTO_INCREMENT. Tabelas atuais do sistema: cliente, item_nota, nota, produto, vendedor.

c. Adicionalmente, você deve adicionar na tabela item_nota uma coluna calculada (armazenada) para o cálculo do total do item vendido naquele registro (preco * quantidade). Nomear a coluna de total.

d. Por fim, na tabela antiga a coluna vl_total_nota está com o valor igual a 0 (zero) para todas as compras. Você deve, já no banco atual do sistema (após a inserção dos dados), criar um comando SQL para atualizar o valor da coluna valorTotal conforme as vendas realizadas em cada nota.
Tabela recebida
nota_antiga:
nr_nota, nome_cliente, cpf_cliente, ds_endereco_cliente, nome_vendedor, data_emissao,
cd_prod1, ds_prod1, unidade1, quantidade1, preco1, vl_total1,
cd_prod2, ds_prod2, unidade2, quantidade2, preco2, vl_total2,
cd_prod3, ds_prod3, unidade3, quantidade3, preco3, vl_total3,
cd_prod4, ds_prod4, unidade4, quantidade4, preco4, vl_total4,
vl_total_nota
Entregas da questão:
 Lista de comandos SQL em ordem correta de execução considerando os dados que devem ser incluídos antes dos outros.


```sql

-- 1. Desativa temporariamente as restrições de chave estrangeira
SET FOREIGN_KEY_CHECKS = 0;

-- 2. Insere clientes (evitando duplicatas por CPF)
INSERT IGNORE INTO cliente (nome, cpf, endereco)
SELECT DISTINCT nome_cliente, cpf_cliente, ds_endereco_cliente
FROM nota_antiga
WHERE cpf_cliente NOT IN (SELECT cpf FROM cliente);

-- 3. Insere vendedores (evitando duplicatas por nome)
INSERT IGNORE INTO vendedor (nome)
SELECT DISTINCT nome_vendedor
FROM nota_antiga
WHERE nome_vendedor NOT IN (SELECT nome FROM vendedor);

-- 4. Insere produtos (evitando duplicatas por código e descrição)
INSERT IGNORE INTO produto (idProduto, descricao, unidade, preco)
SELECT DISTINCT 
    cd_prod1, ds_prod1, unidade1, preco1
FROM nota_antiga
WHERE cd_prod1 IS NOT NULL AND cd_prod1 NOT IN (SELECT idProduto FROM produto)
UNION
SELECT DISTINCT 
    cd_prod2, ds_prod2, unidade2, preco2
FROM nota_antiga
WHERE cd_prod2 IS NOT NULL AND cd_prod2 NOT IN (SELECT idProduto FROM produto)
UNION
SELECT DISTINCT 
    cd_prod3, ds_prod3, unidade3, preco3
FROM nota_antiga
WHERE cd_prod3 IS NOT NULL AND cd_prod3 NOT IN (SELECT idProduto FROM produto)
UNION
SELECT DISTINCT 
    cd_prod4, ds_prod4, unidade4, preco4
FROM nota_antiga
WHERE cd_prod4 IS NOT NULL AND cd_prod4 NOT IN (SELECT idProduto FROM produto);

-- 5. Insere notas (usando os IDs corretos de cliente e vendedor)
INSERT INTO nota (num_nota, idCliente, idVendedor, data_emissao, valorTotal)
SELECT 
    na.nr_nota,
    c.idCliente,
    v.idVendedor,
    na.data_emissao,
    0 -- Será atualizado posteriormente
FROM nota_antiga na
JOIN cliente c ON na.cpf_cliente = c.cpf
JOIN vendedor v ON na.nome_vendedor = v.nome
WHERE na.nr_nota NOT IN (SELECT num_nota FROM nota);

-- 6. Adiciona coluna calculada total na tabela item_nota
ALTER TABLE item_nota
ADD COLUMN total DECIMAL(10,2) GENERATED ALWAYS AS (preco * quantidade) STORED;

-- 7. Insere itens das notas (para todos os produtos não nulos)
INSERT INTO item_nota (idNota, idProd, quantidade, preco)
SELECT 
    n.idNota,
    na.cd_prod1,
    na.quantidade1,
    na.preco1
FROM nota_antiga na
JOIN nota n ON na.nr_nota = n.num_nota
WHERE na.cd_prod1 IS NOT NULL

UNION ALL

SELECT 
    n.idNota,
    na.cd_prod2,
    na.quantidade2,
    na.preco2
FROM nota_antiga na
JOIN nota n ON na.nr_nota = n.num_nota
WHERE na.cd_prod2 IS NOT NULL

UNION ALL

SELECT 
    n.idNota,
    na.cd_prod3,
    na.quantidade3,
    na.preco3
FROM nota_antiga na
JOIN nota n ON na.nr_nota = n.num_nota
WHERE na.cd_prod3 IS NOT NULL

UNION ALL

SELECT 
    n.idNota,
    na.cd_prod4,
    na.quantidade4,
    na.preco4
FROM nota_antiga na
JOIN nota n ON na.nr_nota = n.num_nota
WHERE na.cd_prod4 IS NOT NULL;

-- 8. Atualiza o valorTotal das notas com a soma dos itens
UPDATE nota n
SET n.valorTotal = (
    SELECT SUM(total)
    FROM item_nota it
    WHERE it.idNota = n.idNota
);

-- 9. Reativa as restrições de chave estrangeira
SET FOREIGN_KEY_CHECKS = 1; ```