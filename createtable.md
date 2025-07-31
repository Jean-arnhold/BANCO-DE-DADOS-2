# Exercício 1

Você é o responsável pelo banco de dados de uma empresa recebeu uma tabela populada com dados de pessoas físicas e pessoas júridicas. A tabela possui as seguintes colunas: pessoa_id(pk), nome, tipo_pessoa, cpf_cnpj, rg_inscricao_estadual, fone, endereco, distrito, cidade, cep, pais, last_update.

No sistema atual da empresa, as tabelas foram projetadas considerando alguns conceitos de especialização e, desta forma, estão estruturadas de maneira diferente da tabela recebida.

Como responsável, você precisa definir um conjunto de passos e comandos SQL para incluir os dados da tabela recebida nas tabelas do sistema garantindo a integridade dos dados.

As tabelas e colunas atuais do sistema, que precisarão ser criadas por você, são:

- pessoa: pessoa_id(pk), nome, endereco, distrito, cidade, cep, pais, last_update
- pessoa_juridica: pessoa_id(fk), cnpj, inscricao_estadual
- pessoa_fisica: pessoa_id(fk), cpf, rg.

Além disso, a tabela recebida possui apenas uma coluna para adição de telefone. Você, como responsável, precisa repensar essa estrutura para permitir que um ou mais telefones (não existe uma quantidade fixa) possam ser adicionados (tanto para pessoas físicas quanto jurídicas).
Entregas do exercício 1:

- Lista de passos e comandos SQL executados;
- Banco final com as tabelas criadas e os dados inseridos.

Observação: O script para criação da tabela original e inserção dos dados encontra-se no Moodle da disciplina pra download.

## Criação das Tabelas:

### Tabela pessoa:

```sql
CREATE TABLE pessoa
SELECT pessoa_id, nome, endereco, distrito, cidade, cep, pais, last_update
FROM
pessoaold;
```

### Tabela pessoa_juridica:

```sql
CREATE TABLE pessoa_juridica
SELECT pessoa_id, pessoaold.cpf_cnpj AS CNPJ, pessoaold.rg_inscricao_estadual AS INSCRICAO_ESTADUAL
FROM
pessoaold
WHERE pessoaold.tipo_pessoa = "PJ";
```

```sql 
ALTER TABLE pessoa_juridica
ADD FOREIGN KEY(pessoa_id) REFERENCES pessoa(pessoa_id);
```

### Tabela pessoa_fisica:

```sql
CREATE TABLE pessoa_fisica
SELECT pessoa_id, pessoaold.cpf_cnpj AS CPF, pessoaold.rg_inscricao_estadual AS RG
FROM
pessoaold
WHERE pessoaold.tipo_pessoa = "PF";
```

```sql
ALTER TABLE pessoa_fisica
ADD FOREIGN KEY(pessoa_id) REFERENCES pessoa(pessoa_id);
```

### Tabela telefone_pessoa:

```sql
CREATE TABLE telefone_pessoa
SELECT pessoa_id, fone
FROM 
pessoaold;
```

```sql
ALTER TABLE telefone_pessoa ADD FOREIGN KEY(pessoa_id) REFERENCES pessoa(pessoa_id);
```



