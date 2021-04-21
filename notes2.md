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