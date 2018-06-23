# Stored Procedures

Um procedimento é um lote de instruções SQL que é armazenado em um banco de dados.

Um procedimento armazenado pode ser executada manualmente ou ser invocada por outros programas.

O uso de procedimento armazenado é muito útil em ambientes cliente/servidor em função de fatores como desempenho e facilidade de manutenção.

Um procedimento armazenado pode fazer uma chamada a outro procedimento armazenado. Ela pode retornar um valor de status indicando se foi executada com sucesso ou não.

Apesar de retornar parâmetros de saída, as procedimento armazenado são diferentes de funções, pois não pode ser usada em expressões.

Pode ser criada com o comando **CREATE PROCEDURE** e alterada com o comando **ALTER PROCEDURE**.

```text
CREATE PROC nome_procedure; número
    [@parâmetros datatype [= default ] [INPUT/OUTPUT]
    [WITH RECOMPILE / ENCRYPTION ...
    [FOR REPLICATION]
    AS instruções_sql
```

É um número inteiro usado para agrupar procedimentos de mesmo nome, assim elas podem ser eliminadas juntas com um único comando **DROP**.

• P\_AlterarSal;1

• P\_AlterarSal;2

• P\_AlterarSal;3

```sql
DROP Procedure P_AlterarSal
```

**Varying**

É um datatype que se aplica a parâmetros cursores.

**Default**

É um valor definido como padrão para um parâmetro de uma procedure.

**Output**

É um parâmetro que indica a procedure retornar um valor, sem utilizar o comando RETURN.

```sql
CREATE PROCEDURE P_Conta
@Num_1 int,
@Num_2 int
@Resposta bigint OUTPUT
AS SET @Resposta = @Num_1 * @Num_2
```

```sql
DECLARE @Resposta bigint
EXEC P_Conta 10,7, @Resposta OUTPUT
Print @Resposta
```

**With Recompile**

Indica que o Sistema não deve manter na memória o plano de execução desta procedure, gerando um novo plano todas as vezes que ela for executada.

**With Encription**

Faz com que o SQL Server criptografe código da procedimento armazenado na tabela do sistema syscomments e a procedure não pode ser publicada durante uma replicação.

**For Replication**

Especifica que a procedimento armazenado não pode ser executada no servidor Subscribe. Pode ser executada como filtro em uma replicação.

![](file:////Users/tell/Library/Group%20Containers/UBF8T346G9.Office/msoclip1/01/clip_image002.png)

Tabela: Funcionario

```sql
CREATE TABLE [dbo].[Funcionario](
            [Cod_Func] [int] NOT NULL,
            [Nome_Func] [varchar](100) NULL,
            [Sexo] [char](1) NULL,
            [Salario] [float] NULL,
            [Data_Ad] [datetime] NULL,
            [Regiao] [int] NULL
)
```

Inserindo dados na tabela

```sql
Insert into Funcionario
Values(1,'Manda Chuva','M',5000,'01/01/1998',1)
Insert into Funcionario
Values(2,'Chuchu','M',3000,'01/01/1999',1)
Insert into Funcionario
Values(3,'Bacana','M',2000,'10/10/2000',2)
Insert into Funcionario
Values(4,'Espeto','M',2500,'02/03/2001',2)
Insert into Funcionario
Values(5,'Batatinha','F',4000,'05/06/2002',3)
```

### Exemplo 1: Sem parâmetro de entrada e sem retorno.

Some o salário de todos os funcionários. Se o valor ultrapassar 5000,00 aplique um aumento salarial de 5% para todos os funcionários; caso contrário, aplique um aumento de 10% para todos os funcionários.

```sql
CREATE PROCEDURE P_AlterarSal;1
AS
DECLARE @Percentual Decimal(4,2)
IF (SELECT SUM(Salario) FROM Funcionario ) >= 5000.00
    SET @Percentual = 1.05
ELSE
    SET @Percentual = 1.10
UPDATE Funcionario SET Salario = Salario * @Percentual
EXECUTE P_AlterarSal;1
```

### Exemplo 2: com parâmetro de entrada.

O objetivo deste procedimento é aumentar o salário, no percentual recebido como parâmetro, do funcionário também recebido como parâmetro.

```sql
CREATE PROCEDURE P_AlterarSal;2
@Codigo int,
@Percentual decimal(4,2) = 1.0
AS
UPDATE Funcionario SET Salario = Salario * @Percentual
            WHERE Cod_Func = @Codigo
EXECUTE P_AlterarSal;2 1, 1.2
EXECUTE P_AlterarSal;2 @Percentual=1.2, @Codigo = 1
EXECUTE P_AlterarSal;2 1
```

Este procedimento tem como objetivo aplicar um aumento salarial para o funcionário recebido como parâmetro e retornar para quem a executa o valor corrigido do salário. Haverá alteração salarial de acordo com o parâmetro @Percentual se o salário corrigido dessa pessoa for menor ou igual a 5000,00.

```sql
CREATE PROCEDURE P_AlterarSal;3
@Codigo int = 0,
@Percentual decimal(4,2) = 1.0
AS
IF @Percentual = 0 OR @Codigo = 0
      RETURN(0)
DECLARE @Salario decimal(10,2)
SELECT @Salario = Salario * @Percentual FROM Funcionario
            WHERE Cod_Func = @Codigo
IF @Salario <= 5000
      UPDATE Funcionario Set Salario = @Salario
                   WHERE Cod_Func = @Codigo
Return (@Salario)
DECLARE @Var int
EXEC @Var = P_AlterarSal;3 2, 1.1
PRINT @Var
DECLARE @Var int
EXEC @Var = P_AlterarSal;3
PRINT @Var
```

Este procedimento tem como objetivo aplicar um aumento salarial para o funcionário recebido como parâmetro e retornar para quem a executa dados do funcionário e um flag que indica se houve alteração no salário ou não. Haverá alteração salarial de acordo com o parâmetro @Percentual se o salário atual dessa pessoa for menor ou igual a 5000,00

```sql
CREATE PROCEDURE P_AlterarSal;4
@Codigo int, @Percentual decimal(4,2)=1.0,
@Salario decimal(10,2) OUTPUT, @Sexo char(01)  OUTPUT,
@Regiao tinyint         OUTPUT,
@Flag     tinyint          OUTPUT
AS
SELECT @Salario = Salario * @Percentual, @Sexo = Sexo,  @Regiao = Regiao
FROM Funcionario  WHERE Cod_Func = @Codigo
IF @Salario <= 5000
    BEGIN
            UPDATE Funcionario SET Salario = @Salario
                         WHERE Cod_Func = @Codigo
                         SET @Flag = 1
END
ElSE
                    SET @Flag = 0
DECLARE @Sal decimal(10,2),
              @Sex char(01),
              @Reg tinyint,
              @Sinal tinyint
EXEC P_AlterarSal;4 1, 1.2, @Sal OUTPUT,
                                        @Sex OUTPUT,
                                        @Reg OUTPUT,
                                        @Sinal OUTPUT
SELECT @Sex, @Reg, @Sinal, @Sal
```

**Raiserror\(\) -** Raiserror é um comando mais poderoso do que o PRINT para enviar mensagens.

Existem duas formas de utilizar este comando:

Enviar uma mensagem avulsa para uma aplicação

```text
RAISERROR( ‘Isto é uma mensagem avulsa’, 16, 1)
```

Para enviar uma mensagem padronizada da tabela sysmessages para uma aplicação, existem duas possibilidades:

1. Enviar uma mensagem já existente na tabela do sistema sysmessages:

```sql
RAISERROR(13003, 16,1)
RAISERROR(14027,16,1’Tab_Teste’)
```

1. Enviar uma mensagem colocada na tabela do sistema sysmessages pelo usuário. O primeiro passo é acrescentar esta mensagem à tabela sysmessages:

```sql
Exec sp_AddMessage 50001, 16, ‘O usuário %s cometeu um erro’
```

Depois execute o RAISERROR:

```sql
RAISERROR(50001, 16,1, ‘José Antônio’)
```

**Importante:**

1. O número atribuído à mensagem de erro tem que ser maior ou igual a 50001.
2. As severidades de 0 a 18 podem ser utilizada por qualquer usuário.
3. As severidades de 19 a 25 podem ser utilizadas apenas pelos administradores do sistema. Essas severidades são consideradas erros fatais. Se ocorrer um erro fatal, a conexão é encerrada, depois que ele receber a mensagem.

**Nota**: Os procedimentos armazenados, sempre melhoram a performance de processamento dos dados do sistema, em comparação com a execução de queries avulsas.

Nesse exercício vamos fazer uma implementação completa de um caso de uso. O caso de uso escolhido, por fins didáticos, foi a manutenção no castrado de clientes.

Nos slides seguintes teremos todos os passos para essa implementação, com diagramas de caso de uso, detalhamento, diagrama de classe, diagrama de seqüência, script do banco de dados e implementação propriamente dita em java.

## Diagrama de Casos de uso

![](file:////Users/tell/Library/Group%20Containers/UBF8T346G9.Office/msoclip1/01/clip_image004.png)

### Detalhamento – CUS01

Caso de Uso: **Cadastrar Cliente**\(CSU01\)

**Sumário**: O usuário utiliza esse caso de uso para fazer o cadastro de novos clientes.

**Ator Principal**: Usuário \(funcionário\)

**Pré-condição**: O cliente não está cadastrado no sistema.

**Fluxo Principal**:

1- O usuário abre o formulário de manutenção do cliente.

2- para inserir um novo cliente o usuário digita as informações do cliente, tais como ID, Nome, SobreNome, Endereço, Cidade, Estado, CEP, IDPaís e Email, e submete ao sistema.

3- O sistema verifica se o cliente é realmente um cliente novo. Se se assim proceder, então o cliente é cadastrado.

4- O sistema envia uma mensagem "Cliente cadastrado com sucesso" e encerra o caso de uso.

**Fluxo Alternativo\(3\):** Se o cliente já estiver cadastrado.

1- O sistema reporta esse fato ao usuário e volta ao passo 2.

### Detalhamento – CUS02

1- O usuário abre o formulário de manutenção do cliente.

2- O usuário seleciona a opção Atualizar Cliente.

3- O usuário digita as informações do cliente, tais como ID, Nome, SobreNome, Endereço, Cidade, Estado, CEP, IDPaís e Email, e submete ao sistema.

4- O sistema executa o procedimento para atualizar o cadastro do cliente. Se não ocorrer erro, então os dados do cliente é atualizado.

5- O sistema envia uma mensagem "Dados do Cliente atualizado com sucesso" e encerra o caso de uso.

Fluxo Alternativo\(4\): Se oorreu erro.

1- O sistema reporta esse fato ao usuário, exibindo o erro ocorrido e volta ao passo 2.

### Detalhamento – CUS03

Caso de Uso: Excluir Cliente\(CSU03\)

Sumário: O usuário utiliza esse caso de uso para excluir um clientes do sistema.

Ator Principal: Usuário \(funcionário\)

Pré-condição: O cliente é cadastrado no sistema.

Fluxo Principal:

1- O usuário abre o formulário de manutenção do cliente.

2- Seleciona a opção Excluir cliente.

3- O usuário digita o ID do cliente a ser excluído e submete ao sistema.

4- O sistema executa o procedimento para excluir cliente. Se se não ocorrer erro, então o cliente é excluído do sistema.

5- O sistema envia uma mensagem "Cliente Excluído do cadastrado com sucesso" e encerra o caso de uso.

Fluxo Alternativo\(5\): Ocorreu erro.

1- O sistema reporta esse fato ao usuário e exibe o erro ocorrido ao usuário e volta ao passo 2

### Detalhamento – CUS04

Caso de Uso: Localizar Cliente\(CSU04\)

Sumário: O usuário utiliza esse caso de uso para localizar um cliente cadastrado no sistema.

Ator Principal: Usuário \(funcionário\)

Pré-condição: O cliente é cadastrado no sistema.

Fluxo Principal:

1- O usuário abre o formulário de manutenção do cliente.

2- Seleciona a opção Localizar cliente.

3- O usuário digita o ID do cliente a ser localizado e submete ao sistema.

4- O sistema executa o procedimento para localizar cliente. Se se não ocorrer erro, então o cliente é localizado e suas informações são exibidas.

5- O Caso de uso encerra.

Fluxo Alternativo\(4\): Ocorreu erro.

1- O sistema reporta esse fato ao usuário e exibe o erro ocorrido ao usuário e volta ao passo 2

Diagrama de Classe

![](file:////Users/tell/Library/Group%20Containers/UBF8T346G9.Office/msoclip1/01/clip_image006.png)

Diagrama de Seqüência – CSU01

![](file:////Users/tell/Library/Group%20Containers/UBF8T346G9.Office/msoclip1/01/clip_image008.png)

Diagrama de Seqüência – CSU02

![](file:////Users/tell/Library/Group%20Containers/UBF8T346G9.Office/msoclip1/01/clip_image010.png)

Diagrama de Seqüência – CSU03

![](file:////Users/tell/Library/Group%20Containers/UBF8T346G9.Office/msoclip1/01/clip_image012.png)

Diagrama de Seqüência – CSU04

![](file:////Users/tell/Library/Group%20Containers/UBF8T346G9.Office/msoclip1/01/clip_image014.png)

### Código Java da Classe Cliente

```java
public class Cliente
{
   private int clienteid;
   private int nome;
   private int sobrenome;
   private int endereco;
   private int cidade;
   private int estado;
   private int cep;
   private int idpais;
   private int email;
public Cliente() { }
   public void InserirCliente() { }
   public void atualizarCliente() { }
   public void ExcluirCliente() { }
   public void LocalizarCliente() { }
}
```

Script do Banco de Dados

```sql
/* Criando a tabela cliente no banco de dados Teste */
IF EXISTS (SELECT name FROM sysobjects WHERE name = 'Cliente' AND Type= 'U')
            DROP TABLE Cliente
Go
CREATE TABLE Cliente (
            ClienteID int not null primary key,
            Nome varchar(35) not null,
            SobreNome varchar(15) not null,
            Endereco varchar(50) not null,
            Cidade varchar(25) not null,
            Estado varchar(2)
                       check (Estado IN ('RN', 'PB', 'CE', 'PE', 'BA', 'MA', 'AL', 'SE')),
            CEP varchar(8) not null,
            IDPais int,
            Email varchar(50))
IF EXISTS (SELECT name FROM sysobjects WHERE name = 'InserirCliente' AND Type= 'P')
            DROP PROC InserirCliente
Go
CREATE PROC InserirCliente
            @ClienteID int, @Nome varchar(35), @SobreNome varchar(15),
            @Endereco varchar(50), @Cidade varchar(25), @Estado varchar(2),
            @CEP varchar(8), @IDPais int, @Email varchar(50)
AS
            INSERT INTO Cliente (ClienteID,Nome,SobreNome,Endereco,Cidade,
            Estado,CEP,IdPais,Email)
VALUES 
            (@ClienteID,@Nome,@SobreNome,@Endereco,@Cidade,@Estado,
            @CEP,@IDPais,@Email)
exec InserirCliente 1,'Larissa', 'Cunha','Rua José Ovídio Vale',
                       'Natal', 'RN', '59015410',55, '@sxp.com.br‘
exec InserirCliente 2,'José Antônio', 'Cunha','Rua José Ovídio Vale',
                       'Natal', 'RN', '59015410',55, '@sxp.com.br'
CREATE PROC AtualizarCliente
            @ClienteID int, @Nome varchar(35), @SobreNome varchar(15),
            @Endereco varchar(50), @Cidade varchar(25), @Estado varchar(2),
            @CEP varchar(8), @IDPais int, @Email varchar(50)
AS
            UPDATE Cliente SET
            Nome = @Nome,
            SobreNome = @SobreNome,
            Endereco = @Endereco,
            Cidade = @Cidade,
            Estado = @Estado,
            CEP = @CEP,
            IdPais = @IdPais,
            Email = @Email
WHERE           ClienteID = @ClienteID
```

```sql
exec AtualizarCliente 1,'Larissa Medeiros', 'Cunha','Rua José Bernardo',  'caicó', 'RN', '59300000',55, '@sxp.com.br'
```

procedimento armazenado - ExcluirCliente

```sql
IF EXISTS (SELECT name FROM sysobjects WHERE name = 'ExcluirCliente' AND Type= 'P')
            DROP PROC ExcluirCliente
Go
CREATE PROC ExcluirCliente
            @ClienteID int
AS
            DELETE FROM Cliente
                   WHERE ClienteID = @ClienteID
EXEC ExcluirCliente 2
procedimento armazenado - LocalizarCliente
IF EXISTS (SELECT name FROM sysobjects WHERE name = 'LocalizarCliente' AND Type= 'P')
            DROP PROC LocalizarCliente
Go
CREATE PROC LocalizarCliente
            @ClienteID int
AS
            SELECT * FROM Cliente
                   WHERE ClienteID = @ClienteID
```

```sql
exec LocalizarCliente 1
```

### Chamando um procedimento armazenado

```java
//Carrega o Drive
Class.forName(“sun.jdbc.odbc.JdbcOdbcDriver”);
...
//Conecta-se ao banco
Connection con = DriverManager.getConnection(“jdbc:odbc:Teste_proc”,”sa”,””);
//Cria e executa uma instrução
CallableStatement cs = con.prepareCall(“{call InserirCliente(?,?,?,?,?,?,?,?,?)}”);
cs.setInt(1, pClienteId);
cs.setString(2, pnome).
...
cs.setString(9,pemail);
cs.executeUpdate();
cs.close();
con.close();
```

### Chamando um procedimento armazenado

```java
//Carrega o Drive
Class.forName(“sun.jdbc.odbc.JdbcOdbcDriver”);
...
//Conecta-se ao banco
Connection con = DriverManager.getConnection(“jdbc:odbc:Teste_proc”,”sa”,””);
//Cria e executa uma instrução
CallableStatement cs = con.prepareCall(“{call LocalizarCliiente(?,)}”);
cs.setInt(1, pClienteId);
ResultSet rs = cs.executeQuery();
...
cs.close();
con.close();
```

### Chamando um procedimento armazenado em C\#\

```csharp
protected void Button1_Click(object sender, EventArgs e)
        {  
            string aSQLProc = "SP_CLIENTE_FIXARPAIS";
            string aSQLConecStr =
 "Data Source=IFRNNB465;Initial Catalog=Ap02C;Persist Security Info=True;User ID=sa;Password=sa";
            // Abrindo a Conexão com o banco de dados
            SqlConnection aSQLCon = new SqlConnection(aSQLConecStr);
            aSQLCon.Open();
            // Executando o comando
            SqlCommand aSQL = new SqlCommand(aSQLProc, aSQLCon);
            aSQL.CommandType = CommandType.StoredProcedure;
            aSQL.ExecuteNonQuery();
        }
```

