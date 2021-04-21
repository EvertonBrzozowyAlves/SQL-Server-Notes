# SQL Server 

Bancos de dados são coleções de dados que se relacionam de forma a criar algum sentido (informação).
Os dados são organizados através de uma estrutura de duas dimensões (**Linhas** x **Colunas**), denominada **Tabela**, onde as linhas representam **registros** e as colunas os **campos**.

## Criando novo banco de dados - SSMS

Usando o SGBD (Sistema Gerenciador de Banco de Dados) SSMS (SQL Server Management Studio), temos várias funções que ajudam no processo de administração do banco de dados.  
Podemos criar uma solução que contenha todos os scripts necessários para gerir um projeto.

Para criar um novo banco de dados, devemos informar:

- Nome do banco de dados
- Owner (se não for informado, será assumido o usuário SA)

Quando é criado um banco de dados, fisicamente, são criados dois arquivos:

- Arquivo de dados (**.mdf**)
- Arquivo de log (**.ldf**)

Podemos escolher a pasta onde serão salvos esses arquivos.

Após criado o banco de dados, e adicionado automaticamente um novo node na aba **DataBases**, já com subitems.

## Criando Tabelas - SSMS

Para criar as Tabelas, selecionamos o banco de dados e sobre a pasta, **Tables**, escolhemos a opção **New Table**. Devemos informar:
- Nome do Campo
- Tipo de Dados / Tamanho do Campo
- Regra de Nullabilidade

>Evite usar acentos ou espaços em nome de tabelas

Podemos definir uma chave primária clicando em uma das linhas com o botão direito e escolher a opção **set primary key**.
Depois de terminar, clique em salvar para adicionar um nome para a tabela.
Apesar de não ser obrigatório, é uma boa prática de programação em banco de dados adicionar um prefixo **tbl** ao nome da tabela.



## Inserindo/Alterando Dados - SSMS

Podemos inserir ou alterar dados facilmente, utilizando a opção **Edit top 200 rows**. Nesse modo podemos inserir / alterar dados em modeo de digitação.

>O padrão para inserção de datas é AAAA-MM-DD mas, nesse modo, ao alterar o foco do campo é assumido esse formato automaticamente.

Esse modo é ideal para trabalhar com pequena quantidade de dados. Para grandes quantidades, usamos a linguagem T-SQL.


## Modelo Relacional e a Normalização de Dados

A normalização consiste em um conjunto de procedimentos que permitirá termos dados consistentes e com baixa redundância.   
Como muitos gerenciadores de banco não tem separação suficiente entre o projeto lógico da base de dados e a implementação física do banco de dados, isto traz como consequência perda de desempenho nas consultas.  
Em casos mais extremos é sugerido trabalhar com dados desnormalizados.  

### Formas de Normalização

#### First Normal Form (1NF)

Nesta forma, cada coluna deve ter um único valor exclusivo (nenhum dado se repete):

| Código | Gerente | Departamento |
| :----: | :----:  | :----:       |
| 12345  | Ana     | RH           |
| 54321  | Carlos  | TI           |

#### Normalizando (1NF)

Imagine uma tabela de clientes em que seja possível armazenar vários números de telefone para o mesmo cliente:

| Código | Gerente | Departamento | Telefone  | Celular  | TelefoneRecado  |
| :----: | :----:  | :----:       | :----:    | :----:   | :----:          |
| 12345  | Ana     | RH           | 99999999  | 99999999 | 77777777        |
| 54321  | Carlos  | TI           | 88888888  | 88888888 | 66666666        |

Percebemos aqui que existe um conjunto de informações relacionadas a mesma informação (telefone). Sempre que uma linha de uma tabela tiver N informações relacionadas a ela, recomenda-se criar outra tabela para armazenar essas N informações:


| Código | Gerente | Departamento |            
| :----: | :----:  | :----:       |      
| 12345  | Ana     | RH           |
| 54321  | Carlos  | TI           |

----------------------------------------------------------------------

| Código | Telefone  | Celular  | TelefoneRecado  |
| :----: | :----:    | :----:   | :----:          |
| 12345  | 99999999  | 99999999 | 77777777        |
| 54321  | 88888888  | 88888888 | 66666666        |

Fazendo isso, estamos passando essa tabela da primeira forma normal (1NF) para segunda forma normal (2NF).

#### Second Normal Form (2NF)

No segundo nível de normalização, devemos criar tabelas separadas para conjuntos de valores que se aplicam a vários registros, ou seja, que se repetem:

**tblPedidos**
| Pedido | Data       | Cliente  | Entrega   | Frete    | Produto | Quantidade | ValorUnitario |
| :----: | :----:     | :----:   | :----:    | :----:   | :----:  | :----:     | :----:        |
| 12345  | 06/04/2021 | Everton  | São Paulo | R$ 15,00 | A       | 3          | R$ 25,00      |
| 12345  | 06/04/2021 | Everton  | São Paulo | R$ 15,00 | B       | 2          | R$ 38,00      |

Note que os dados *Pedido, Data, Cliente, Entrega e Frete* se repetem.
Para evitar esse tipo de redundância, aplicamos a 2NF. Essa normalização separa essa tabela em duas tabelas distintas:  

**tblPedidos**  
| Pedido | Data       | Cliente  | Entrega   | Frete    |
| :----: | :----:     | :----:   | :----:    | :----:   |
| 12345  | 06/04/2021 | Everton  | São Paulo | R$ 15,00 |

**tblPedidoDetalhes**  
| Pedido | Produto | Quantidade | ValorUnitario |
| :----: | :----:  | :----:     | :----:        |
| 12345  | A       | 3          | R$ 25,00      |
| 12345  | B       | 2          | R$ 38,00      |

Note que para essa normalização, precisamos de uma coluna para fazer a relação entre as duas tabelas. Chamamos essa coluna de chave. Nesse caso, a coluna chave é a *Pedido*. Por conta dessa necessidade, não é possível normalizar o banco de dados 100%, pois esse dado será repetido sempre que precisarmos associar mais de uma linha da tabela *tblPedidoDetalhes* com a tabela *tblPedidos*.

Essa coluna chave recebe o nome de **Primary Key (PK)** na tabela a qual ela pertence *(tblPedidos)* e de **Foreign Key (FK)** na tabela onde ela é referenciada *(tblPedidoDetalhes)*.

#### Third Normal Form (3NF)

É considerado terceiro nível de normalização somente se todas as regras do 1NF e do 2NF forem atendidas. Por fim, devemos eliminar colunas que não dependem de chaves primárias, como campos que poderiam ser resultado de cálculos: 

**tblFuncionarios** 
| Matricula | Funcionario | Admissão   | Salario    |VT         |VR         |INSS       |IR         |
| :----:    | :----:      | :----:     | :----:     |:----:     |:----:     |:----:     |:----:     |
| 12345     | Everton     | 06/04/2021 | R$ 2000,00 |R$ 120,00  |R$ 40,00   |R$ 160,00  |R$ 220,00  |
| 12345     | Joyce       | 06/04/2021 | R$ 3200,00 |R$ 192,00  |R$ 45,00   |R$ 256,00  |R$ 352,00  |

**tblFuncionarios** 
| Matricula | Funcionario | Admissão   | Salario    |
| :----:    | :----:      | :----:     | :----:     |
| 12345     | Everton     | 06/04/2021 | R$ 2000,00 |
| 12345     | Joyce       | 06/04/2021 | R$ 3200,00 |

A tabela Funcionários passaria a ter apenas esses campos, pois os campos cálculáveis podem ser gerados sempre que necessário, no momento de consultar esses dados, através da linguagem de consulta.



### Modelo Relacional

A partir do momento que entendemos o objetivo da normalização, veremos o conceito de **Modelo Relacional**.
Este modelo foi criado por Edgar Frank Codd na IBM, no início dos anos 70 e se tornou padrão de aplicações com Banco de Dados.  
Consiste na forma de termos os dados separados em tabelas que possuem algum tipo de relação entre si.  

#### Tipos de Modelos 
Para se construir um modelo relacional, será necessário passarmos pelas etapas:  

- Modelo Descritivo:  
Após o levantamento de requisitos do sistema, consiste na documentação e/ou descrição da forma na qual os dados serão armazenados.  

- Modelo Conceitual:  
É a transformação do modelo descritivo a fim de identificarmos as tabelas, também denominadas de entidades, e seus campos, também denominados de atributos.  

- Modelo Lógico:  
É a transformação do modelo conceitual em uma forma visual de diagrama.  

- Modelo Físico:  
Já é a implementação do modelo lógico e é feita dentro do aplicativo gerenciador de banco de dados (SGBD).  

#### Dicionário de Dados
Um dicionário de dados *(data dictionary)* é uma coleção de metadados que contém definições e representações de elementos de dados, ou seja, trata-se de dados sobre a estrutura:

**tblAlunos**
| Campo Lógico | Campo Físico       | Tipo         | PK        | FK (Tabela/Campo)    | Restrições                | Observações              |
| :----:       | :----:             | :----:       | :----:    | :----:               | :----:                    | :----:                   |
| Código       | CodigoAluno        | SmallInt     | PK        |                      | Não nulo / Maior que zero | campo auto incrementavel |
| Nome         | NomeAluno          | varchar(100) |           |                      | Não nulo                  |                          |






## Apresentando o T-SQL

### Origem do SQL

**SQL - Structured Query Language** é a linguagem base para se trabalhar com dados.  
Também criada por Edgar Frank Codd no início dos anos 70, na IBM, para implementar o modelo relacional.  
Com o tempo, outras fabricantes de software começaram a construir suas linguagens baseadas no SQL. Houve então a necessidade de uma padronização, a qual foi realizada pela American National Standards Institute (ANSI) em 1986 e pela ISO em 1987.  
Seu último release padronizado ocorreu em 2003.  

A partir de então, a fim de concorrência de mercado, os fabricantes de SGBDs começar a inverstir em melhorias e novos recursos do seu próprio SQL, mas mantendo o padrão ANSI.  
Principais fornecedores de software para banco de dados:  

| Fabricante  | Produto       | Linguagem                        |
| :----:      | :----:        | :----:                           |    
| Microsoft   | MS-SQL Server | T-SQL (Transact SQL)             | 
| ORACLE      | ORACLE        | PL-SQL (Procedural Language SQL) | 
| ORACLE      | MySQL         | PL-SQL                           | 
| IBM         | DB2           | SQL-PL                           | 
| Open Source | PostgreSQL    | PL/pgSQL                         | 

### Subconjuntos do SQL

- **DQL - (Data Query Language):** Parte do SQL focada em extração de dados.  
- **DML - (Data Manipulation Language):** Parte do SQL para manipulação (inserção, edição, exclusão de registros).  
- **DCL - (Data Control Language):** Parte do SQL para gerenciamento e controle de permissões.  
- **DTL - (Data Transaction Language):** Parte do SQL para trabalhar com transações que nos permite realizar operações de manipulação de dados com segurança.  
- **DDL - (Data Definition Language):** Parte do SQL para criação, edição e exclusão referentes a estruturas do objetos.  

Por padrão, os comandos T-SQL são case-insensitive.
Exemplo de comando DQL para consultar todos campos e registros de uma tabela chamada **tblClientes**:

```sql
SELECT * FROM tblClientes;
```

Onde:
- *SELECT* = Selecionar  
- *** = Todas as colunas  
- *FROM* = Origem dos dados (Tabelas ou Consultas)
- *tblClientes* = Nome do objeto
- *;* = Fim de instrução (opcional)



## Criando banco de dados via T-SQL


Caso não tenha uma solução para o projeto, é recomendável criar uma:
- File => New => Project *(CTRL + SHIT + N)*

Após criada essa solução, será gerado um arquivo com extensão .ssmssln, que poderá armazenar as queries relacionadas aquele projeto.

Caso já tenha criado o projeto, para abrir:
- File => Open => Project / Solution *(CTRL + SHIFT + O)*

No Solution Explorer *(CTRL + ALT + L)*, será exibido todo o trabalho já feito nessa solução.

Para inserir novos scripts, no Solution Explorer, clicamos em **Queries** com o botão direito e escolhemos **New Query**.
Será criado um arquivo físico com extensão .sql na solução. Renomeie o arquivo de acordo com a necessidade, no próprio Solution Explorer.

Para facilitar o trabalho ao escrever queries, podemos ocultar as abas laterais, desataxando essas abas ao clicar no ícone de pin.

- Bloco de comentários: 
```sql
/*
    Este é um comentário
    De várias
    linhas
*/
```

- Comentário de linha única:
```sql
-- Comentário de linha única
```

Ao trabalharmos, sempre salvar o arquivo, para não perder trabalho.
Na lateral esquerda do arquivo, ficarão algumas cores. Verde para trechos já salvos e amarelo para não salvos.

- Criando banco de dados:
```sql
CREATE DATABASE DepartamentoPessoal;
```

Para executar o comando, podemos utilizar a telca *F5* ou clicar em Executar.
Podemos também selecionar o trecho de código a ser executado e usar o atalho *ALT + X*.

Após criado o banco, para colocarmos em uso, podemos utilizar o SSMS e selecioná-lo no menu Availabe Databases *(CTRL + U)* ou via T-SQL:

- Selecionando banco de dados em uso:
```sql
USE DepartamentoPessoal;
```

- Excluindo banco de dados:
```sql
DROP DATABASE DepartamentoPessoal;
```
> O banco de dados não pode estar em uso para fazer a exclusão.

Para exibir ou ocultar o painel de resultados, usamos o atalho *CTRL + R*.

## Criando tabelas via T-SQL

Sintaxe para criação de tabelas:

```sql
CREATE TABLE tblNomeTabela(NomeCampo TipoDeDados [IDENTITY] [NULL/NOT NULL] [CHAVE/ÍNDICE])
```
> Os campos com colchetes são opcionais

- NomeCampo: Nome da coluna
- TipoDeDados: Qual o tipo de dado (data, inteiro, texto)
- Identity: indica se o campo será autonumerado pelo SQL. Padrão é de 1 em 1: Identity(1,1)
> O primeiro número é o primeiro número a ser inserido na primeira inserção, o segundo a quantidade a ser incrementada sobre o primeiro número.

- unique: não será possível inserir outro registro com essa coluna escrita com a mesma grafia.

```sql
CREATE TABLE tblMarcas 
(
	IdMarca			int			identity		primary key,
	NomeMarca		nchar(30)	not null		unique,
	
);
```

Proc para exibir informações da tabela: 
```sql
EXEC sp_help tblMarcas;
```

Sendo mais descritivo na criação da tabela e suas constraints:

```sql
CREATE TABLE tblModelos
(
	IdModelo		INT			IDENTITY
	CONSTRAINT		PK_tblModelos_IdModelo
	PRIMARY KEY (IdModelo),

	IdMarca			INT			NOT NULL
	CONSTRAINT		FK_tblModelos_tblMarcas_IdMarca
	FOREIGN KEY (IdMarca)
	REFERENCES tblMarcas(IdMarca),

	NomeModelo		NCHAR(30)	NOT NULL
	CONSTRAINT		UQ_tblModelos_NomeModelo
	Unique (NomeModelo)
);
```
>Preferir a criação acima, explicitando os nomes das constraints, pois fica mais fácil identificar os índices, caso necessário.
>Fazendo da primeira forma, será gerado um número aleatório como nome para a contraint.

- Chave Primária *(Primary Key - PK)*:
Aquele que irá se relacionar ou ser pesquisado por outras tabelas. Esse campo identifica a linha de registro e deverá ser um campo exclusivo e não nulo.

- Chave Estrangeira *(Foreign Key - FK)*:
É o campo que irá se relacionar com um campo PK.

## Diagramas

Podemos criar diagramas para visualizar as tabelas e seus relacionamentos.
Para isso, no SSMS, expanda o banco de dados desejado, selecione a opção *Database Diagrams* e depois *New Diagram*.
Selecione as tabelas que deseja inserir e clique em *Add*.

Será gerado um diagrama com relacionamentos devidamente definidos.
Se salvar o arquivo, ele ficará salvo na pasta *Database Diagrams*.

Para adicionar no diagrama alguma tabela criada posteriormente, basta clicar em um espaço vazio e escolher a opçao *Add table*.
Se a tabela não estiver aparecendo, dê um refresh.

## Constraints e estrutura de tabelas

Constraints são regras que garantem a consistência nos dados.
Toda Primary Key e Unique já são constraints. Veremos agora mais duas: *DEFAULT* e *CHECK*.

```sql
USE Concessionaria;

CREATE TABLE tblEstoque
(
	IdEstoque		INT		IDENTITY
	CONSTRAINT		PK_tblEstoque_IdEstoque
	Primary Key (IdEstoque),

	IdModelo		INT		NOT NULL
	CONSTRAINT		FK_tblEstoque_tblModelos
	FOREIGN KEY (IdModelo)
	REFERENCES tblModelos(IdModelo),

	DataEntrada		DATE	DEFAULT GETDATE(),

	PrecoEstoque	MONEY	NOT NULL
	CONSTRAINT		CK_tblEstoque_PrecoEstoque
	CHECK			(PrecoEstoque >= 10000 AND PrecoEstoque <= 1000000)
);
```

As constraints são verificadas toda vez que um registro é inserido ou alterado no banco de dados.
Na constraint *DEFAULT* acima, se não for informada a *DataEntrada*, será executada por padrão a função nativa *GETDATE()*, para inserir a data atual.  
Na constraint de *CHECK*, será sempre validado se o valor está dentro da faixa definida, evitando valores menores ou maiores que o estipulado.  

Sintaxe para alterar tabela, adicionando coluna, se necessário:
```sql
ALTER TABLE tblEstoque
ADD Placa NCHAR(8) NOT NULL;
```

Sintaxe para alterar coluna de tabela:
```sql
ALTER TABLE tblEstoque
ALTER COLUMN Placa  NCHAR(7);
```

Sintaxe para excluir coluna de tabela:
```sql
ALTER TABLE tblEstoque
DROP COLUMN Placa;
```

>Não podemos alterar ou excluir colunas que possuam constraints ou seja uma coluna PK ou FK, exceto se excluirmos estas constraints primeiro.

Sintaxe para excluir constraint:
```sql
ALTER TABLE tblEstoque
DROP CONSTRAINT CK_tblEstoque_PrecoEstoque;
```

## Índices

- Como o SQL Server armazena e acessa dados?

Através de índices. índices podem trazer grandes melhorias para o desempenho do banco de dados.
Os registros são armazenados em páginas de dados como se fossem um livro ou revista. Essas páginas possuem cabeçalhos e um tamanho fixo de 8kb.  
A cada 8 páginas, é formado um conjunto denominado extensão.

Os índices servem para localizar os registros como se tivessem um sumário, então é possível acessar os registros desejados sem a necessidade de varrer todos os registros.  
É como se estivesse lendo um livro que possui um sumário, e esse sumário informa em qual página está o assunto desejado.  
Esse sumário permite saltar diretamente para a página que contém a informação desejada.  

**Quando existem índices, as buscas são mais rápidas.**

- Índices são bem vindos em colunas de alta seletividade
- Toda coluna PK ou Unique possuí índice.

Caso queira fazer a consulta em uma coluna que não seja PK ou Unique, pode criar índices para essas colunas também.  

- Desvantagens:
	- Ocupam muito espaço em disco, pois geram uma espécie de backup dos dados
	- Torna extremamente lenta a operação de inserção, atualização ou exclusão em tabelas

>Evite criar índices em colunas que sofram muitas atualizações ou em tabelas que recebem altas cargas de dados.

Existem dois tipos de índices, os *clustered* e os *non clustered*.

- Clustered:  
	Os dados ficam fisicamente ordenados no disco juntamente com a estrutura da tabela (como se fosse uma cópia da tabela, somente com a coluna que possui o índice, de forma ordenada).  
	Toda chave primária é um índice do tipo clusterizado.
- Non Clustered:   
	Os dados não ficam fisicamente ordenados no disco e sua estrutura é separada dos dados da tabela.  
	Não há aquela perda de espaço em disco, pois a ordenação ocorre em tempo de execução, quando for feita uma consulta.  

>Em colunas que são muito utilizadas como condições para pesquisa *(WHERE)*, podemos criar índices, para que a consulta seja mais rápida, observando os pontos já destacados.  

```sql
USE Concessionaria;

--VISUALIZAR ÍNDICES
EXEC sp_help tblEstoque;
EXEC sp_helpindex tblEstoque;

--CRIANDO ÍNDICES
CREATE NONCLUSTERED INDEX IX_tblEstoque_DataEntrada
ON tblEstoque(DataEntrada DESC);

--DELETANDO
drop index IX_tblEstoque_tblEstoque ON tblEstoque;
```
>Para índices que não são chave primária, é recomendável utilizar o prefixo *'IX_'*.

## Anexando Bancos

O processo de anexar bancos serve para importar bancos de dados prontos para o seu servidor.
Para isso, podemos usar a rotina abaixo:

```sql
EXEC SP_ATTACH_DB 
	@DBNAME ='SisDep',
	@FILENAME1 = 'D:\Material\SisDep.mdf',
	@FILENAME2 = 'D:\Material\SisDep_log.ldf';
```


## Manipulando Dados (Inserindo Registros)

Para inserir registros em uma tabela, utilizamos o comando *INSERT*.
Existem duas formas de fazer um insert, a **posicional** e a **declarativa**.

- Posicional
	- Seguirmos a risca a ordem das colunas da tabela e inserimos dados em todas elas
- Declarativo
	- Explicitamos quais colunas irão receber os dados e sua ordem, depois informamos os dados.

Exemplos de INSERT posicional:

```sql
INSERT INTO tblMarcas
VALUES ('FIAT');
```

```sql
-- INSERÇÃO DE VÁRIAS LINHAS
INSERT INTO tblMarcas
VALUES ('FORD'), ('CHEVROLET'), ('VOLKSWAGEN'), ('HONDA');
```
>Não é necessário inserir o valor de campos identity.

Para verificar as linhas inseridas, podemos fazer um *SELECT* simples:
```sql
SELECT * FROM tblMarcas
```

> Caso ocorra um erro ao inserir um dado em uma tabela com identity, a numeração que seria gerada pelo identity é perdida e o próximo registro será inserido pulando esse número.  
> A ordenação é feita a príncipio pela coluna PK. Caso exista uma coluna com unique, a consulta será ordenada por unique.

Exemplo de INSERT declarativo:

```sql
-- INSERT DECLARATIVO
INSERT INTO tblModelos
	(IdMarca, NomeModelo)
VALUES
	(5, 'Onix'), (1, 'Uno'), (4, 'Ka');
```

>Em aplicações, sempre usar o modo declarativo. Isso evita erros causados por posicionamento de colunas ou novas colunas, permitindo também inserir dados apena nos campos desejados, de que os demais aceitem nulos.  

>No SSMS, é possível arrastar a pasta *Columns* de uma tabela para a área de script, para que seja adicionado no script o nome de todas as tabelas.  


## Manipulando Dados (Atualizando Registros)

Sintaxe para atualizar todos os registros:
```sql
UPDATE NomeTabela SET Campo = NovoValor;
```

Para vários campos:
```sql
UPDATE NomeTabela SET Campo = NovoValor, Campo2 = NovoValor, CampoN = NovoValor;
```

Atualizando valor do campo, utilizando seu valor atual:
```sql
UPDATE Funcionario SET Salario = Salario + 100;
```
>Note que todo *UPDATE* sem *WHERE* será realizado para todos os registros da tabela.

Exemplo utilizando operador composto:
```sql
-- REAJUSTE DE 10% PARA TODOS OS FUNCIONÁRIOS
UPDATE Funcionario SET Salario *= 1.10;
```

Exemplo atualizando mais de um campo e para um registro específico:
```sql
UPDATE Funcionario 
SET Salario *= 1.05, idCargo = 2 
WHERE idMatricula = 1000;
```

## Manipulando Dados (Excluindo Registros)

Sintaxe para exclusão de registros:

Exclui todos os registros da tabela:
```sql
DELETE FROM NomeTabela;
```

Exclui os registros da tabela, de acordo com o filtro informado:
```sql
DELETE FROM NomeTabela WHERE Condição;
```

Exemplos:
```sql
DELETE FROM Carteiras
WHERE Cpf = 25155054733;
```

```sql
DELETE FROM Carteiras WHERE Uf = 'GO';
```

>O SQL gerará erro de constraint se, ao tentar apagar um registro, houver algum registro em outra tabela que use esse registro como Foreign Key. Nesse caso, uma primeira solução seria apagar os registros que referenciam como chave estrangeira esse registro que está tentando ser excluído.  

## Trabalhando com Transações

Trata-se de um processo/procedimento que tem início e fim. É uma forma segura de realizar operações.  
Existem dois tipos de transação: a *implícita* e a *explícita*.

- Implícita: É do próprio SGBD. Qualquer erro que ocorra, a aplicação aborta a operação, cancelando todas as etapas. 
- Explícita: Quando o usuário define o início e término da operação.

Algumas operações requerem atenção, como *UPDATE* E *DELETE*. Não é possível desfazer uma dessas operações, exceto quando executadas dentro de uma transação.  

**DTL - Data Transaction Language**
São comando utilizados para definir o início e o fim de uma transação.

- *BEGIN TRANSACTION* ou *BEGIN TRAN*: Define o início
- *COMMIT TRANSACTION* ou *COMMIT TRAN* ou *COMMIT*: Define o fim e confirma todas as operações
- *ROLLBACK TRANSACTION* ou *ROLLBACK TRAN* ou *ROLLBACK*: Define o fim e cancela todas as operações

Exemplos:
```sql
-- INICIAR TRANSAÇÃO
BEGIN TRANSACTION;

	UPDATE Apolices
	SET valorApolice += 1500;

-- CANCELA A OPERAÇÃO
ROLLBACK TRANSACTION;
```

```sql
BEGIN TRAN;

	UPDATE Apolices
	SET valorApolice += 1500
	WHERE nContrato = 1000;

-- CONFIRMA A OPERAÇÃO
COMMIT TRAN
```

Para verificar se existe alguma transação ativa:
```sql
SELECT @@TRANCOUNT;
```

### Instrução OUTPUT
Toda vez que realizamos operações de *INSERT* - *UPDATE* - *DELETE*, o SQL Server utiliza-se de tabelas de sistema para (não visíveis no painel Object Explorer) justamente para conseguir confirmar ou cancelar essa operações de forma implícita.  
São essas tabelas:  

| Operação | Tabela de Sistema 							   |
| :----:   | :----:            							   |
| *INSERT* | Inserted          							   |
| *DELETE* | Deleted           							   |
| *UPDATE* | Inserted (novo valor), Deleted (valor antigo) |

```sql
-- UTILIZANDO OUTPUT COM AS TABELAS DE SISTEMA INSERTED E DELETED, PARA VERIFICAR VALORES ANTES E APÓS ATUALIZAÇÃO
USE SisDep;

BEGIN TRAN
	UPDATE Funcionario
	SET Salario *= 1.1
	OUTPUT
		deleted.idMatricula,
		deleted.NomeFuncionario,
		deleted.Salario AS SalarioAntigo,
		inserted.Salario AS NovoSalario
	WHERE Salario <= 3000;
		
COMMIT;
```

## Consulta a Dados

**DQL - Data Query Language**

Sem sombra de dúvidas, é o grupo de comandos mais importante para extrair dados a fim de gerar informações.  
Sua principal cláusula é o *SELECT*. Abaixo a sintaxe do SQL ANSI:

```sql
SELECT [DISTINCT] [TOP] COLUNAS FROM TABELA1 
[JOIN TABELA2] [ON] [TABELA1.COLUNA = TABELA2.COLUNA]
[WHERE]
[GROUP BY]
[HAVING]
[ORDER BY]
```
> As cláusulas entre colchetes são opcionais.

Exemplos:

```sql
-- TODAS AS COLUNAS DE UMA TABELA
SELECT * FROM Funcionario;

-- ALGUMAS COLUNAS DA TABELA
SELECT idMatricula, NomeFuncionario, Admissao FROM Funcionario;
```

### Ordenando Dados

Por padrão, os dados são ordenados na ordem de entrada caso não haja índices clusterizados como PK ou Unique.  
Caso existam índices PK ou unique, os dados, por padrão, serão ordenados de forma crescente.

Através da cláusula *ORDER BY* poderemos definir a ordenação:
- ASC: ordenação crescente (Padrão, sua escrita é opcional)
- DESC: ordenação decrescente

Exemplos:

```sql
-- ORDENANDO
SELECT idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario
FROM Funcionario
ORDER BY NomeFuncionario ASC;

SELECT idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario 
FROM Funcionario
ORDER BY Salario DESC;

-- ORDENAR POR MAIS DE UMA COLUNA
SELECT idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario 
FROM Funcionario
ORDER BY idDepartamento ASC, Salario DESC;

-- ORDENAR PELA POSIÇÃO DA COLUNA
SELECT idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario 
FROM Funcionario
ORDER BY 1 ASC, 5 DESC; --idDepartamento e Salario
```


### RANK (TOP)

A cláusula *TOP* pode ser usada para exibir uma determinada quantidade de linhas como amostragem ou mostrar um ranking de maiores ou menores.  
É muito útil combinada com o *ORDER*.

Exemplos:  

```sql
-- RANK TOP 20 primeiras linhas
SELECT TOP 20 idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario 
FROM Funcionario;

-- RANK TOP 20 porcento dos dados
SELECT TOP 20 PERCENT idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario 
FROM Funcionario;

-- 10 MAIORES SALÁRIOS
SELECT TOP 10 idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario 
FROM Funcionario
ORDER BY Salario DESC; --ÚLTIMAS LINHAS DUPLICADAS

-- 9 MAIORES SALÁRIOS, MAS COM EMPATES
SELECT TOP 9 WITH TIES idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario 
FROM Funcionario
ORDER BY Salario DESC; --ÚLTIMAS LINHAS DUPLICADAS
```

## Filtro com operadores

Operadores Relacionais:

| Operador | Descrição      |
| :----:   | :----:         |
| >        | Maior          |
| <        | Menor          |
| >= ou !< | Maior ou Igual |
| <= ou !> | Menor ou Igual |
| =        | Igual          |
| <> ou != | Diferente      |

Operadores Aritméticos:

| Operador | Descrição        |
| :----:   | :----:           |
| *        | Multiplicação    |
| /        | Divisão   		  |
| +        | Soma 			  |
| -        | Subtração        |
| %        | Resto da Divisão |

Operadores lógicos:

| Operador | Descrição        |
| :----:   | :----:           |
| NOT      | Negado ou não    |
| AND      | E      		  |
| OR       | Ou	 			  |

Operadores Compostos (Para updates):

| Operador | Descrição        				   |
| :----:   | :----:           				   |
| +=       | Soma ao próprio valor    		   |
| -=       | Subtrai ao próprio valor   	   |
| /=       | Divide ao próprio valor 		   |
| *=       | Multiplica ao próprio valor       |
| %=       | Resto da Divisão do próprio valor |

Exemplos:

```sql
--Operadores relacionais
SELECT * FROM Apolices
WHERE valorApolice >= 50000;

--Operadores lógicos
SELECT * FROM Apolices
WHERE idSeguradora = 1 OR idSeguradora = 3;

SELECT * FROM Apolices
WHERE idSeguradora = 1 AND valorApolice >= 50000;

SELECT * FROM Apolices
WHERE NOT idCidade = 5; --IDCIDADE NÃO SEJA 5
```
> Esse último comando poderia ser feito da forma abaixo. Evitar esse comando sempre que possível, pois ele demanda muito processamento, pois primeiro verifica todos os registros que atendem a condição e depois devolve os outros registros.  

```sql
SELECT * FROM Apolices
WHERE idCidade <> 5; --IDCIDADE NÃO SEJA 5
```

Mais exemplos:

```sql
SELECT * FROM Apolices
WHERE valorApolice >= 50000
ORDER BY valorApolice DESC; 

--Operadores aritméticos
SELECT nContrato, valorApolice, valorApolice * 1.1 AS Reajuste FROM Apolices;

--Operadores Compostos
BEGIN TRAN

	UPDATE Apolices
	SET valorApolice *= 1.1

COMMIT

```

## Filtros BETWEEN, LIKE e IN

- *BETWEEN*
Útil para realizar pesquisas por intervalos

Exemplos:

```sql
SELECT
	idMatricula, NomeFuncionario, Admissao, Salario
FROM Funcionario
WHERE Salario BETWEEN 1500 AND 3000;

SELECT
	idMatricula, NomeFuncionario, Admissao, Salario
FROM Funcionario
WHERE Admissao BETWEEN '2005-01-01' AND '2005-12-31';

SELECT
	idMatricula, NomeFuncionario, Admissao, Salario
FROM Funcionario
WHERE Admissao NOT BETWEEN '2005-01-01' AND '2005-12-31';
```

- *IN*
Útil para inserir vários filtros de uma mesma coluna em uma só cláusula

Exemplos:
```sql
SELECT
	idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario
FROM Funcionario
WHERE idDepartamento IN (1, 3, 5, 6, 10);

SELECT
	idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario
FROM Funcionario
WHERE idDepartamento NOT IN (1, 7);
```
> Note que poderíamos atingir o mesmo resultando colocando o operador *OR*, porém a sintaxe ficaria mais verbosa.

- *LIKE*
Útil para realizar pesquisas por parte de contéudo.
>Atenção, esse é um comando muito custoso para o banco de dados, evitar ao máximo sua utilização.

Para usar o *LIKE*, normalmente usamos caracteres coringa. No SQL Server, temos esses caracteres coringa:


| Caractere | Descrição      					  |
| :----:    | :----:         					  |
| %         | Um ou mais caracteres desconhecidos |
| _         | Apenas um caractere desconhecido    |

Exemplos:
```sql
SELECT
	idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario
FROM Funcionario
WHERE NomeFuncionario LIKE 'A%'
ORDER BY NomeFuncionario;

SELECT
	idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario
FROM Funcionario
WHERE NomeFuncionario LIKE 'A_A%'
ORDER BY NomeFuncionario;

SELECT
	idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario
FROM Funcionario
WHERE NomeFuncionario LIKE '%OLIVEIRA'
ORDER BY NomeFuncionario;

SELECT
	idDepartamento, idMatricula, NomeFuncionario, Admissao, Salario
FROM Funcionario
WHERE NomeFuncionario LIKE '%SILVA%'
ORDER BY NomeFuncionario;
```

## Unindo dados

O operador *UNION* permite unir os resultados de dois ou mais comandos *SELECT*.  
O retorno de seu resultado é exclusivo, ou seja, sem  repetições de linhas considerando todas as colunas envolvidas na seleção.  
Temos também o operador *UNION ALL*, que utiliza as mesmas regras do *UNION*, mas retorna os registros duplicados.

Regras de utilização:
- Todos os comandos *SELECT* deverão conter o mesmo número de colunas
- As colunas que serão unidas deverão ter o mesmo tipo de dados

Sintaxe:
```sql
SELECT COLUNAS FROM TABELA1 [WHERE]
UNION
SELECT COLUNAS FROM TABELA2 [WHERE]
```

Exemplos:

```sql
--UNION SIMPLES
SELECT * FROM Clientes2015
UNION
SELECT * FROM Clientes2016;

-- SE TIVESSEM COLUNAS COM NOMES DIFERENTES, O IDEAL SERIA FAZER O ORDER BY PELA POSIÇÃO DA COLUNA
SELECT * FROM Clientes2015
UNION
SELECT * FROM Clientes2016
ORDER BY 2; --COLUNA DO NOME

-- COLUNA VIRTUAL PARA IDENTIFICAR OS CLIENTES DE CADA TABELA
SELECT 'CLIENTE 2015' AS Ano, * FROM Clientes2015
UNION
SELECT 'CLIENTE 2016', * FROM Clientes2016
ORDER BY Cliente; --COLUNA DO NOME

--UNION ALL TRÁS OS REGISTROS DUPLICADOS
SELECT * FROM Clientes2015
UNION ALL
SELECT * FROM Clientes2016
ORDER BY Cliente; 

--PODEMOS TER CONDIÇÕES
SELECT 'CLIENTE 2015' AS Ano, * FROM Clientes2015
WHERE Cidade = 'SÃO PAULO'
UNION
SELECT 'CLIENTE 2016', * FROM Clientes2016
WHERE Cidade = 'RIO DE JANEIRO'
ORDER BY Cliente; --COLUNA DO NOME
```

## Comparando dados

**Operador *INTERSECT***

Utilizado para verificar a existência de dados comuns entre duas tabelas.

Regras de utilização:
- Todos os comandos *SELECT* deverão conter o mesmo número de colunas
- As colunas que serão unidas deverão ter o mesmo tipo de dados
- Esse comando só compara uma coluna por vez

Sintaxe:
```sql
SELECT COLUNA FROM TABELA1 [WHERE]
INTERSECT
SELECT COLUNA FROM TABELA2 [WHERE]
```

Exemplos:
```sql
--CLIENTES QUE CONSTAM NAS DUAS TABELAS, NA COLUNA 'CLIENTE'
SELECT Cliente FROM Clientes2015
INTERSECT
SELECT Cliente FROM Clientes2016; 
```
>O própio sql irá criar um índice para ordenar os resultados

**Operador *EXCEPT***
Utilizado para verificar a não existência de dados comuns entre duas tabelas.
Possui as mesmas regras de utilização do *INTERSECT*.

Exemplos:
```sql
--CLIENTES QUE RECINDIRAM O CONTRATO (CONSTAM EM 2015 MAS NÃO EM 2016)
SELECT Cliente FROM Clientes2015
EXCEPT -- A ORDEM ALTERA O RESULTADO!
SELECT Cliente FROM Clientes2016; 


--NOVOS CLIENTES (CONSTAM EM 2016 MAS NÃO EM 2015)
SELECT Cliente FROM Clientes2016
EXCEPT
SELECT Cliente FROM Clientes2015; 
```

## Trabalhando com valores nulos

**Função *ISNULL***

Utilizada para tratar valores nulos em um resultado.  

Sintaxe:

```sql
ISNULL(CAMPO, VALOR)
```
> VALOR será o texto ou número que irá substituir o nulo.  

Exemplos:

```sql
SELECT Pedido, ISNULL(Vendedor, 'Sem Informação') as Vendedor, Produto, Total
FROM Dados 
ORDER BY Pedido;

SELECT Pedido, ISNULL(Vendedor, '') as Vendedor, Produto, Total
FROM Dados 
ORDER BY Pedido;
```

**Operador *IS NULL* / *IS NOT NULL***

Utilizado para filtar dados nulos ou não nulos

Exemplos:

```sql
--FILTRAR NULOS 
SELECT Pedido, Vendedor, Produto, Total
FROM Dados 
WHERE Vendedor IS NULL
ORDER BY Pedido;

--FILTAR NÃO NULOS
SELECT Pedido, Vendedor, Produto, Total
FROM Dados 
WHERE Vendedor IS NOT NULL
ORDER BY Pedido;
```

**Função Coalesce**

Utilizado para verificar uma lista de um ou mais campos e retornar o primeiro valor não nulo.  

Sintaxe:

```sql
COALESCE(CAMPO1, CAMPO2, CAMPON, VALOR)
```
> VALOR será o texto ou número que irá ser exibido caso todos os valores dos CAMPOS sejam nulos.  

Exemplos:

```sql
-- COALESCE PARA EXIBIR RESULTADO
SELECT 
	Produto, 
	COALESCE(Cotacao1, Cotacao2, Cotacao3, 0) AS Cotacao
FROM tblCotacao;

-- COALESCE PARA FILTRAR
SELECT 
	Produto
FROM tblCotacao
WHERE
	COALESCE(Cotacao1, Cotacao2, Cotacao3, 0) = 0;
```

## Junções Internas

Entende-se por junção quando necessitamos extrair dados de duas ou mais tabelas, pois estamos trabalhando em um sistema de banco de dados baseado no modelo relacional.  

Uma junção interna é quando dados de duas ou mais tabelas possuem relação, não necessariamente por PK-FK (mas altamente recomendável), ou há ocorrência de dados de uma determinada coluna nas duas tabelas.  
Para esse tipo de junção utilizamos o *INNER JOIN*.

Sintaxe:

```sql
SELECT COLUNAS FROM TABELA1 [INNER] JOIN TABELA2
ON TABELA1.COLUNA = TABELA2.COLUNA
```

Exemplos:

```sql
-- DUAS TABELAS
SELECT 
	NomeFuncionario, Admissao, Salario, Uf, Cidade
FROM 
	Funcionario INNER JOIN Localidade ON Funcionario.idLocalidade = Localidade.idLocalidade;

-- TRÊS TABELAS
SELECT 
	NomeFuncionario, NomeDepartamento, Admissao, Salario, Uf, Cidade
FROM 
	Funcionario 
INNER JOIN 
	Localidade ON Funcionario.idLocalidade = Localidade.idLocalidade
INNER JOIN 
	Depto ON Depto.idDepartamento = Funcionario.idDepartamento;

-- NOMES QUALIFICADOS
SELECT 
	Funcionario.NomeFuncionario, Depto.NomeDepartamento, Funcionario.Admissao, Funcionario.Salario, Localidade.Uf, Localidade.Cidade
FROM 
	Funcionario 
INNER JOIN 
	Localidade ON Funcionario.idLocalidade = Localidade.idLocalidade
INNER JOIN 
	Depto ON Depto.idDepartamento = Funcionario.idDepartamento
ORDER BY Funcionario.NomeFuncionario;

-- ALIAS
SELECT 
	F.NomeFuncionario, D.NomeDepartamento, F.Admissao, F.Salario, L.Uf, L.Cidade
FROM 
	Funcionario AS F
INNER JOIN 
	Localidade AS L ON F.idLocalidade = L.idLocalidade
INNER JOIN 
	Depto AS D ON D.idDepartamento = F.idDepartamento
ORDER BY F.NomeFuncionario;
```

> As junções internas somente retornam resultados cujos registros existam nas tabelas informadas pelas intersecções

## Junções externas

Para verificar dados que estão em uma base mas não em outra.

**LEFT OUTER JOIN**
Este operador de função (*LEFT*) irá forçar a retornar todos os dados da tabela a esquerda de *JOIN* e, da tabela a direita, somente os dados que existirem nas duas.  

Sintaxe:

```sql
SELECT COLUNAS FROM TABELA1 LEFT [OUTER] JOIN TABELA2
ON TABELA1.COLUNA = TABELA2.COLUNA
```

Exemplos:

```sql
SELECT * FROM Funcionario --100 REGISTROS

SELECT * FROM Dependente  --27 REGISTROS

--27 REGISTROS (FUNCIONÁRIOS QUE TEM DEPENDENTES)
SELECT 
	F.NomeFuncionario,
	D.NomeDependente,
	D.NascimentoDependente
FROM 
	Funcionario AS F 
INNER JOIN					
	Dependente AS D ON F.idMatricula = D.idMatricula
ORDER BY
	F.NomeFuncionario;

--107 REGISTROS (PELO MENOS UM REGISTRO DE FUNCIONÁRIO(TENDO OU NÃO DEPENDENTES), CONTENDO UM REGISTRO PARA CADA DEPENDENTE)
SELECT 
	F.NomeFuncionario,
	D.NomeDependente,
	D.NascimentoDependente
FROM 
	Funcionario AS F 
LEFT JOIN					
	Dependente AS D ON F.idMatricula = D.idMatricula
ORDER BY
	F.NomeFuncionario;
```

**RIGHT OUTER JOIN**
Este operador de função (*RIGHT*) irá forçar a retornar todos os dados da tabela a direita de *JOIN* e, da tabela a esquerda, somente os dados que existirem nas duas.  

Sintaxe:

```sql
SELECT COLUNAS FROM TABELA1 RIGHT [OUTER] JOIN TABELA2
ON TABELA1.COLUNA = TABELA2.COLUNA
```

Exemplos:

```sql
SELECT * FROM Funcionario --100 REGISTROS

SELECT * FROM Cargo  --20 REGISTROS

-- 102 REGISTROS...
SELECT 
	C.NomeCargo,
	F.NomeFuncionario,
	F.Salario
FROM 
	Funcionario AS F 
RIGHT JOIN					
	Cargo AS C ON F.idCargo = C.idCargo
ORDER BY
	C.NomeCargo;

-- PARA SABER CARGOS EM QUE NÃO HÁ FUNCIONÁRIOS TRABALHANDO, BASTA FILTRAR POR UM DOS CAMPOS NULL
SELECT 
	C.NomeCargo,
	F.NomeFuncionario,
	F.Salario
FROM 
	Funcionario AS F 
RIGHT JOIN					
	Cargo AS C ON F.idCargo = C.idCargo
WHERE 
	F.NomeFuncionario IS NULL
ORDER BY
	C.NomeCargo;
```

## Junção total e por cruzamento cartesiano

**FULL JOIN**
Esse operador de junção (*FULL*) irá forçar a retornar todos os dados das duas tabelas, tanto à esquerda de *JOIN* quanto à direita.  

Sintaxe:

```sql
SELECT COLUNAS FROM TABELA1 FULL JOIN TABELA2
ON TABELA1.COLUNA = TABELA2.COLUNA
```

Exemplo:

```sql
SELECT
	F.NomeFuncionario, 
	F.Admissao,
	D.NomeDependente,
	D.NascimentoDependente
FROM 
	Funcionario AS F 
FULL JOIN 
	Dependente AS D ON F.idMatricula = D.idMatricula
ORDER BY 
	F.NomeFuncionario;
```

**CROSS JOIN**
Esse operador de junção irá aplicar a regra de produto cartesiano, ou seja, cada elemento de uma tabelaX será combinado com cada elemento de uma tabelaY.

![Alt text](Material/cartesiano.jpg?raw=true "Plano Cartesiano")

> Nesse tipo de junção não precisamos especificar a coluna de referência para a junção.


Sintaxe:

```sql
SELECT COLUNAS FROM TABELA1 CROSS JOIN TABELA2;
```

Exemplo:

```sql
SELECT * FROM Depto; --10 REGISTROS
SELECT * FROM Projeto; -- 8 REGISTROS

-- 80 REGISTROS
SELECT 
	*
FROM 
	Depto AS D 
CROSS JOIN
	Projeto AS P;
```

## Manipulando dados com junções

**UPDATE COM JOINS**

Sintaxe:

```sql
UPDATE TABELA1 
SET CAMPO = VALOR
FROM TABELA1 [INNER LEFT RIGHT] JOIN TABELA2
ON TABELA1.COLUNA = TABELA2.COLUNA
```

Exemplos:

```sql
USE SisDep;

-- BÔNUS DE R$ 100,00 PARA TODOS OS FUNCIONÁRIOS QUE POSSUAM DEPENDENTES
BEGIN TRAN
UPDATE 
	Funcionario
SET Salario += 100
FROM 
	Funcionario
INNER JOIN 
	Dependente ON Funcionario.idMatricula = Dependente.idMatricula;

COMMIT

-- EXEMPLO UTILIZANDO OUTPUT PARA VER O ANTES E DEPOIS DO SALARIO
BEGIN TRAN
UPDATE 
	Funcionario
SET Salario += 100
OUTPUT
	deleted.idMatricula,
	deleted.NomeFuncionario,
	deleted.Salario AS SalarioAntigo,
	inserted.Salario AS NovoSalario
FROM 
	Funcionario
INNER JOIN 
	Dependente ON Funcionario.idMatricula = Dependente.idMatricula;

COMMIT

-- BÔNUS DE 10% PARA TODOS OS FUNCIONÁRIOS QUE NÃO POSSUAM DEPENDENTES
SELECT 
	F.NomeFuncionario, D.NomeDependente
FROM 
	Funcionario F
LEFT JOIN 
	Dependente D ON F.idMatricula = D.idMatricula
WHERE 
	D.NomeDependente IS NULL;

BEGIN TRAN
UPDATE 
	Funcionario
SET Salario *= 1.1
FROM 
	Funcionario
LEFT JOIN 
	Dependente ON Funcionario.idMatricula = Dependente.idMatricula
WHERE 
	Dependente.NomeDependente IS NULL;

COMMIT

-- DESLIGAMENTO DE FUNCIONÁRIOS DO SAC COM SALÁRIO MAIOR QUE R$1.500,00
BEGIN TRAN
DELETE 
	Funcionario
FROM 
	Funcionario AS F
INNER JOIN 
	Depto AS D ON D.idDepartamento = F.idDepartamento
WHERE
	F.Salario > 1500 AND D.NomeDepartamento = 'SAC';

COMMIT

-- DESLIGAMENTO DE FUNCIONÁRIOS COM SALÁRIO MAIOR QUE R$4.000,00 QUE NÃO POSSUEM DEPENDENTES
BEGIN TRAN

DELETE 
	Funcionario
FROM 
	Funcionario AS F
LEFT JOIN
	Dependente AS D ON F.idMatricula = D.idMatricula
WHERE
	D.NomeDependente IS NULL AND F.Salario > 4000;

COMMIT	
```

## Subconsultas

Subconsulta é a escrita de um comando *SELECT* dentro de outro.

>O SQL Server permite até 32 níveis de encadeamento de *SELECTS*, embora não seja recomendado pela significativa perda de performance.

Operadores para verificar a existência ou não de dados relacionados entre duas tabelas:

```sql
IN - NOT IN - EXISTS - NOT EXISTS
```
> No *EXISTS*, cruzamos os campos através do *WHERE*. No *IN*, informamos o campo anteriormente.
> A instrução *IN* é considerada uma evolução do *EXISTS*.

Exemplos:

```sql
--QUAIS CLIENTES NA BASE DE 2015 TAMBÉM EXISTEM NA BASE DE 2016
SELECT Cliente FROM Clientes2015 C15
WHERE EXISTS
	(SELECT Cliente FROM Clientes2016 C16 WHERE C15.Codigo = C16.Codigo); --SERVINDO DE FILTRO. 

--QUAIS CLIENTES NA BASE DE 2015 QUE NÃO EXISTEM NA BASE DE 2016
SELECT Cliente FROM Clientes2015 C15
WHERE NOT EXISTS
	(SELECT Cliente FROM Clientes2016 C16 WHERE C15.Codigo = C16.Codigo); --SERVINDO DE FILTRO. 


--NOME DOS FUNCIONÁRIOS QUE POSSUAM DEPENDENTES
SELECT
	Funcionario.NomeFuncionario
FROM 
	Funcionario
WHERE 
	Funcionario.idMatricula IN
	(
		SELECT Dependente.idMatricula FROM Dependente
	);

--NOME DOS FUNCIONÁRIOS QUE NÃO POSSUAM DEPENDENTES
SELECT
	Funcionario.NomeFuncionario
FROM 
	Funcionario
WHERE 
	Funcionario.idMatricula NOT IN
	(
		SELECT Dependente.idMatricula FROM Dependente
	);
```

Podemos também utilizar Subconsultas juntamente com operadores relacionais.

Exemplos:

```sql
SELECT
	Funcionario.idMatricula,
	Funcionario.NomeFuncionario,
	Funcionario.Salario
FROM Funcionario
WHERE 
	Funcionario.Salario > 
	(
		SELECT AVG(Funcionario.Salario) FROM Funcionario
	);
```

## Tabelas temporárias

Tratam-se de objetos criados no banco de sistema **TempDB**.  
Podem ser úteis para **armazenar dados na RAM** e obter uma melhora no desempenho da consulta, ou poderão ser simplesmente objetos de teste para comandos de exclusão/atualização.  

Uma tabela temporária pode ser:
- Local: **#NomeTabela** (somente a conexão ativa em que a tabela foi criada terá visibilidade deste objeto)
- Global: **##NomeTabela** (Todas as conexões ativas no servidor terão visibilidade deste objeto)

> Note que cada aba do SSMS é uma conexão ativa com o banco de dados, então a tabela local só ficará disponível na aba em que foi criada.

Para tabela local, os dados ficam disponíveis até que a conexão ativa seja finalizada (ou a aba fechada).
Para tabela global, os dados ficam disponíveis até que seja finalizada a conexão com o banco de dados (ou fechar o SSMS).

Existem duas formas de se criar tabelas temporárias:
- *CREATE TABLE*
- *SELECTINTO*

> Caso queira visualizar a tabela criada, é possível acessá-la em: Databases, System Databases, tempdb, Temporary Tables.

Exemplos:

```sql
-- TABELA TEMPORÁRIA LOCAL (CREATE TABLE)
CREATE TABLE #Clientes
(
	Codigo			INT,
	NomeCliente		VARCHAR(50),
	Cadastro		DATE
);

INSERT INTO #Clientes
VALUES
(1, 'Everton', GETDATE()),
(2, 'Joyce', GETDATE());

SELECT * FROM #Clientes;
-------------------------------------------------------
USE SysConVendas;

SELECT * FROM Dados;

SELECT * 
INTO #Pesquisa1
FROM Dados;

SELECT * FROM #Pesquisa1;

--FILTROS
SELECT * FROM #Pesquisa1 WHERE Mes = 'AGOSTO';

--ATUALIZAÇÕES
UPDATE #Pesquisa1 SET Vendedor = 'Everton' WHERE Pedido = 21794;

--PASSANDO DADOS PARA UMA TABELA GLOBAL
SELECT * 
INTO ##Pesquisa2
FROM #Pesquisa1;
```

## Funções de agregação

Agregação em banco de dados pode se compreendido como consolidar ou totalizar uma determinada informação.  
Existem várias funções de agregação. As principais são:  

| Função  | Descrição 		 |
| :----:  | :----:           |
| *SUM*   | Soma          	 |
| *AVG*   | Média aritmética |
| *MAX*   | Maior valor 	 |
| *MIN*   | Menor valor 	 |
| *COUNT* | Contagem    	 |

Retornando uma agregação de domínio geral:
```sql
SELECT SUM(COLUNA) [AS ROTULO] FROM TABELA
```

Domínio pode ser uma tabela ou consulta.  
Domínio geral pode ser entendido como todas as colunas desse domínio.  
> Campos com valores nulos não são considerados.


Exemplos:

```sql
-- RETORNAR TOTAL GERAL DE SALÁRIOS PAGOS
SELECT SUM(SALARIO) AS TotalSalarios FROM Funcionario;

-- RETORNAR MÉDIA DE SALÁRIOS PAGOS
SELECT AVG(SALARIO) AS MediaSalarios FROM Funcionario;

-- MAIS DE UMA AGREGAÇÃO NO MESMO COMANDO
SELECT 
	MAX(SALARIO) AS MaiorSalario,
	MIN(SALARIO) AS MenorSalario,
	COUNT(SALARIO) AS NumeroFuncionarios
FROM Funcionario;


-- COUNT
SELECT * FROM Dados; -- 450 REGISTROS

SELECT COUNT(Vendedor) AS ContagemColuna FROM Dados; --447 REGISTROS, COUNT NÃO CONTA NULOS
SELECT COUNT(Pedido) AS ContagemColuna FROM Dados;   --450 REGISTROS
SELECT COUNT(*) AS ContagemLinha FROM Dados;		 --450 REGISTROS. PREFERIR ESSE MÉTODO PARA CONTAS LINHAS DA TABELA
```

## Agrupando dados

**Agragação de tabela única**

Uma agragação poderá consolidar ou totalizar por um determinado agrupamento de uma ou mais colunas.

Sintaxe:

```sql
SELECT COLUNA_NAO_AGREGADA, FUNCAO_AGREGACAO(COLUNA_AGREGADA)
FROM TABELA
GROUP BY COLUNA_NAO_AGREGADA;
```

**Agragação com junções**

Uma agragação poderá consolidar ou totalizar por um determinado agrupamento de uma ou mais colunas através de junção com *JOIN* de duas ou mais tabelas.  

Sintaxe:

```sql
SELECT COLUNA_NAO_AGREGADA, FUNCAO_AGREGACAO(COLUNA_AGREGADA)
FROM TABELA1 [OPERADOR_JUNCAO] TABELA2
ON TABELA1.COLUNA = TABELA2.COLUNA
GROUP BY COLUNA_NAO_AGREGADA;
```

Exemplos:

```sql
SELECT * FROM Dados;

SELECT SUM(Total) as FaturamentoTotal FROM Dados; -- SOMANDO TODAS AS COLUNAS (DOMÍNIO GERAL)

-- AS COLUNAS NÃO AGREGADAS DEVEM RECEBER UM AGRUPAMENTO PARA CONTAR NO RESULTADO
SELECT 
	Cidade, SUM(Total) as FaturamentoTotalPorCidade
FROM 
	Dados
GROUP BY
	Cidade;

-- AGRUPAMENTO POR MAIS DE UM CAMPO, NOTE QUE O VALOR DO FATURAMENTO É AJUSTADO
SELECT 
	Produto, Cidade, SUM(Total) as FaturamentoTotalPorProdutoECidade, COUNT(*) AS Ocorrencias
FROM 
	Dados
GROUP BY
	Cidade,	Produto;

-- FILTROS EM AGRUPAMENTO HAVING
SELECT 
	Cidade, SUM(Total) as FaturamentoTotalPorCidade
FROM 
	Dados
GROUP BY
	Cidade
HAVING
	SUM(Total) > 20000
ORDER BY 2;

-- SUBTOTAIS
SELECT 
	Cidade, SUM(Total) as FaturamentoTotalPorCidade
FROM 
	Dados
GROUP BY
	Cidade
WITH ROLLUP; --ADICIONA SUBTOTAL PELA PRIMEIRA COLUNA 

-- SUBTOTAIS POR CIDADE
SELECT 
	Cidade, Produto, SUM(Total) as FaturamentoTotalPorCidade
FROM 
	Dados
GROUP BY
	Cidade, Produto
WITH ROLLUP; 

-- SUBTOTAIS POR CIDADE E PRODUTO
SELECT 
	Cidade, Produto, SUM(Total) as FaturamentoTotalPorCidade
FROM 
	Dados
GROUP BY
	Cidade, Produto
WITH CUBE; --ADICIONA SUBTOTAL POR CADA COLUNA NÃO AGREGADA

-- AGRUPAMENTO COM JUNÇÕES
SELECT 
	NomeFuncionario, COUNT(*) AS NumeroDependentes
FROM 
	Funcionario INNER JOIN Dependente ON Funcionario.idMatricula = Dependente.idMatricula
GROUP BY
	NomeFuncionario;
```

## Funções built-in (strings)

Funções *BUILT-IN* são aquelas que são do próprio SGBD, no caso o MS-SQL Server. As mesmas são organizadas em categorias.  
Abaixo, uma relação das funções para manipular texto (string):

| Função  	  | Descrição 		 							    		 |
| :----:  	  | :----:           							    		 |
| *LEN*   	  | Retorna o comprimento de um texto          	 			 |
| *LEFT*      | Retorna os primeiros caracteres da esquerda  			 |
| *RIGHT*     | Retorna os últimos caracteres da direita 	 			 |
| *REPLACE*   | Substituir um texto por outro 				 			 |
| *CHARINDEX* | Localizar a posição de um texto dentro do outro 		 |
| *SUBSTRING* | Retornar parte de um texto 			      	    		 |
| *RTRIM* 	  | Remover espaços à direita de um texto			      	 |
| *LTRIM* 	  | Remover espaços à esquerda de um texto 			      	 |
| *UPPER* 	  | Converter texto em letras maiúsculas 			      	 |
| *LOWER* 	  | Converter texto em letras minúsculas 			      	 |
| *REPLICATE* | Repetir um caractere uma determinada quantidade de vezes |
| *REVERSE*   | Inverter um texto 			      	    				 |
| *CONCAT*    | Concatenar conteúdos        			      	    	 |

Exemplos:

```sql
SELECT 
	NomeFuncionario,
	LEN(NomeFuncionario) AS NumeroDeCaracteresDoNome,
	idMatricula, 
	LEFT(idMatricula, 2) AS DoisPrimeirosCaracteresMatricula,
	RIGHT(idMatricula, 2) AS DoisUltimosCaracteresMatricula,
	REPLACE(idMatricula, '10', '20') AS Substituindo,
	CHARINDEX('SILVA', NomeFuncionario, 1) AS LocalizandoTexto,
	SUBSTRING(NomeFuncionario, 1, CHARINDEX(' ', NomeFuncionario, 1)-1) AS SomentePrimeiroNome
FROM Funcionario;

SELECT RTRIM('EVERTON     ');
SELECT LTRIM('     EVERTON');
SELECT LTRIM(RTRIM('     EVERTON     '));

SELECT
	UPPER(NomeFuncionario) AS Maiusculas,
	LOWER(NomeFuncionario) AS Minusculas
FROM Funcionario;

SELECT REPLICATE('*', 10);
SELECT REVERSE('EVERTON');

SELECT 
	idMatricula, 
	NomeFuncionario,
	idDepartamento,
	idCargo,
	CONCAT(idMatricula, idDepartamento, idCargo) AS Concatenar
FROM Funcionario;
```

## Funções built-in (datetime)

Abaixo, uma relação das funções para manipular data e hora:

| Função  	  	  | Descrição 		 						 |
| :----:  	  	  | :----:           						 |
| *GETDATE*   	  | Retorna data e hora do sistema           |
| *DAY*       	  | Retorna o dia de uma data  			     |
| *MONTH*     	  | Retorna o mês de uma data 	 			 |
| *YEAR*      	  | Retorna o ano de uma data 				 |
| *EOMONTH* 	  | Retorna o último dia do mês 		     |
| *DATEFROMPARTS* | Retorna uma data a partir de parâmetros  |
| *DATEDIFF* 	  | Retorna a diferença entre duas datas	 |
| *DATEADD* 	  | Adicionar um valor em uma data 			 |
| *DATENAME* 	  | Retorna o nome do dia da semana ou mês 	 |

Exemplos:

```sql
SELECT GETDATE(); --DATA E HORA DO SERVIDOR

SELECT
	NomeFuncionario,
	Admissao,
	DAY(Admissao) AS Dia,
	MONTH(Admissao) AS Mes,
	YEAR(Admissao) AS Ano
FROM Funcionario;

--RETORNAR ADMITIDOS NA 1° QUINZENA DE QUALQUER MÊS DO 2° SEMESETRE DOS SEGUINTES ANOS: 2000, 2003, 2005, 2008, 2010
SELECT
	NomeFuncionario,
	Admissao
FROM 
	Funcionario
WHERE
	DAY(Admissao) <= 15 AND
	MONTH(Admissao) >= 7 AND
	YEAR(Admissao) IN (2000, 2003, 2005, 2008, 2010);

SELECT EOMONTH(GETDATE(), 0);
SELECT EOMONTH(GETDATE(), 1);

SELECT DATEFROMPARTS(2020, 1, 10); --PODERIAM SER VARIÁVEIS

/*
	=> YEAR		YYYY
	=> QUARTER	Q
	=> MONTH    M
	=> WEEK		WW
	=> DAY		D
*/
SELECT DATEDIFF(DAY, '1991-12-05', GETDATE()) AS DiasVividos;
SELECT DATEDIFF(MONTH, '1991-12-05', GETDATE()) AS MesesVividos;
SELECT DATEDIFF(QUARTER, '1991-12-05', GETDATE()) AS TrimestresVividos;
SELECT DATEDIFF(WEEK, '1991-12-05', GETDATE()) AS SemanassVividas;

SELECT DATEADD(DAY, 65, GETDATE()) AS AdicionandoDias;
SELECT DATEADD(MONTH, 18, GETDATE()) AS AdicionandoMeses;
SELECT DATEADD(QUARTER, 5, GETDATE()) AS AdicionandoTrimestres;
SELECT DATEADD(WEEK, 3, GETDATE()) AS AdicionandoSemanas;
SELECT DATEADD(YEAR, 3, GETDATE()) AS AdicionandoAnos;

SET LANGUAGE 'BRAZILIAN';
SELECT 
	NomeFuncionario,
	Admissao,
	DATENAME(WEEKDAY, Admissao) AS DiaDaSemanaDaAdmissao,
	DATENAME(MONTH, Admissao) AS MesDaAdmissao
FROM Funcionario;
```

## Funções built-in (conversão e formato)

| Função  	| Descrição 		 	  |
| :----:  	| :----:           		  |
| *CAST*   	| Converter tipo de dados |
| *CONVERT* | Converter tipo de dados |
| *FORMAT*  | Formatar dados		  |

Exemplos:

```sql
SELECT
	NomeFuncionario,
	Admissao			--HORA ZERADA EM CAMPO DATETIME, POIS HORA NÃO FOI INFORMADA
FROM 
	Funcionario;

EXEC sp_help Funcionario; -- O CAMPO É DATETIME

SELECT
	NomeFuncionario,
	CAST(Admissao AS date) AS SomenteData			
FROM 
	Funcionario;

SELECT 'Média Final: ' + CAST(6.5 AS VARCHAR);

SELECT
	NomeFuncionario,
	Admissao,
	CONVERT(VARCHAR, Admissao, 1) AS Codigo1, --EN
	CONVERT(VARCHAR, Admissao, 2) AS Codigo2, --ISO
	CONVERT(VARCHAR, Admissao, 3) AS Codigo3, --BR
	CONVERT(VARCHAR, Admissao, 4) AS Codigo4,
	CONVERT(VARCHAR, Admissao, 5) AS Codigo5,
	CONVERT(VARCHAR, Admissao, 101) AS Codigo101, --EN com Século
	CONVERT(VARCHAR, Admissao, 102) AS Codigo102, --ISO com Século
	CONVERT(VARCHAR, Admissao, 103) AS Codigo103, --BR com Século
	CONVERT(VARCHAR, Admissao, 104) AS Codigo104,
	CONVERT(VARCHAR, Admissao, 105) AS Codigo105
FROM 
	Funcionario;

SELECT
	NomeFuncionario,
	Salario,
	FORMAT(Salario, 'C', 'PT-BR') AS MoedaPtBr,
	FORMAT(Salario, 'C', 'EN-US') AS MoedaEnUs,
	Admissao,
	FORMAT(Admissao, 'd', 'PT-BR') AS DataPtBr,
	FORMAT(Admissao, 'D', 'PT-BR') AS DataExtensoPtBr,
	FORMAT(Admissao, 'd', 'EN-US') AS DataEnUs,
	FORMAT(Admissao, 'D', 'EN-US') AS DataExtensoEnUs
FROM 
	Funcionario;

--TANTO CONVERT COMO FORMAT CONVERTEM OS DADOS PARA TEXTO
```