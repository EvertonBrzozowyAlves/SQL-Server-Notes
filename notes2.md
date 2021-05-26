# T-SQL

## Bancos de sistema e metadados

**Bancos de sistema do SQL Server**

- Master
    Principal banco de dados que o SQL Server possui.  
    Dentro dele é feito o controle e gerenciamento de todos os objetos que são criados tanto pelo sistema quanto pelos usuários.  

- TempDB
    Objetos temporários. Objetos que trabalham, geralmente, diretamente com a memória RAM do servidor.  

- Model
    Modelo base para criação de outros bancos de dados.  

- MSDB
    Esse banco é responsável por serviços que ocorrem no servidor, como logs, backups, restaurações, etc.  

- Resource
    Esse banco não fica visível. Utilizado somente em casos de atualização de servidor, como atualização da versão do SQL Server.  

> Esses bancos de dados ficam localizados em *Databases/System Databases/*. 
> Os bancos *model* e *msdb* são apenas leitura. O *master* e *tempdb* têm somente os objetos de servidor em modo somente leitura, mas aceita inclusão de objetos de usuário.  

### Metadados

Tratam-se dados sobre a estrutura de objetos no servidor. Podemos extrair métadados através de:
- Views (consultas armazenadas)
- Tabelas de sistema
- Funções
- Procedures (rotinas armazenadas)

> sys é um objeto de sistema

Exemplos:

```sql
-- Visualizando metadados de uma view
SELECT * FROM sys.objects;

-- Tabelas de sistema (XTYPE informa se é tabela de usuário)
SELECT * FROM SYSOBJECTS;

-- Tabelas de usuários
USE SisDep;
SELECT * FROM SYSOBJECTS WHERE xtype = 'U'; -- U => Tabelas de usuário

-- chaves primárias
USE SisDep;
SELECT * FROM SYSOBJECTS WHERE xtype = 'PK'; 

-- constraints unique
USE SisDep;
SELECT * FROM SYSOBJECTS WHERE xtype = 'UQ'; 

-- todas as colunas de todas as tabelas
USE SisDep;
SELECT * FROM SYSCOLUMNS; 

-- nome de uma tabela e suas colunas, como tipo de dados
USE SisDep;
SELECT 
	T1.name AS TableName, T2.name as ColumnName, T3.name as TypeName
FROM 
	SYSOBJECTS T1, SYSCOLUMNS T2, SYSTYPES T3
WHERE
	T1.id = T2.id AND 
	T3.xtype = T2.xtype AND 
	T1.xtype = 'U';

-- views de catálogo (do banco em uso)
USE SisDep;

SELECT * FROM SYS.TABLES;
SELECT * FROM SYS.COLUMNS;
SELECT * FROM SYS.TYPES;
SELECT * FROM SYS.INDEXES;

-- Procedures de catálogo
EXEC sp_tables;
EXEC sp_helpdb SisDep;
EXEC sp_spaceused 'Funcionario';

-- Funções de catálogo
SELECT OBJECT_ID('Funcionario');
```

## Funções adicionais de decisão e deslocamento

### Funções de decisão

**IIF - CHOOSE**

Sintaxe:
```SQL
IIF(ExpressaoLogica, RetornoVerdadeiro, RetornoFalso);

CHOOSE(Indice, Valor1, Valor2, ValorN);
```

Exemplos:

```sql
USE SisDep;

--IIF
SELECT 
	NomeFuncionario, 
	Admissao, 
	Salario,
	IIF(Salario <= 2000, 'Reajuste', '') AS AnaliseSalarial
FROM Funcionario

-- UPDATE COM IIF
BEGIN TRAN
	UPDATE Funcionario
	SET Salario *= IIF(Salario <= 2000, 1.1, 1.05)
	OUTPUT
		deleted.NomeFuncionario,
		deleted.Salario AS SalarioAnterior,
		inserted.Salario AS NovoSalario;
COMMIT

--CHOOSE
SELECT
	idMatricula, NomeFuncionario, Admissao,
	CHOOSE(DATEPART(WEEKDAY, Admissao), 'Dom', 'Seg', 'Ter', 'Qua', 'Qui', 'Sex', 'Sab') -- SELECIONADO O TEXTO DE ACORDO COM O ÍNDICE RETORNADO PELO DATEPART
FROM Funcionario;
```

### Funções de deslocamento

**LAG - LEAD**

Funções que permitem recuperar linhas abaixo ou acima em relação a um determinado registro de referência.  

Sintaxe:

```sql
LAG(Coluna, OffSet, DefaultText); --Linhas acima
LEAD(Coluna, OffSet, DefaultText); --Linhas abaixo
```

Exemplos:

```sql
-- LAG LEAD
SELECT
	idMatricula, NomeFuncionario, Admissao, Salario,
	LAG(Admissao, 1) OVER (ORDER BY idMatricula) AS AdmissaoLinhaAcima,
	LEAD(Admissao, 1) OVER (ORDER BY idMatricula) AS AdmissaoLinhaAbaixo
FROM Funcionario;

-- LAG LEAD (FAZENDO CÁLCULOS ENTRE LINHAS)
SELECT
	idMatricula, NomeFuncionario, Admissao, Salario,
	LAG(Salario, 1) OVER (ORDER BY idMatricula) AS SalarioLinhaAcima,
	LEAD(Salario, 1) OVER (ORDER BY idMatricula) AS SalarioLinhaAbaixo,
	DATEDIFF(DAY, Admissao, LAG(Admissao, 1, 0) OVER (ORDER BY idMatricula))
FROM Funcionario;
```

**FETCH - OFFSET**

Para tirar amostragens de registros da tabela, selecionando apenas parte de seu conteúdo

Sintaxe:

```sql
SELECT COLUNAS FROM TABELA
ORDER BY CAMPO
OFFSET N ROWS
FETCH NEXT N ROWS ONLY;
```

Exemplos:

```sql
--FETCH -- OFFSET
-- 20 PRIMEIRAS LINHAS A PARTIR DA 50°
SELECT *
FROM Funcionario
ORDER BY idMatricula
OFFSET 50 ROWS
FETCH NEXT 20 ROWS ONLY;

-- 10 LINHAS ALEATÓRIAS A PARTIR DA 30°
SELECT *
FROM Funcionario
ORDER BY NEWID() --GERA ID RANDÔMICO
OFFSET 50 ROWS
FETCH NEXT 20 ROWS ONLY;

```

## Manipulando colunas do tipo IDENTITY

**Recuperar informações de colunas auto incrementadas**

Exemplos:

```sql
USE RecursosAdicionais;

INSERT INTO Instrutores (nomeInstrutor) VALUES ('Hélio'), ('Agnaldo'), ('Magno'), ('Luciana');

SELECT * FROM Instrutores;

-- Excluir instrutor com id 20
DELETE FROM Instrutores WHERE id = 20;

-- Reutilizando o id 20
-- Ativando o modo insert em colunas identity 
SET IDENTITY_INSERT Instrutores ON; -- (Não é um comando de sessão!)

-- Agora é possível fazer o insert
INSERT INTO Instrutores (id, nomeInstrutor) VALUES (20, 'Everton');

-- Voltando a ter INSERT do tipo identity
SET IDENTITY_INSERT Instrutores OFF;

-- Listar uma coluna identity de uma tabela e seus valores
SELECT IDENTITYCOL FROM Instrutores;

-- Retornando o valor inicial
SELECT IDENT_SEED('Instrutores');

-- Retornando o valor de incremento
SELECT IDENT_INCR('Instrutores');

-- Retornando o último identity gerado no escopo da conexão de qualquer tabela
SELECT @@IDENTITY; -- Todas as funções iniciadas com @@ são funções globais

-- Retornando o último identity gerado no escopo da conexão de qualquer tabela
-- Mas gerado por procedure ou trigger
SELECT SCOPE_IDENTITY();

-- Retornar o valor original mais atual de uma coluna de identity de uma tabela
SELECT IDENT_CURRENT('Instrutores');

-- Zerar o contador do identity (Truncate)
DELETE FROM Instrutores;
INSERT INTO Instrutores (nomeInstrutor) VALUES ('Everton'); --depois desse insert, o identity continua a partir do último registro
SELECT * FROM Instrutores;

TRUNCATE TABLE Instrutores; --Apaga os dados da tabela e restaura os seus valores de campo identity para o inicial
INSERT INTO Instrutores (nomeInstrutor) VALUES ('Everton'); --depois desse insert, o identity volta para o inicial, com base no seed configurado
SELECT * FROM Instrutores;

--Zerar o contador do Identity (DBCC CHECKIDENT)
TRUNCATE TABLE Instrutores;
DBCC CHECKIDENT('Instrutores', RESEED, 5); --Alterando a configuração de seed inicial, feita para essa tabela
INSERT INTO Instrutores (nomeInstrutor) VALUES ('Everton'); 
SELECT * FROM Instrutores;
```

## Equiparando tabelas com MERGE

**Equiparar dados**

A cláusula *MERGE* nos permite comprar duas tabelas com o objetivo de equipararmos, ou seja, igualarmos as duas bases.  
Podemos chamar esta técnica de sincronização de réplicas.  

Útil quando se deseja ajustar alguns dados de uma tabela, com base em dados de outra tabela.  

Exemplos:

```sql
USE RecursosAdicionais;

SELECT * FROM Base1;
SELECT * FROM Base2;

MERGE Base2 AS Destino
USING Base1 AS Origem
ON Destino.id = Origem.id
WHEN MATCHED AND
	Destino.produto <> Origem.produto THEN
		UPDATE
		SET Destino.Produto = Origem.Produto
WHEN NOT MATCHED BY TARGET THEN
	INSERT
		(id, produto, valor)
	VALUES
		(id, produto, valor);

---------------------------------------------------------------------------------------------- 

MERGE Base1 AS Destino
USING Base2 AS Origem
ON Destino.id = Origem.id
WHEN MATCHED AND
	Destino.produto <> Origem.produto THEN
		UPDATE
		SET Destino.produto = Origem.produto
WHEN NOT MATCHED BY TARGET THEN
	INSERT
		(id, produto, valor)
	VALUES
		(id, produto, valor);

---------------------------------------------------------------------------------------------- 

MERGE Base2 AS Destino
USING Base1 AS Origem
ON Destino.id = Origem.id
WHEN MATCHED AND
	Destino.Valor <> Origem.Valor THEN
		UPDATE 
		SET Destino.Valor = Origem.Valor
WHEN NOT MATCHED BY TARGET THEN				-- o que fazer quando não encontrar no destino
	INSERT
		(id, produto, valor)
	VALUES
		(id, produto, valor)
WHEN NOT MATCHED BY SOURCE THEN				-- o que fazer quando não encontrar na origem
	DELETE
OUTPUT 
	$ACTION				AS	AcaoRealizada,	-- informa o verbo da ação realizada
	inserted.id			AS  IdInserido,
	inserted.Produto	AS	ProdutoInserido,
	inserted.Valor		AS	ValorInserido,
	deleted.id			AS	IdDeletado,
	deleted.produto		AS  ProdutoExcluido,
	deleted.valor		AS  ValorExcluido;
```

## Transpondo linhas e colunas

Algo que está na horizontal precisa ficar na vertical, ou vice versa?

**PIVOT**
Operador que permite transpor linhas em colunas

**UNPIVOT**
Operador que permite transpor colunas em linhas

Normalmente utilizada para alterar layouts de relatórios.

>Sempre preferir um alto números de linhas do que colunas, para fazer análises.

Exemplos:

```sql
--Transformando linhas em colunas (PIVOT)
USE RecursosAdicionais;
SELECT * FROM CotacoesPorDataMoeda;


SELECT
	DataCotacao, [USD] AS Dolar, [EUR] AS Euro, [LIB] AS Libra 
FROM 
	(
		SELECT 
			DataCotacao, CodMoeda, ValorCotacao
		FROM 
			CotacoesPorDataMoeda
	) AS PV
PIVOT
	(
		AVG(ValorCotacao) --médias
		FOR CodMoeda IN ([USD], [EUR], [LIB])
	) AS P;


--Transformando colunas em linhas (UNPIVOT)
USE RecursosAdicionais;
SELECT * FROM Investidores;


SELECT 
	Banco, Ano, Tipo, Valor --colunas ainda não existem
FROM Investidores 
UNPIVOT
	(
		Valor
		FOR Tipo IN(Investimentos, Despesas)
	) AS UPV
ORDER BY
	Banco, Ano, Tipo;

```
Documentação:
https://docs.microsoft.com/pt-br/sql/t-sql/queries/from-using-pivot-and-unpivot?view=sql-server-ver15

## Tabelas CTE

**CTE - Common Table Expression**

É um conjunto de dados temporários utilizados em instruções: *SELECT - UPDATE - INSERT - DELETE*.  
Pode ser referenciada várias vezes na mesma consulta, o que facilita a criação de consultas mais complexas.  

Exemplos: 

```sql
USE SysConVendas;

-- Exemplo com SELECT
WITH CTE(Ano, Mes, MaiorPedido)
AS
	(
		SELECT 
			YEAR(DataPedido) AS Ano, 
			Mes,
			MAX(DataPedido) AS MaiorPedido
		FROM 
			Dados
		GROUP BY
			YEAR(DataPedido),
			Mes
	)

SELECT * FROM CTE WHERE Ano = 2006 AND Mes = 'Abril'
UNION
SELECT * FROM CTE WHERE Ano = 2007 AND Mes = 'Junho';


-- Exemplo com UPDATE
-- Aplicar um reajuste de salários em 10% para os 10 funcionários mais antigos que ganham até 2.000 e ser admitido antes de 2005

USE SisDep;

WITH Reajuste
AS
	(
		SELECT TOP 10
			Salario, 
			Admissao,
			NomeFuncionario
		FROM 
			Funcionario
		WHERE 
			Salario <= 2000 AND YEAR(Admissao) < 2005
		ORDER BY
			Admissao ASC

	)
UPDATE 
	Reajuste  
SET 
	Salario *= 1.1 --Fazendo o update aqui, o valor do salário de funcionário será atualizado também
OUTPUT 
	deleted.NomeFuncionario,
	deleted.Salario AS SalarioAnterior,
	inserted.Salario AS NovoSalario;

```

## CROSS APPLY

**Correlação entre consultas**

Com o *CROSS APLLY*, podemos correlacionar resultados de duas ou mais consultas, sendo elas do tipo de junção (*JOIN*) ou subquery (subconsulta).  
Utilizado para evitar complexidade entre junções e subconsultas.

Exemplos:

```sql
USE SisDep;

SELECT DISTINCT
	F.NomeFuncionario,
	CA.MaisNovo,
	CA.MaisVelho
FROM 
	Funcionario F
		INNER JOIN
	Dependente D ON F.idMatricula = D.idMatricula
CROSS APPLY
(
	SELECT
		MAX(NascimentoDependente) AS MaisNovo,
		MIN(NascimentoDependente) AS MaisVelho
	FROM 
		Dependente AS D 
	WHERE 
		D.idMatricula = F.idMatricula
) AS CA;

--Quando ocorreu a primeira contratação, entre 2000 e 2005, nos departamentos de TI e Cobrança?
SELECT DISTINCT
	D.NomeDepartamento,
	CA.PrimeiraContratacao,
	CA.UltimaContratacao
FROM 
	Funcionario AS F
		INNER JOIN
	Depto AS D ON D.idDepartamento = F.idDepartamento AND 
	D.NomeDepartamento IN ('Administrativo', 'Cobrança')
CROSS APPLY
(
	SELECT 
		MAX(Admissao) AS UltimaContratacao,
		MIN(Admissao) AS PrimeiraContratacao
	FROM 
		Funcionario AS F
	WHERE 
		F.idDepartamento = D.idDepartamento AND 
		YEAR(Admissao) BETWEEN 2000 AND 2005
) AS CA;
```

## Tipos de dados de usuário - UDDT

**UDDT - User Defines Data Type**

É a possibilidade de criarmos um tipo de dados personalizado.  
Pode ser útil para agilizar a criação de tabelas.  
Além de podermos criar nossos próprios tipos, também é possível criar regras e valores padrão.  

Esse tipo de dado deve herdar de um dos tipos primitivos do SQL Server.  

Exemplo:
```sql
-- Tipos de dados do SQL Server
SELECT * FROM SYSTYPES; --Pegando dados de uma tabela de sistema
SELECT * FROM SYS.types; -- Pegando dados de uma view de sistema

--Criando banco para testes
CREATE DATABASE UDDT;
GO -- O GO server para informar ao SQL Server que, somente execute a ação seguinte, quando a anterior estiver concluída
USE UDDT;

----------------------------------------------------------------------

--Criando tipos de dados personalizados
CREATE TYPE NomePessoa
FROM NVARCHAR(50) NOT NULL;

CREATE TYPE ValorMonetario
FROM DECIMAL(10,2) NOT NULL; -- 8 números a esquerda da vírgula, 2 a direita

CREATE TYPE OpcaoSN
FROM CHAR(1) NOT NULL; 

CREATE TYPE DataRegra
FROM DATE NULL; 

--Visualizando apenas tipos criados pelo usuário (UDDT)
SELECT * FROM SYSTYPES WHERE uid = 1;

----------------------------------------------------------------------

-- Criando regras de validação
GO --Pausa necessária para criação de regras.
CREATE RULE RU_ValorMonetario AS @valor >= 0;
GO
CREATE RULE RU_OpcaoSN AS @sn IN ('S', 'N');
GO
CREATE RULE RU_DataRegra AS @data BETWEEN '2020-01-01' AND GETDATE();
GO

--Visualizando as regras criadas
SELECT * FROM SYSOBJECTS WHERE xtype = 'R'; --RULE

--Associando as regras aos tipos criados
EXEC sp_bindrule 'RU_ValorMonetario', 'ValorMonetario';
EXEC sp_bindrule 'RU_OpcaoSN', 'OpcaoSN';
EXEC sp_bindrule 'RU_DataRegra', 'DataRegra';

----------------------------------------------------------------------

--Criando DEFAULTS
GO
CREATE DEFAULT DF_SN AS 'S';
GO
CREATE DEFAULT DF_DataRegra AS GETDATE();
GO

--Visualizando os DEFAULTS
SELECT * FROM SYSOBJECTS WHERE xtype = 'D'; --DEFAULT

--Associando os defaults aos tipos criados
EXEC sp_bindefault 'DF_SN', 'OpcaoSN';
EXEC sp_bindefault 'DF_DataRegra', 'DataRegra';

----------------------------------------------------------------------

--Criando Sequencial (Identity personalizado)
CREATE SEQUENCE SQ_Registro
	START WITH			1000
	INCREMENT BY		10
	MINVALUE			10
	MAXVALUE			1000000
	CYCLE				--SE ATINGIR O MAX, RETORNA PARA 10

-- Visualizando sequências criadas
SELECT * FROM SYS.sequences;

----------------------------------------------------------------------

-- Utilizando UDDT e Sequência criada
CREATE TABLE tblClientes
(
	Id				INT				PRIMARY KEY,
	NomeCliente		NomePessoa,		--utilizando os tipos que criamos, já com suas regras
	StatusAtivo		OpcaoSN, 
	DataCadastro	DataRegra
);

CREATE TABLE tblFornecedores
(
	Id				INT				PRIMARY KEY,
	NomeFornecedor	NomePessoa,
	StatusAtivo		OpcaoSN,
	DataCadastro	DataRegra
);

--Inserindo dados
--Para utilizarmos a sequência devemos utilizar a instrução NEXT VALUE
INSERT INTO tblClientes
	(Id, NomeCliente, StatusAtivo, DataCadastro)
VALUES
	(NEXT VALUE FOR DBO.SQ_Registro, 'Everton', 'S', GETDATE());

-- Testando DEFAULTS
INSERT INTO tblClientes
	(Id, NomeCliente)
VALUES
	(NEXT VALUE FOR DBO.SQ_Registro, 'Joyce');

SELECT * FROM tblClientes;

-- Testando RULES
INSERT INTO tblClientes
	(Id, NomeCliente, StatusAtivo)
VALUES
	(NEXT VALUE FOR DBO.SQ_Registro, 'Joyce', 'J'); --Não permite inserção de 'J'

----------------------------------------------------------------------

--Sinônimos
--São como apelidos para objetos do banco
CREATE SYNONYM SN_IdRegistro FOR DBO.SQ_Registro;
CREATE SYNONYM SN_Cli FOR DBO.tblClientes;

--Visualizando sinônimos criados
SELECT * FROM SYS.synonyms;

-- Utilizando um sinônimo
SELECT * FROM SN_Cli;
```

## Objetos binários e FileStream

Campos binários permitem a inclusão de arquivos binários diretamente na tabela.  
Os tipos de dados binários são: *binary*, *varbinary* e *image*.  

**FileTable**
Recurso que permite carregar objetos binários numa do sistema Windows e ser gerenciado pelo SQL Server
Necessário habilitar FileStream no SQL Server Configuration Manager e no servidor.  

```sql
USE RecursosAdicionais;

CREATE TABLE tblDocsClientes (
	IdDocument		INT			IDENTITY		PRIMARY KEY,
	Description		VARCHAR(50),
	Document		VARBINARY(MAX) --MAX 8000 Caracteres
);
/*
Inserção de um arquivo binário
Single_Blob - Utilizado para codificação de leitura binária para arquivos
Ex.:
XML (Codificação de esquema)
TXT (Codificação ASCII)
PDF (Codificação binária)
*/

INSERT INTO tblDocsClientes
	(Description, Document)
SELECT 'Planilha Excel', BulkColumn
FROM OPENROWSET(BULK 'D:\Files\teste.xlsx', Single_Blob) AS ARQ; --armazena essa planilha como binário na tabela

SELECT * FROM tblDocsClientes;

--Para atualizar
UPDATE tblDocsClientes
SET Document = 
(
	SELECT * FROM OPENROWSET 
	(
		BULK 'D:\Files\teste2.xlsx', Single_Blob
	) AS ARQ
);

-- O fluxo de trabalho acima consome bastante performance. Para muitos arquivos, o ideal é trabalhar com FILETABLE
-- Passo 1: Habilitar FILESTREAM no SQL Server Configuration Manager => properties, filestream, habilita tudo

-- Passo 2: Habilitar FILESTREAM no servidor (2 = Permissão FULL) => script abaixo
EXEC sp_configure filestream_access_level, 2
RECONFIGURE;

-- Passo 3: Banco de Dados com Grupo de Arquivos
CREATE DATABASE Banco_Filestream 
ON PRIMARY (
	NAME = FG_Filestream_PRIMARY, 
	FILENAME = 'C:\DADOS\Filestream_DATA.mdf'), 
	FILEGROUP FG_Filestream_FS 
	CONTAINS FILESTREAM (
		NAME = Filestream_ARQ, 
		FILENAME='C:\DADOS\Filestream_ARQ') 
	LOG ON (
		NAME = Filestream_log, 
		FILENAME = 'C:\DADOS\Filestream_log.ldf') 
		WITH FILESTREAM (NON_TRANSACTED_ACCESS = FULL, 
		DIRECTORY_NAME = N'Filestream_ARQ');


-- Passo 4: Verificar Options - FileStream - Directory Name (Propriedades do Banco)
-- no SSMS, properties no banco de dados, options, FILESTREAM => tem que ter o Filestream directory name e permissão FULL.

-- Passo 5: Criar FileTable
USE Banco_Filestream;

CREATE TABLE FT_Documento AS FileTable    
	WITH 
	(
		FileTable_Directory = 'Filestream_ARQ',
		FileTable_Collate_Filename = database_default --configuração padrão do banco para acentuação, etc
	); 

--Após criada a tabela FileTable, ela ficará dentro do banco de dados, na pasta tables, mas dentro da pasta FileTables.
--Ao clicar com o botão direito do mouse, será dada a opção de explorar arquivos, na qual podemos fazer operações de arquivos dia diretório do windows explorer.
--Depois, basta fazer um select na tabela e os arquivos estarão lá.
SELECT * FROM FT_Documento;
```

## Colunas computadas

No SQL é possível realizarmos operações matemáticas.  
Geralmente isso é feito dentro da própria instrução SQL.  
Imagine um cálculo complexo que desejamos utilizar em várias consultas.  
Nesse caso, o ideal é criar uma coluna computada com este cálculo.  

Exemplo:

```sql
USE SisDep;

SELECT 
	idMatricula, NomeFuncionario,Salario, Admissao,
	FLOOR(CAST((GETDATE() - Admissao) AS FLOAT) / 365.25) AS AnosTrabalho --cast para float pois o sql não faz conversão implícita 
FROM Funcionario;

--CRIANDO COLUNA COMPUTADA
ALTER TABLE FUNCIONARIO
ADD TempoServicoEmAnos AS
FLOOR(CAST((GETDATE() - Admissao) AS FLOAT) / 365.25);

--Conferindo
SELECT * FROM Funcionario;

```

## Otimizando consultas

**Plano de execução**
Através desse recurso é possível acompanhar o plano estratégico de execução de uma consulta pelo SQL Server.  

Como ter consultas mais ágeis?

- Ter um banco normalizado (Bancos OLTP com baixa quantidade de dados).
- Adicionar Constraint de PK.
- Evite usar '*'. Selecione apenas as colunas que realmente precisar, nomeando-as.
- Evite campos binários (salvo exceções em que há a necessidade de armazenar arquivos)
- Se for necessário trabalhar com binários, utilize FileStream/FileTable
- Quando possível, evite tabelas temporárias (exceto se houver índice)
- Evite trabalhar com cursores
- *WHERE* - Procure utilizá-lo em colunas estratégicas que possuam índices
- Evite a todo custo: *LIKE - NOT*
- Evite Hints ao máximo (customizações de índices dentro de uma tabela)
- Crie índices
- Evite consultas ad hoc (distribuídas/externas)

**Conceito de índice**

Tem um papel importante dentro do banco de dados, pois permite agilizar a busca de registros dentro de uma tabela.  

- Estruturas
	- **CLUSTERED**: A coluna de índice é ordenada fisicamente na tabela
	- **NONCLUSTERED**: A coluna de índice é ordenada logicamente na tabela

>Evite criar índices em tabelas/colunas que sofram muitos updates, pois o índice torna esse processo muito mais lento.  

Exemplos:

```sql
USE SysConVendas;

SELECT * FROM Dados;

EXEC sp_help Dados;

-- Visualizar especificamente os índices
EXEC sp_helpindex Dados;

-- Configurar padrão de entrada de datas
SET DATEFORMAT DMY; --Para inserir datas no format DD/MM/AAAA (Comando de sessão)

-- Visualizando plano de execução (selecione a query e CTRL + L). Nesse momento, 41% table scan, 59% Sort, 0% select.
SELECT 
	* 
FROM 
	Dados
WHERE
	DataPedido BETWEEN '01-01-2007' AND '30-06-2007'
ORDER BY DataPedido DESC;

-- Criar índices
CREATE INDEX IX_Dados_DataPedido
ON DADOS(DataPedido);

CREATE INDEX IX_Dados_Vendedor
ON DADOS(Vendedor);

CREATE INDEX IX_Dados_Produto
ON DADOS(Produto);

-- Criando índices compostos e que não permitem inclusão de registro com quantidade de colunas repetidas
CREATE UNIQUE INDEX UQ_Dados_Registro
ON DADOS(Pedido, DataPedido, Vendedor, Cidade, Produto);

-- Excluir índice
DROP INDEX DADOS.IX_Dados_Produto;

-- HINTS
-- forçar o SQL a utilizar pesquisa no índice

SELECT 
	* 
FROM 
	Dados
WITH
	(INDEX = IX_Dados_Produto)
WHERE 
	Produto = 'Camisa'; --nesse momento, o plano de execução já não utiliza table scan, o que é ótimo

SELECT 
	*
FROM 
	Dados
WITH(INDEX = UQ_Dados_Registro)
WHERE Vendedor = 'Gisele'  AND Produto = 'Gravata';

-- MAXDOP
-- Definir a quantidade de processadores para executar a consulta
SELECT 
	*
FROM 
	Dados
WHERE 
	Mes = 'Junho'
ORDER BY
	Regiao, Cidade
OPTION(MAXDOP 2); -- 2 processadores - por padrão as consultas são feitas com apenas 1 processador
-- nas propriedades do servidor, na guia geral, é possível visualizar a quantidade de processadores da máquina
```

**Imagens do plano de execução**

- Plano de execução antes de inserir índices:  
![Alt text](Material/ExecPlan1.png?raw=true "Plano de execução antes de inserir índices")

- Plano de execução depois de inserir índices:  
![Alt text](Material/ExecPlan2.png?raw=true "Plano de execução depois de inserir índices")


## Controlando bloqueios

**Bloqueio na atualização de registros**

São bloqueios empregados quando estamos fazendo atualizações de registro, como *UPDATE*, por exemplo.

- Tipos de bloqueio
	- Linha (*ROWLOCK*): É chamado toda vez que um registro está sofrendo atualização. Os demais registros ficam disponíveis.  
	- Página (*PAGELOCK*): Funciona como o registro de linha, mas para a página em que o registro está.  
	- Tabela (*TABLOCK*): Bloqueio de todos os registros da tabela.  

**Ler dados bloqueados**

Para ler dados bloqueados, podemos incluir em nosso *SELECT* as seguintes opções:

- *WITH NO LOCK*: Esta opção fará o que chamamos de "Dirty Read", ou seja, leitura suja, considerando que a instrução *UPDATE* ainda não foi confirmada.  
- *WITH READPAST*: Esta opção retornará somente os registros que não estão sendo afetados pelo *UPDATE*.

**Personalizar bloqueio**

Temos uma instrução que nos permite configurar no servidor o comportamento da leitura de registros:

*SET TRANSACTION ISOLATION LEVEL*  

Que pode ser configurado para:  

- *READ COMMITED*: Ler somente os dados já confirmados dentro da transação de um comando *UPDATE - INSERT - DELETE*.  
- *READ UNCOMMITED*: Lert todos os registros, mesmo aqueles que ainda não foram confirmados dentro da transação.  

Exemplos:

```sql
-- Usuário A
USE SISDEP;

-- Atualização na tabela funcionários
BEGIN TRAN TR01;
	UPDATE FUNCIONARIO
	SET Salario += 100
	WHERE Salario <= 1500;
-- Usuário A ainda não confirmou a transação
COMMIT;
ROLLBACK;

-- Ir até a janela Usuário B e tentar selecionar dados da tabela funcionario
-- Usuário B tenta ler dados e não consegue, pois tem um bloqueio em andamento
SELECT * FROM Funcionario;

-- Customizando nível de bloqueio
-- (Nível de Linha) Default
BEGIN TRAN TR02;
	UPDATE FUNCIONARIO WITH(ROWLOCK)
	SET Salario += 100
	WHERE idMatricula IN(1000,1025,1040);

-- Usuário A ainda não confirmou a transação
COMMIT;

-- Forçar a ler dados com atualizações não confirmadas
-- Isto causa leitura suja (Dirty Read)
SELECT * FROM Funcionario WITH(NOLOCK);
-- Ler somente dados que não estão participando da transação (Leitura segura)

-- Ir até a janela usuário B e executar o select with readpast (97 linhas)
-- Isto causa uma leitura de dados segura
SELECT * FROM Funcionario WITH(READPAST);
-- Confirmar transação

-- (Nível de Tabela)
BEGIN TRAN TR03;
	UPDATE FUNCIONARIO WITH(TABLOCK)
	SET Salario += 100
	WHERE idMatricula IN(1000,1025,1040);
-- Ir até a janela usuário B e executar o select ReadPast e NoLock
-- Aqui o READPAST não funcionará, pois existe um bloqueio travando toda a tabela

-- Confirmar transação
COMMIT;

-- Configurando TimeOut (nesta Seção)
-- Unidade de Tempo (Milisegundos)
DBCC USEROPTIONS;
SET LOCK_TIMEOUT 5000; -- 5 segundos

-- Executar novamente a transação TR03
-- Ir até a janela usuário B e configurar timeout

-- visualizando bloqueios Pelo nome do objeto
SELECT * FROM sys.dm_tran_locks
WHERE resource_associated_entity_id = OBJECT_ID('Funcionario')

-- visualizando bloqueios por transação
select * from sys.dm_tran_locks
where request_owner_type = 'transaction';

-- Executar a transação TR02 (Usuário A) e TR04 (Usuário B)
COMMIT

-- Executar a transação TR01 e TR03

-- Visualizando transações ativas nesta seção
SELECT @@TRANCOUNT;
-- Visualizando Active Monitor
-- CTRL + ALT + A => podemos tirar um relatório para ver bloqueios em outras seções
-- Visualizando Report All Transactions

-- Alterando o nível de isolamento
SET TRANSACTION ISOLATION LEVEL READ UNCOMMITTED;

-- Vá até usuário A e execute transação TR03
-- Usuário B tenta ler dados
SELECT * FROM Funcionario;

-- Retornar para o nível de isolamento read committed
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;

-- Usuário B tenta ler dados novamente
SELECT * FROM Funcionario;

-- Confirmar transação TR03 em usuário A

-- Usuário B inicia nova transação
BEGIN TRAN TR04
	UPDATE Projeto WITH (ROWLOCK)
	SET [DataTermino] = GETDATE()
	WHERE idProjeto = 2

SELECT * FROM projeto
```

## Integrando dados com o Microsoft Office

```sql
IF(EXISTS(SELECT Name FROM master.dbo.sysdatabases WHERE name = 'DadosExternos'))
	BEGIN
		USE master;
		DROP DATABASE DadosExternos;
	END
GO
CREATE DATABASE DadosExternos
GO
USE DadosExternos;

CREATE TABLE Pedidos
(
	id			int				PRIMARY KEY,
	Produto		nvarchar(100)	NOT NULL,
	DataPedido	date			NOT NULL,
	Quantidade	smallint		NOT NULL,
	PrecoUnit	decimal(9,2)	NOT NULL
);
GO
CREATE TABLE Clientes
(
	id					int				PRIMARY KEY,
	NomeCliente			nvarchar(50),
	CadastroCliente		date
);
INSERT INTO Clientes
(id,NomeCliente,CadastroCliente)
VALUES
(10,'Ana','2015-1-5'),
(20,'Carlos','2015-2-10'),
(30,'Eduardo','2015-3-3'),
(40,'Fernanda','2015-4-15'),
(50,'Mariana','2015-5-20');
GO
CREATE TABLE Produtos
(
	codProduto			int		primary key,
	descricaoProduto	nvarchar(100),
	precoProduto		smallmoney,
	qtdeEstoque			int
);

-- Habilitar visibilidade das opções avançadas
EXEC sp_configure 'show advanced option','1';
RECONFIGURE;
-- Habilitar a função OPENROWSET
EXEC sp_configure 'Ad Hoc Distributed Queries','1';
RECONFIGURE;

-- Importar Dados de um arquivo Excel
-- C:\DADOS\Pedidos.xlsx

-- Habilitar Allow InProcess (Server Objects - Linked Server - Providers)
INSERT INTO Pedidos
SELECT * FROM OPENROWSET
	(
		'Microsoft.ACE.OLEDB.12.0',
		'Excel 12.0;Database=C:\Dados\Pedidos.xlsx',
		'SELECT * FROM [Dados$]'
	);

SELECT * FROM Pedidos;

-- Exportar Dados para um arquivo Excel
INSERT INTO OPENROWSET
	(
		'Microsoft.ACE.OLEDB.12.0',
		'Excel 12.0;Database=C:\Dados\Clientes.xlsx',
		'SELECT * FROM [Dados$]'
	)
SELECT * FROM Clientes;

-- Consultar dados de um arquivo Excel
SELECT * FROM OPENROWSET
	(
		'Microsoft.ACE.OLEDB.12.0',
		'Excel 12.0;Database=C:\Dados\Medicamentos.xlsx;HDR=YES',
		'SELECT * FROM [Base$] where qtde > 100'
	);

-- Consultar dados de um Banco Access
SELECT * FROM OPENROWSET
	(
	'Microsoft.ACE.OLEDB.12.0',
	'C:\Dados\Controle Operacional.accdb';'admin';'',DADOS
	);
SELECT * INTO #teste
FROM OPENROWSET
	(
	'Microsoft.ACE.OLEDB.12.0',
	'C:\Dados\Controle Operacional.accdb';'admin';'',DADOS
	);
```

## Integrando dados com arquivos de texto

**Bulk Insert**

Permite importar dados nos formatos *TXT* ou *CSV*.

Exemplo:

```sql
BULK INSERT
NOME_TABELA
FROM 'C:DADOS.TXT'
WITH (FIELDTERMINATOR=';', ROWTERMINATOR='\n', CODEPAGE='acp')
```
- FIELDTERMINATOR: Registra qual caractere irá indicar fim de um campo para o próximo
- ROWTERMINATOR: Registra qual caractere irá indicar fim de um registro para o próximo.
- CODEPAGE: Configuração para leitura correta de acentuação. Para países latinos, utilizar o 'acp'.

Exemplos:

```sql
-- Importando Dados de um arquivo de texto
USE DadosExternos;

BULK INSERT 
Produtos
FROM 'C:\DADOS\Produtos.txt'
WITH     
	(
		FIELDTERMINATOR =';',
		ROWTERMINATOR = '\n',
		CODEPAGE = 'ACP'
	);

SELECT * FROM Produtos;
---------------------------------------------------------------------------------------
CREATE TABLE Cursos(
	idCurso		int		primary key,
	nomeCurso	nvarchar(50),
	valorCurso	smallMoney
);

-- Utilizar BULK INSERT para inserir o arquivo C:\Dados\Cursos.txt
-- na tabela Cursos

BULK INSERT 
Cursos
FROM 'C:\DADOS\Cursos.txt'
WITH     
	(
		FIELDTERMINATOR =';',
		ROWTERMINATOR = '\n',
		CODEPAGE = 'ACP'
	);

SELECT * FROM Cursos;
```

>Caso seja necessário inserir arquivos diariamente, que contenham dados cumulativos, o ideal é usar um *TRUNCATE TABLE* ao invés de *DELETE*.  
O *DELETE* gera fragmentos que, ao longo de várias instruções de *DELETE*, gera informações imprecisas sobre o tamanho das tabelas,por exemplo.  


## Integrando dados com XML e JSON

**XML**

*eXtensible Markup Language*: é uma recomendação da W3C para gerar linguagens de marcação.  
A principal característica do xml é criar uma infraestrutura única para diversas linguagens.  

**JSON**

*JavaScript Object Notation*: é um formato leve para intercâmbio de dados que possui uma notação de escrita mais fácil do que o XML.  

Exemplos:
```sql
RAW('Ítem_Pedido')-- Gerando dados XML
USE SysConVendas;
SELECT * FROM Dados;

-- XML RAW sem tag raíz (somente linha de registro)
SELECT * FROM Dados
FOR XML RAW;

-- Criando uma tag principal
SELECT * FROM Dados
FOR XML RAW('Ítem_Pedido'),ROOT('Pedidos'); --Pedidos fica como node principal, Ítem_Pedido cada linha

-- Dados com estrutura de elemento (PREFERÍVEL)
SELECT * FROM Dados
FOR XML RAW('Ítem_Pedido'),ROOT('Pedidos'),ELEMENTS;

-- Lidando com campos nulos
BEGIN TRAN TR01
	UPDATE Dados WITH(ROWLOCK)
	SET Vendedor = NULL
	WHERE Pedido = 21768
COMMIT;

-- Perceber que o campo vendedor não é exibido no pedido 21768
SELECT * FROM Dados
FOR XML RAW('Ítem_Pedido'),ROOT('Pedidos'),ELEMENTS; -- não é gerado o campo no xml

-- Solução XSINIL
SELECT * FROM Dados
FOR XML RAW('Ítem_Pedido'),ROOT('Pedidos'),ELEMENTS XSINIL; --Dessa forma, é gerado o campo no xml, mesmo sem valor

-- XML Tag AUTO
SELECT * FROM Dados
FOR XML AUTO,ROOT('Pedidos');


-- Método QUERY
-- Declarando uma variável XML 
DECLARE @xml XML
-- Carrega as informações da consulta para a variável XML, utilizando o FOR XML: 
SET @xml =  
( 
	SELECT Pedido,DataPedido,Regiao,produto,total 
	FROM Dados AS Pedidos
	FOR XML RAW('ItemPedido'), ROOT('Pedidos'), ELEMENTS XSINIL 
 );

SELECT @xml.query('Pedidos');

-- Consultando um campo específico

SELECT @xml.query('Pedidos/DataPedido');

-- Consultando um registro específico
SELECT @xml.query('Pedidos[10]/DataPedido');

-- Verificar Existência de um campo
SELECT @XML.exist('Pedidos/Codigo');
  
SELECT @XML.exist('Pedidos/Pedido');


-- Verificar Existência de um registro
SELECT @XML.exist('Pedidos/Codigo[100]');
  
SELECT @XML.exist('Pedidos/Pedido[5000]');

--Habilitar opções avançadas 
EXEC sp_configure 'show advanced option',1
RECONFIGURE
--Habilitar a execução da procedure XP_CMDSHELL
EXEC sp_configure 'xp_cmdshell',1
RECONFIGURE

-- Exportar XML
/*
	BCP
	• Queryout: Nome do arquivo de saída;
	• S:Nome do servidor;
	• T: Acesso através de conta do Windows;
	• w: Utiliza UNICODE;
	• r: Terminador de linha;
	• t: Terminador de campo {TAB}.

*/


DECLARE @comando VARCHAR(4000);

SET @comando = 'BCP "SELECT * FROM SysConVendas.dbo.Dados AS TIPO FOR XML AUTO, ROOT(''RESULTADO''), ELEMENTS " '
	 + ' QUERYOUT "C:\DADOS\Dados.XML" -SDESKTOP-UASTCAV\SQLEXPRESS -t -w -t -T'

EXEC MASTER..XP_CMDSHELL  @comando;

-- Gerar Dados JSON
SELECT * FROM Dados
FOR JSON AUTO,ROOT('Pedidos');

-- Ler Dados JSON
USE master;
SELECT * FROM OPENJSON(
	'["São Paulo", "Rio de Janeiro", "Minas Gerais", "Paraná", "Santa Catarina"]'
	);

-- Exportar JSON

DECLARE @comandojs VARCHAR(4000)
SET @comandojs = 
	'BCP "SELECT * FROM SysConVendas.dbo.Dados AS TIPO FOR JSON AUTO" '
		+ ' QUERYOUT "C:\DADOS\ARQUIVOJSON.XML" -SDESKTOP-UASTCAV\SQLEXPRESS -t -w -t -T'
EXEC MASTER..XP_CMDSHELL  @comandojs
```

## Objetos de consulta – View padrão

Trata-se de um objeto do servidor que nada mais é do que uma consulta que foi armazenada e contém as mesmas características de uma tabela (linhas e colunas).  
Não contém dados, apenas uma visão dos dados de uma tabela.  
Poderá ser útil no encapsulamento de consultas complexas, além de permitir restrição de visualização e inserção de dados.  

**Tipos de View**

- Padrão: Mais simples. Objetivo de selecionar algumas colunas de uma determinada tabela, encapsular um comando *SELECT* mais complexo.  
- Indexadas: Objetivo de obter um ganho maior de performance na execução de uma consulta.  
- Particionadas: É possível criar algum tipo de restrição para que o dado não entre incorretamente em uma tabela.  

Exemplos:

```sql
/*
	View: Objeto no catálogo de banco formado por linhas e colunas formando
	uma tabela virtual que é referenciado a tabelas físicas definidas dentro
	de uma query.

	Tipos de View:

	Standard		Dados reunidos provenientes de tabelas ou outras views

	Indexada		Estrutura de dados criada por índice clusterizados
					Boa opção para otimizar leitura de informações

	Particionadas	Opção que permite que uma tabela massiva de dados seja
					dividida em tabelas menores

	Vantagens
	---------

	. Reutilização
	. Redução no custo de execução
	. Segurança
	. Compatibilidade
	. Cópia de dados
	. Encapsulamento de códigos complexos
	. Extensibilidade a outros aplicativos

	Restrições
	----------

	Uma view não poderá:

	. Gerar duas colunas com o mesmo nome
	. Utilizar ORDER BY (exceto se incluída na instrução TOP)
	. Utilizar a cláusula INTO
	. Fazer referência a uma tabela temporária ou variáveis
	. Utilizar variáveis
	. Utilizar (*) para referenciar todos os campos (exceto se especificarmos SCHEMABINDING
	. Ultrapassar a 1024 colunas
	. Ter mais de 32 níveis de aninhamento
	. Ter mais de um CREATE VIEW dentro de um Batch (exceto especificado com instrução GO)

*/

-- Banco Consórcio
USE Consorcio;
SELECT * FROM Carteiras;
GO
CREATE VIEW vw_Vendas_SP
AS
	SELECT UF,Cidade,CAST(DataContrato AS date) AS [Data Contrato],
	CAST(ValorAprovado + Entrada AS decimal(10,2)) AS Total
	FROM Carteiras;
GO
-- Visualizando dados de uma view
SELECT * FROM vw_Vendas_SP;

-- Alterando uma view
GO
ALTER VIEW vw_Vendas_SP
AS
	SELECT UF,Cidade,CAST(DataContrato AS date) AS [Data Contrato],
	CAST(ValorAprovado + Entrada AS decimal(10,2)) AS Total
	FROM Carteiras
	WHERE Uf = 'SP';
GO

-- Visualizando o código de uma view (CTRL + T = Saída de Texto tem melhor exibição)
-- Via procedure
EXEC sp_helptext vw_Vendas_SP;

-- View de Catálogo
SELECT * FROM syscomments;
SELECT * FROM SYS.views;
SELECT * FROM Sys.sql_dependencies 
where object_id = 773577794
-- Encriptografando a view

-- Banco Sisdep
USE SisDep;
GO
CREATE VIEW vw_DadosFuncionarios
WITH ENCRYPTION
AS
	SELECT
		F.idMatricula,F.NomeFuncionario,D.NomeDepartamento,
		C.NomeCargo,F.Admissao,F.Salario
	FROM Funcionario AS F 
		INNER JOIN Depto AS D		ON F.idDepartamento = D.idDepartamento
		INNER JOIN Cargo AS C		ON F.idCargo = C.idCargo;

GO
-- Tentativa de visualizar o código da view
EXEC sp_helptext vw_DadosFuncionarios;

SELECT * FROM syscomments;

-- Bloqueio de Estrutura
GO
CREATE VIEW vw_MaioresSalarios
WITH SCHEMABINDING
AS
	SELECT	TOP 5 idMatricula,NomeFuncionario,Admissao,Salario
	FROM dbo.Funcionario -- Owner obrigatório
	ORDER BY Salario DESC;
GO

SELECT * FROM vw_MaioresSalarios;

-- Tentativa de alterar a estrutura da tabela funcionário
exec sp_help Funcionario;

-- Alterar tamanho do campo CPF (Vai funcionar, pois Cpf não participa da View)
ALTER TABLE Funcionario
ALTER COLUMN Cpf nvarchar(11);

-- Alterar tamanho do campo idMatricula (Vai dar erro, pois este campo pertence a View)
ALTER TABLE Funcionario
ALTER COLUMN idMatricula int;

-- CHECK OPTION (Restrição de Entrada de Dados)
GO
CREATE VIEW vw_NovaLocalidadeSP
AS
	SELECT idLocalidade,Uf,Cidade FROM Localidade
	WHERE Uf = 'SP'
WITH CHECK OPTION;
GO 
-- Testando
-- Vai dar erro
INSERT INTO vw_NovaLocalidadeSP
(idLocalidade,Uf,Cidade)
VALUES
(12,'RJ','Parati');
--Vai Funcionar
INSERT INTO vw_NovaLocalidadeSP
(idLocalidade,Uf,Cidade)
VALUES
(12,'SP','Ribeirão Preto');

-- Excluir uma View
DROP VIEW vw_NovaLocalidadeSP;
```

## Objetos de consulta – View particionada

Através da instrução *WITH CHECK OPTION* podemos criar uma View com instrução de entrada de dados em uma tabela.  

Exemplo:

```sql
USE SisDep;
-- CHECK OPTION (Restrição de Entrada de Dados)
GO
CREATE VIEW vw_NovaLocalidadeSP
AS
	SELECT idLocalidade,Uf,Cidade FROM Localidade
	WHERE Uf = 'SP'
WITH CHECK OPTION;
GO 

-- Testando
-- Vai dar erro
INSERT INTO vw_NovaLocalidadeSP
(idLocalidade,Uf,Cidade)
VALUES
(12,'RJ','Parati');

--Vai Funcionar
INSERT INTO vw_NovaLocalidadeSP
(idLocalidade,Uf,Cidade)
VALUES
(12,'SP','Ribeirão Preto');
```

> Lembrando que esses bloqueios ficam limitados a view, e não a tabela em si.  

## Objetos de consulta – View indexada

Podemos criar um índice dentro de uma view.  
Podemos ter mais de um índice, porém um índice *UNIQUE CLUSTERED* é obrigatório.  

Exemplos:

```sql
USE SisDep;
-- VIEW INDEXADA (Obter Melhor Performance)
GO
CREATE VIEW vw_ConsultaSimples
WITH SCHEMABINDING
AS
	SELECT
		idMatricula,NomeFuncionario,Admissao,Salario
	FROM dbo.Funcionario;
GO

CREATE UNIQUE CLUSTERED INDEX IX_ConsultaSimples
ON dbo.vw_ConsultaSimples(NomeFuncionario,Admissao);

SELECT * FROM vw_ConsultaSimples
WHERE NomeFuncionario LIKE 'C%';
```

## Variáveis e controle de decisão

### Variáveis

São endereços na memória (RAM) para armazenamento temporário de informações.  
Podem ser úteis para programar o fluxo de uma rotina programada, bem como validar tipos de dados e realizar operações mais complexas.  

**Escopo**

Deve-se entender por escopo a visibilidade de uma variável.  

Existem dois tipos:
- Local: Existe somente na sessão onde ela foi declarada, criada com *@*.  
- Global: Reconhecida por todas as sessões com conexão ativa ao servidor, criada com *@@*.  

**Declaração**

É a ação de definir o nome da variável, seu tipo de dado e sua dimensão.  

```sql
DECLARE @NomeVariavel TipoDeDado;
```

**Atribuição**

Toda variável no SQL Server obrigatoriamente tem sua inicialização com algum determinado valor.  
Para realizarmos essa operação, é utilizada a instrução *SET*:  

```sql
SET @NomeVariavel = Valor;
```

Também podemos definir o valor da variável no mesmo momento em que a declaramos:  

```sql
DECLARE @NomeVariavel TipoDeDado = Valor;
```

### Controle de decisão

**Teste de comando único**

```sql
IF CondicaoVerdadeira
```

**Bloco de comandos**

```sql
IF CondicaoVerdadeira
	BEGIN
		Comandos
	END
```

**Teste com verdadeiro e falso**

```sql
IF CondicaoVerdadeira
	BEGIN
		Comandos
	END
ELSE
	BEGIN
		Comandos
	END
```

### Exemplos

```sql
/*
	Variáveis:
	
	. São endereços de memória utilizados para alocar informações temporárias
	. Podem ser criadas em procedures ou batches
	. Devem obrigatoriamente ser declaradas (DECLARE)
	. Devem obrigatoriamente ser inicializadas (SET)
	. Possuem escopo local (@nome) e global (@@nome)
	. Nomes de variáveis:
		> Não podem iniciar por numeral
		> Não podem utilizar palavras reservadas
		> Tamanho máximo de 255 caracteres
		> Restrição a caracteres especiais !@#$%&*(){}[]<>,.;:
*/ 

-- Exemplos
DECLARE @a int,@b int,@c decimal(6,2);
SET @a = 20;
SET @b = 4;
SET @c = @a / @b;
PRINT @c;

-- Falha na conversão implícita de tipos
DECLARE @a2 int,@b2 int,@c2 decimal(6,2);
SET @a2 = 20;
SET @b2 = 3;
SET @c2 = CAST(@a2 AS decimal(6,2)) / CAST(@b2 AS decimal(6,2));
PRINT @c2;

/*
	Operadores Aritiméticos
	+	Adição
	-	Subtração
	/	Divisão
	*	Multiplicação
	%	Resto da divisão

	Operadores Relacionais

	>		Maior
	<		Menor
	>=		Maior ou igual
	!<		Maior ou igual
	<=		Menor ou igual
	!>		Menor ou igual
	=		igual
	<>		Diferente
	!=		Diferente
	
	Operadores Lógicos

	ALL			Verdadeiro se todas as condições forem (V)
	AND			Verdadeiro se duas expressões forem (V)
	ANY			Verdadeiro se qualquer condição for (V)
	BETWEEN		Verdadeiro se estiver dentro do intervalo de valores
	EXISTS		Verdadeiro se existir no conjunto de valores
	IN			Verdadeiro se existir na lista de valores
	LIKE		Verdadeiro se parte da informação é encontrada
	NOT			Reverter um valor lógico (Boleano)
	OR			Verdadeiro se uma das expressões for (V)
	SOME		Verdadeiro se parte das comparações for (V)


*/

-- Controle de Fluxo

-- IF / ELSE
DECLARE @rnd1 float, @rnd2 float; 
SET @rnd1 = (SELECT RAND());
SET @rnd2 = (SELECT RAND());
select @rnd1 = RAND() 
IF @rnd1 > @rnd2
	PRINT 'O 1º Aleatório é maior'
ELSE
	PRINT 'O 2º Aleatório é maior';

-- Bloco com mais de um comando
DECLARE @rnd3 float, @rnd4 float ;
SET @rnd3 = (SELECT RAND());
SET @rnd4 = (SELECT RAND());

IF @rnd3 > @rnd4
	BEGIN
		PRINT @rnd3;
		PRINT 'O 1º Aleatório é maior';
	END
ELSE
	IF @rnd4 > @rnd3
		BEGIN
			PRINT @rnd4;
			PRINT 'O 2º Aleatório é maior';
		END
	ELSE
		PRINT @rnd3;
		PRINT 'Os número são iguais!!'

-- Aleatório entre 1 e 60
DECLARE @rnd5 int
SET @rnd5 = ((SELECT RAND() * 59) + 1);
PRINT @rnd5;
```

## Operadores lógicos e controle de decisão

Podemos utilizar operadores lógicos para controlar o fluxo de execução do código.  

Exemplos:

```sql
-- Operadores Lógicos OR AND
DECLARE @uf char(2),@valorPedido smallmoney,@imposto smallmoney;
SET @uf = 'PR';
SET @valorPedido = 1000;
IF @uf = 'SP' OR @uf = 'RJ'
	BEGIN
		SET @imposto = @valorPedido * 0.1;
		PRINT 'Imposto Retido (10%): ' + FORMAT(@imposto,'c','pt-br')
	END
ELSE
	BEGIN
		SET @imposto = @valorPedido * 0.18;
		PRINT 'Imposto Retido (18%): ' + FORMAT(@imposto,'c','pt-br')
	END
---------------------------------------------------------------------------------
DECLARE @salario smallmoney,@reajuste smallmoney,@tempoServico tinyint;
SET @salario = 1500;
SET @tempoServico = 4;
IF @salario <= 2000 AND @tempoServico >= 3
	BEGIN
		SET @reajuste = @salario * 1.1;
		PRINT 'Reajuste de 10% - Novo Salário: ' + FORMAT(@reajuste,'C','pt-br');
	END
ELSE
	BEGIN
		SET @reajuste = @salario * 1.05;
		PRINT 'Reajuste de 5% - Novo Salário: ' + FORMAT(@reajuste,'C','pt-br');
	END
```

## Estrutura de repetição

**WHILE**

O *WHILE* é utilizado para repetir instruções que estão dentro do seu bloco, enquanto a condição de verificação for verdadeira:

```sql
WHILE CondicaoVerdadeira
	BEGIN
		Comandos
	END
```

Exemplos:

```sql
-- Repetição (WHILE)
DECLARE @cont1 int = 1
WHILE @cont1 <= 100
	BEGIN
		PRINT 'Valor do contador: ' + CAST(@cont1 AS varchar)
		SET @cont1 += 1
	END

-- Encadeando Loops
DECLARE @t int = 1,@n int;
WHILE @t <= 10
	BEGIN
		PRINT 'Tabuada do ' + CAST(@t AS VARCHAR(2));
		PRINT '';
		SET @n = 1;
		WHILE @n <= 10
			BEGIN
				PRINT	CAST(@t AS VARCHAR(2)) + ' X ' +
						CAST(@n AS VARCHAR(2)) + ' = ' +
						CAST(@t * @n AS VARCHAR(3));
				SET @n += 1;
			END
		SET @t += 1;
	END

-- CONTINUE
DECLARE @dezena int, @cont int = 1;
IF OBJECT_ID('MegaSena') IS NOT NULL DROP TABLE MegaSena;   

CREATE TABLE MegaSena(dezena int);   

WHILE @cont <= 6   
	BEGIN   
		SET @dezena = 1 + 60 * RAND();   
		
		IF EXISTS(SELECT * FROM MegaSena WHERE dezena = @dezena)
			CONTINUE;
			
		INSERT INTO MegaSena VALUES (@dezena);  
		SET @cont += 1;   
	END 

SELECT * FROM MegaSena ORDER BY 1;
```

## Funções de usuário

Além das funções nativas do SQL Server (Built-in), podemos criar nossas próprias funções.  
Existem dois tipos de função personalizada ou de usuário:

- Escalar: São aquelas em que o retorno de dados se dá em somente um único registro.  
- Tabular: São aquelas em que o retorno de dados se dá em um conjunto de registros.  

>Funções precisam retornar algum valor.

Sintaxe Função Escalar
```sql
CREATE FUNCTION NomeFuncao ([Parametros])
RETURNS TipoDeDado
AS
	Comandos
	RETURN Valor
```

Sintaxe Função Tabular
```sql
CREATE FUNCTION NomeFuncao ([Parametros])
RETURNS TABLE
AS
	RETURN InstrucaoSelect
```

Exemplos:

```sql
-- Criando Funções Escalares
GO
CREATE FUNCTION fn_Maior(@n1 int,@n2 int)
RETURNS int
AS
	BEGIN
		DECLARE @ret int
		IF @n1 > @n2
			SET @ret = @n1
		ELSE
			SET @ret = @n2
	
		RETURN @ret
	END
GO
-- Utilizando nossa função
-- Ao chamar a função é obrigatório especificar OWNER (dbo)
SELECT dbo.fn_Maior(2,5);

GO
CREATE FUNCTION fn_AdicionarData2(@dt date,@op char(1),@n int)
RETURNS date
WITH ENCRYPTION
AS
	BEGIN
		DECLARE @ret date
		SET @ret = 
			CASE UPPER(@op)
				WHEN 'D'	THEN DATEADD(DAY,@n,@dt)
				WHEN 'S'	THEN DATEADD(WEEK,@n,@dt)
				WHEN 'M'	THEN DATEADD(MONTH,@n,@dt)
				WHEN 'Q'	THEN DATEADD(QUARTER,@n,@dt)
				WHEN 'A'	THEN DATEADD(YEAR,@n,@dt)
			END
		RETURN @ret
	END
GO
exec sp_helptext fn_AdicionarData2
SELECT dbo.fn_AdicionarData(GETDATE(),'d',20);
SELECT dbo.fn_AdicionarData(GETDATE(),'s',6);
SELECT dbo.fn_AdicionarData(GETDATE(),'m',3);
SELECT dbo.fn_AdicionarData(GETDATE(),'q',2);
SELECT dbo.fn_AdicionarData(GETDATE(),'a',5);

-- Visualizar o código da function (SEM OWNER)
EXEC sp_help fn_Maior;

SELECT
	T2.name,T1.text
FROM SYSCOMMENTS T1,SYSOBJECTS T2
WHERE T2.xtype = 'FN' AND T2.uid = 1 AND T1.id = T2.id


-- Funções Tabulares
-- Banco Seguro Veículo
USE SeguroVeiculo;
GO
CREATE FUNCTION fn_Apolices(@ano int)
RETURNS table
AS
RETURN
(SELECT        
Clientes.nomeSegurado, Clientes.cadastroSegurado, Seguradoras.nomeSeguradora, 
Apolices.valorApolice, Apolices.dataContratacao, Apolices.prazoMeses, Cidades.nomeCidade
FROM Apolices INNER JOIN
       Cidades ON Apolices.idCidade = Cidades.idCidade INNER JOIN
       Clientes ON Apolices.nContrato = Clientes.nContrato INNER JOIN
       Seguradoras ON Apolices.idSeguradora = Seguradoras.idSeguradora
	   WHERE YEAR(Clientes.cadastroSegurado) = @ano
);
GO

-- Testando a função
SELECT * FROM dbo.fn_Apolices(2013);

```

## Procedures com retorno de dados

Tratam-se de rotinas armazenadas no servidor.  
Uma vez executadas, as instruções da stored procedure, estes comandos ficam armazenados em uma memória chamada de *CACHE*.  
Isso representa um ganho de performance, pois as consultas não precisam mais ser interpretadas, uma vez que a procedure foi criada.  
Causa redução de tráfego de rede em ambientes multiusuário com grande quantidade de utilizadores.  

Exemplos:

```sql
/*
	Procedures: Procedimentos armazenados

	Um procedimento armazenado (Stored Procedure), é uma coleção de instruções
	que, uma vez armazenadas ou salvas, ficam dentro do servidor de forma pré-compilada.

	Vantagens

		. Modularidade: 
		
			Procedimento divido em partes facilitando manutenção de uma aplicação

		· Diminuição de I/O: 
		
			uma vez que é passado parâmetros para o servidor, chamando o procedimento armazenado,
			as operações são executadas utilizando processamento do servidor 
			e no final deste, é retornado ou não os resultados de uma transação, 
			sendo assim, não há um tráfego imenso de dados pela rede

		· Rapidez na execução: 
		
			os stored procedures, após salvos no servidor, ficam somente aguardando,
			já em uma posição da memória cache, serem chamados para executarem uma operação,
			ou seja, como estão pré-compilados, as ações também já estão pré-carregadas, 
			dependendo somente dos valores dos parâmetros.
			Após a primeira execução, elas se tornam ainda mais rápidas (CACHE)

 
		. Encapsulamento:
			
			podemos ocultar complexidades dentro de um servidor, bastante somente aos
			usuários que irão executar o procedimento conhecer seus parâmetros


		· Segurança de dados: 
		
			 podemos bloquear o código de um procedimento (criptografado) com WITH ENCRYPTION
			 e ainda criar limitações de dados aos usuários.

		. Leitura de Dados Dinâmicos

			através de parâmetros fornecidos, o resultado da busca de informações no
			banco poderá se dar de forma dinâmica

		Restrições

		Dentro de procedures não podemos:

			> Utilizar comandos:
				. CREATE PROCEDURE
				. CREATE DEFAULT
				. CREATE RULE
				. CREATE TRIGGER
				. CREATE VIEW

			> Ter mais que 32 níveis de aninhamento

*/

-- Banco SisDep
USE SisDep;

-- Procedure sem parâmetros para retorno de dados
GO
CREATE PROCEDURE spr_DadosFuncionarios
AS
	SELECT
		F.idMatricula,F.NomeFuncionario,D.NomeDepartamento,
		C.NomeCargo,F.Admissao,F.Salario
	FROM Funcionario F
		INNER JOIN Depto D		ON F.idDepartamento = D.idDepartamento
		INNER JOIN Cargo C		ON F.idCargo = C.idCargo;
GO

-- Procedure com parâmetros para retorno de dados
GO
create PROCEDURE spr_AdmissoesPorAno
@inicio int,@fim int
AS
	SELECT
		F.idMatricula,F.NomeFuncionario,D.NomeDepartamento,
		C.NomeCargo,F.Admissao,F.Salario
	FROM Funcionario F
		INNER JOIN Depto D		ON F.idDepartamento = D.idDepartamento
		INNER JOIN Cargo C		ON F.idCargo = C.idCargo
	WHERE YEAR(F.Admissao) BETWEEN @inicio AND @fim;
GO

-- Parâmetros opcionais
CREATE PROCEDURE spr_ParteDoNome
@parteNome varchar(50) = '%'
AS
	SELECT
		F.idMatricula,F.NomeFuncionario,D.NomeDepartamento,
		C.NomeCargo,F.Admissao,F.Salario
	FROM Funcionario F
		INNER JOIN Depto D		ON F.idDepartamento = D.idDepartamento
		INNER JOIN Cargo C		ON F.idCargo = C.idCargo
	WHERE F.NomeFuncionario Like @parteNome
GO

-- Executando procedures

EXECUTE spr_DadosFuncionarios;
EXEC spr_AdmissoesPorAno 2000,2005;
EXEC spr_AdmissoesPorAno 2000;
EXEC spr_ParteDoNome '%Oliveira';
EXEC spr_ParteDoNome '%Silva%';
EXEC spr_ParteDoNome;

-- Executando procedures com parâmetros explícitos
EXEC spr_AdmissoesPorAno  
	@inicio = 2008,
	@fim = 2008;

-- Procedures com retorno
GO
CREATE PROCEDURE spr_PesquisaMatricula
@id int
AS
	IF NOT EXISTS(SELECT * FROM Funcionario WHERE idMatricula = @id)
		RETURN -1

	SELECT
		F.idMatricula,F.NomeFuncionario,D.NomeDepartamento,
		C.NomeCargo,F.Admissao,F.Salario
	FROM Funcionario F
		INNER JOIN Depto D		ON F.idDepartamento = D.idDepartamento
		INNER JOIN Cargo C		ON F.idCargo = C.idCargo
	WHERE F.idMatricula = @id;

EXEC spr_PesquisaMatricula 1025;

EXEC spr_PesquisaMatricula 5000; -- Não conseguimos ver o retorno -1

-- Recuperando retorno
DECLARE @ret int;
EXEC @ret = spr_PesquisaMatricula 5000;
IF @ret < 0 PRINT 'Matrícula Inexistente';

-- Alterando procedure
GO
ALTER PROCEDURE spr_ParteDoNome
@parteNome varchar(50) = '%'
WITH ENCRYPTION
AS
	SELECT
		F.idMatricula,F.NomeFuncionario,D.NomeDepartamento,
		C.NomeCargo,F.Admissao,F.Salario
	FROM Funcionario F
		INNER JOIN Depto D		ON F.idDepartamento = D.idDepartamento
		INNER JOIN Cargo C		ON F.idCargo = C.idCargo
	WHERE F.NomeFuncionario Like @parteNome;

-- Visualizando código de uma procedure
EXEC sp_helptext spr_AdmissoesPorAno;

EXEC sp_helptext spr_ParteDoNome; -- Proc está criptografada

-- Excluir Procedure
DROP PROCEDURE spr_AdmissoesPorAno;
```


## Procedures de manipulação de dados

Além de retornar dados, também é possível criarmos procedures para manipular dados através destes comandos: *ISNERT, UPDATE, DELETE*.  

```sql
-- Procedure de Atualização de Dados
USE SisDep;
GO
CREATE PROCEDURE spr_ReajusteSalarial
@matricula int
AS
	UPDATE Funcionario
	SET Salario *= 1.1
	OUTPUT
		deleted.idMatricula,
		deleted.NomeFuncionario,
		deleted.Salario AS [Salário Anterior],
		inserted.Salario AS [Novo Salário]
	WHERE idMatricula = @matricula;

-- Testar Procedure
EXEC spr_ReajusteSalarial 1025;
-------------------------------------------------------------------------------
-- Tabela Localidade possui campo IDENTITY no idLocalidade
GO
CREATE PROCEDURE spr_NovaLocalidade
@cidade nvarchar(50),@uf nchar(2)
AS
	INSERT INTO Localidade
	(Cidade,Uf)
	VALUES
	(@cidade,@uf)

-------------------------------------------------------------------------------
GO
Use SysConVendas;
GO
CREATE PROCEDURE spr_ExcluirPedido
@pedido int
AS
	DELETE FROM Dados
	WHERE Pedido = @pedido
```

## Triggers

São rotinas especiais com instruções a serem executadas mediante a um evento.  
Em bancos de dados, são chamadas de "gatilhos". Eles permanecem monitorando ações dentro de um servidor.  
Sua grande vantatem está na automatização de processos.  

**Tipos de Triggers**

- DDL (Comandos de definição de dados)
*CREATE - ALTER - DROP*

- DML (Comandos de manipulação de dados)
*INSERT - UPDATE - DELETE*

- Logon (Eventos de servidor)

Exemplos:

```sql
/*
	Triggers - O que são?

	O termo trigger (gatilho em inglês) define uma estrutura do banco de dados que funciona,
	como o nome sugere, como uma função que é disparada mediante alguma ação.

	Tipos de Triggers

	. DDL (Definição de Dados)
	. DML (Manipulação de Dados)
*/

-- Trigger Nível Servidor
GO
CREATE TRIGGER tr_LogServer
ON ALL SERVER
FOR CREATE_DATABASE,DROP_DATABASE,ALTER_DATABASE,DDL_DATABASE_LEVEL_EVENTS --Todos os eventos DDL
AS
	DECLARE @dados XML;
	SET @dados = EVENTDATA();
	SELECT @dados;
GO
-- Testando Trigger
CREATE DATABASE TESTE;

DROP DATABASE TESTE;

-- Visualizando triggers do servidor
SELECT * FROM SYS.server_triggers;

SELECT * FROM sys.server_trigger_events;

-- Visualizando triggers do banco
SELECT * FROM SYS.triggers;

-- Visualizando somente os eventos de trigger no banco
SELECT * FROM SYS.trigger_events;

-- Visualizando todos os eventos possíveis de triggers
SELECT * FROM SYS.trigger_event_types;

-- Triggers DML
CREATE TABLE Historico_AlteraSalario
(
	idMatricula			int,
	salarioAnterior		money,
	salarioNovo			money,	
	dataAlteracao		date,
	usuario				nvarchar(50)
);

GO
CREATE TRIGGER tr_HistAlteraSalario
ON Funcionario
FOR UPDATE
AS
	DECLARE @id int,@salarioAnterior money,@salarioNovo money,@dataAlt date,@user nvarchar(50)

	SELECT @id = idMatricula FROM inserted;
	SELECT @salarioAnterior = Salario FROM deleted;
	SELECT @salarioNovo = Salario FROM inserted;
	IF @salarioAnterior <> @salarioNovo
		INSERT INTO Historico_AlteraSalario
		(idMatricula,salarioAnterior,salarioNovo,dataAlteracao,usuario)
		VALUES
		(@id,@salarioAnterior,@salarioNovo,GETDATE(),SUSER_NAME()) 

-- Testando a Triggers
BEGIN TRAN;
	UPDATE Funcionario WITH (TABLOCK)
	SET Salario += 100
	WHERE idMatricula = 1010;
COMMIT;

SELECT * FROM Historico_AlteraSalario;

SELECT * FROM Funcionario WHERE salario <= 1500

-- Alterando uma Trigger
GO
ALTER TRIGGER tr_HistAlteraSalario
ON Funcionario
WITH ENCRYPTION
FOR UPDATE
AS
	DECLARE @id int,@salarioAnterior money,@salarioNovo money,@dataAlt date,@user nvarchar(50)

	SELECT @id = idMatricula FROM inserted;
	SELECT @salarioAnterior = Salario FROM deleted;
	SELECT @salarioNovo = Salario FROM inserted;
	IF @salarioAnterior <> @salarioNovo
		INSERT INTO Historico_AlteraSalario
		(idMatricula,salarioAnterior,salarioNovo,dataAlteracao,usuario)
		VALUES
		(@id,@salarioAnterior,@salarioNovo,GETDATE(),SUSER_NAME()); 

-- Testando após alteração
EXEC sp_helptext tr_HistAlteraSalario;

-- Excluindo uma trigger
DROP TRIGGER tr_HistAlteraSalario;

-- Desabilitando uma trigger
DISABLE TRIGGER tr_HistAlteraSalario
ON Funcionario;

-- Habilitando uma trigger
ENABLE TRIGGER tr_HistAlteraSalario
ON Funcionario;
```
