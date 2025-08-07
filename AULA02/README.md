## Você é o responsável pelo banco de dados de uma empresa e percebeu que a tabela funcionarios possui os nomes dos departamentos diretamente inseridos nela. Você identifica que seria interessante ter uma tabela para armazenar as informações dos departamentos e na tabela funcionarios apenas fazer referência ao dados desta nova tabela. Crie um script para fazer todas as alterações necessárias. O script deve estar em um único arquivo que será importado para realizar todas as alterações.

```sql
CREATE TABLE IF NOT EXISTS departamentos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    nome_departamento VARCHAR(30) NOT NULL UNIQUE,
    cpf_gerente BIGINT(11) NOT NULL,
    FOREIGN KEY (cpf_gerente) REFERENCES funcionario(Cpf)
);
``` 
-- 2. Popula a tabela de departamentos com dados existentes
```sql
INSERT INTO departamentos (nome_departamento, cpf_gerente)
SELECT DISTINCT departamento, gerente_departamento 
FROM funcionario;
``` 
-- 3. Adiciona coluna de referência ao departamento na tabela 
```sql
funcionario
ALTER TABLE funcionario
ADD COLUMN id_departamento INT NOT NULL AFTER departamento;
``` 
-- 4. Atualiza a referência aos departamentos
```sql
UPDATE funcionario f
JOIN departamentos d ON f.departamento = d.nome_departamento
SET f.id_departamento = d.id;
``` 
-- 5. Cria a chave estrangeira
```sql
ALTER TABLE funcionario
ADD CONSTRAINT fk_func_departamento
FOREIGN KEY (id_departamento) REFERENCES departamentos(id);
``` 
-- 6. Remove a coluna departamento original
```sql

ALTER TABLE funcionario
DROP COLUMN departamento;

``` 

-- 7. Remove a coluna gerente_departamento (agora na tabela departamentos)
```sql 
ALTER TABLE funcionario
DROP COLUMN gerente_departamento;
``` 