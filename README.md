# Análise de Dados em People Analytics com MySQL e Power BI

**Autor:** Jeferson Oliveira Pacheco  
**Data:** 17 de outubro de 2023

## Índice
1. [Introdução](#introdução)
2. [Contexto da Empresa](#contexto-da-empresa)
3. [Problema de Negócio](#problema-de-negócio)
4. [Objetivos](#objetivos)
5. [Dados Analisados](#dados-analisados)
6. [Ferramentas Utilizadas](#ferramentas-utilizadas)
7. [Desenvolvimento](#desenvolvimento)
8. [Visualização de Dados com Power BI](#visualização-de-dados-com-power-bi)
9. [Considerações Finais](#considerações-finais)

## Introdução
Em um mundo movido a dados, a importância de People Analytics no RH é destacada neste projeto aplicado na GourmetGo, uma empresa fictícia de fast food. Utilizando MySQL para gestão de dados e Power BI para visualizações, abordamos o ciclo completo dos dados, desde a criação de um banco de dados relacional até o desenvolvimento de KPIs focados em headcount.

## Contexto da Empresa
A GourmetGo, inaugurada em 2022, enfrentou crescimento acelerado, ressaltando a necessidade de um sistema robusto de gerenciamento de dados para entender as dinâmicas operacionais e gerir eficazmente os recursos humanos.

## Problema de Negócio
A GourmetGo precisava armazenar informações dos colaboradores eficientemente e rastrear KPIs relacionados ao headcount, sendo crucial para uma gestão eficaz dos recursos humanos.

## Objetivos
- **Criar um Banco de Dados Relacional:** Desenvolvimento de um banco de dados para gerenciar informações dos colaboradores.
- **Estruturar as Tabelas:** Implementação de uma estrutura de tabelas otimizada para análises complexas.
- **Criar Indicadores de Acompanhamento Mensal:** Desenvolvimento de KPIs centrados em headcount.

## Dados Analisados
Utilização de dados fictícios da GourmetGo, organizados em tabelas inter-relacionadas, formando um modelo lógico coerente para facilitar consultas e análises.

<div align="center">
<img src = "https://github.com/JefersonOPacheco/DataInsights/assets/151678235/6f9c4fee-da49-489c-bd8c-b398c2885448" width="1000px" />
</div>

## Ferramentas Utilizadas
- **MySQL:** Para armazenamento e gestão de dados.
- **Python:** Para importação de dados.
- **Power BI:** Para criação de visualizações impactantes.

## Desenvolvimento
### Configuração Inicial no MySQL
Criação de um banco de dados chamado `rh_analytics` e estruturação das tabelas necessárias, abrangendo diversos aspectos da gestão de RH.
``` SQL
CREATE DATABASE rh_analytics
DEFAULT CHARACTER SET utf8
DEFAULT COLLATE utf8_general_ci;

```
### Estruturação das Tabelas:
Após o banco de dados estar pronto, começamos a estruturar as tabelas necessárias. Elas abrangem diversos aspectos, como informações dos estados, empresas, locais de trabalho, cargos, cores de pele, vínculos empregatícios, situações de trabalho e detalhes de contratos.
### Query de Criação das Tabela
-  **Contratos**
>Nesta Tabela
``` SQL
CREATE TABLE IF NOT EXISTS contratos(
    matricula INT NOT NULL AUTO_INCREMENT,
    nome_completo VARCHAR(80),
    data_nascimento DATE,
    data_admissao DATE,
    data_rescisao DATE,
    data_ultima_promocao DATE,
    id_local INT,
    id_situacao INT,
    id_vinculo INT,
    id_cargo INT,
    id_sexo INT,
    id_cor_pele INT,
    salario DECIMAL(6,2),
    PRIMARY KEY(matricula),
    FOREIGN KEY(id_local) REFERENCES locais(id_local),
    FOREIGN KEY(id_situacao) REFERENCES situacao(id_situacao),
    FOREIGN KEY(id_vinculo) REFERENCES vinculo(id_vinculo),
    FOREIGN KEY(id_cargo) REFERENCES cargos(id_cargo),
    FOREIGN KEY(id_sexo) REFERENCES sexo(id_sexo),
    FOREIGN KEY(id_cor_pele) REFERENCES cores_pele(id_cor_pele)
) DEFAULT CHARSET=utf8;
```
-  **Empresas**
``` SQL
CREATE TABLE IF NOT EXISTS empresas(
	id_empresa INT NOT NULL AUTO_INCREMENT,
	nome_empresa VARCHAR(40),
	PRIMARY KEY(id_empresa)
) DEFAULT CHARSET=utf8;
```
-  **Estados**
``` SQL
CREATE TABLE IF NOT EXISTS estados(
	id_estado INT NOT NULL AUTO_INCREMENT,
	nome_estado VARCHAR(20),
	sigla_estado CHAR(2),
	regiao VARCHAR(20),
	PRIMARY KEY(id_estado)
) DEFAULT CHARSET=utf8;
```
-  **Locais**
``` SQL
CREATE TABLE IF NOT EXISTS locais(
	id_local INT NOT NULL AUTO_INCREMENT,
	id_empresa INT,
	local VARCHAR(50),
	centro_custo VARCHAR(30),
	id_estado INT,
	PRIMARY KEY(id_local),
	FOREIGN KEY(id_empresa) REFERENCES empresas(id_empresa),
	FOREIGN KEY(id_estado) REFERENCES estados(id_estado)
) DEFAULT CHARSET=utf8;
```
-  **Situacao**
``` SQL
CREATE TABLE IF NOT EXISTS situacao(
	id_situacao INT NOT NULL AUTO_INCREMENT,
	desc_situacao VARCHAR(40),
	PRIMARY KEY(id_situacao)
) DEFAULT CHARSET=utf8;
```
-  **Cores_pele**
``` SQL
CREATE TABLE IF NOT EXISTS cores_pele(
	id_cor_pele INT NOT NULL AUTO_INCREMENT,
	nome_cor_pele VARCHAR(30),
	PRIMARY KEY(id_cor_pele)
) DEFAULT CHARSET=utf8;
```
-  **Vinculo**
``` SQL
CREATE TABLE IF NOT EXISTS vinculo(
	id_vinculo INT NOT NULL AUTO_INCREMENT,
	desc_vinculo VARCHAR(50),
	PRIMARY KEY(id_vinculo)

) DEFAULT CHARSET=utf8;
```
-  **Cargos**
``` SQL
CREATE TABLE IF NOT EXISTS cargos(
	id_cargo INT NOT NULL AUTO_INCREMENT,
	nome_cargo VARCHAR(40),
	nivel_cargo VARCHAR(20),
	PRIMARY KEY(id_cargo)
) DEFAULT CHARSET=utf8;
```
-  **Sexo**
``` SQL
CREATE TABLE IF NOT EXISTS sexo(
	id_sexo INT NOT NULL AUTO_INCREMENT,
	sexo ENUM('M', 'F'),
	desc_sexo VARCHAR(20),
	PRIMARY KEY(id_sexo)
) DEFAULT CHARSET=utf8;
```

### Importação de Dados com Python
Importação de dados de arquivo excel para o banco de dados usando Python e Pandas.

O código Python abaixo ilustra o processo de importação dos dados na tabela contratos:
``` Python
import pandas as pd
import pymysql

# Carregar o DataFrame do Excel
df = pd.read_excel(r"Contratados.xlsx")

# Renomear as colunas para corresponder às colunas da tabela MySQL
df.rename(columns={
    'Nome': 'nome_completo',
    'DATA_NASCIMENTO': 'data_nascimento',
    # ... Outros campos renomeados
}, inplace=True)

# Conectar ao banco de dados MySQL
conn = pymysql.connect(
    host='',  # Host
    user='',  # Usuário
    password='',  # Senha
    db='rh_analytics'  # Nome do banco
)

cursor = conn.cursor()

# Inserir os dados em pedaços
chunk_size = 500
for i in range(0, len(df), chunk_size):
    df_chunk = df.iloc[i:i + chunk_size]
    
    for index, row in df_chunk.iterrows():
        sql = """INSERT INTO contratos (
            nome_completo, data_nascimento,
            # ... Outros campos
        ) VALUES (%s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s, %s)"""
        
        cursor.execute(sql, tuple(row))
    
    # Commit para salvar as alterações
    conn.commit()

# Fechar a conexão
cursor.close()
conn.close()
```

## Criação da View para Indicadores
Após a configuração do banco de dados, tabelas e a inserção dos dados, o próximo passo foi a criação de uma view. Essa view, nomeada folha_mensal_view, serve como um recurso essencial para facilitar a construção de indicadores de RH.

A query SQL a seguir cria a view, fazendo junções com várias tabelas e calculando campos derivados, como idade e faixa etária dos colaboradores:

``` SQL
CREATE VIEW folha_mensal_view AS
SELECT
	ct.matricula AS 'Matricula',
	UPPER(ct.nome_completo) AS 'Nome Completo',
	ct.data_nascimento 'Data de Nascimento',
	FLOOR(DATEDIFF(CURRENT_DATE, ct.data_nascimento) / 365.25) AS Idade,
    	CASE
		WHEN FLOOR(DATEDIFF(CURRENT_DATE, ct.data_nascimento) / 365.25) BETWEEN 20 AND 24 THEN '20-24'
	        WHEN FLOOR(DATEDIFF(CURRENT_DATE, ct.data_nascimento) / 365.25) BETWEEN 25 AND 29 THEN '25-29'
	        WHEN FLOOR(DATEDIFF(CURRENT_DATE, ct.data_nascimento) / 365.25) BETWEEN 30 AND 34 THEN '30-34'
	        WHEN FLOOR(DATEDIFF(CURRENT_DATE, ct.data_nascimento) / 365.25) BETWEEN 35 AND 39 THEN '35-39'
	        WHEN FLOOR(DATEDIFF(CURRENT_DATE, ct.data_nascimento) / 365.25) BETWEEN 40 AND 44 THEN '40-44'
	        WHEN FLOOR(DATEDIFF(CURRENT_DATE, ct.data_nascimento) / 365.25) BETWEEN 45 AND 49 THEN '45-49'
        	ELSE '50+'
	END AS 'Faixa Etaria',
	ct.data_admissao AS 'Data de Admissao',
	ct.data_rescisao AS 'Data de Rescisao',
	ct.data_ultima_promocao AS 'Data da ultima Promocao',
	loc.id_local AS 'id Local',
	emp.id_empresa AS 'id Empresa',
	sit.id_situacao AS 'id Situacao',
	vic.id_vinculo AS 'id Vinculo',
	car.id_cargo AS 'id Cargo',
	cp.id_cor_pele AS 'id Cor Pele',
	sx.id_sexo AS 'id Sexo',
	ct.salario AS 'Salario'
FROM contratos ct
LEFT JOIN locais loc ON ct.id_local = loc.id_local
LEFT JOIN empresas emp ON loc.id_empresa = emp.id_empresa
LEFT JOIN situacao sit ON sit.id_situacao = ct.id_situacao
LEFT JOIN vinculo vic ON ct.id_vinculo = vic.id_vinculo
LEFT JOIN cargos car ON ct.id_cargo = car.id_cargo
LEFT JOIN cores_pele cp ON ct.id_cor_pele = cp.id_cor_pele
LEFT JOIN sexo sx ON ct.id_sexo = sx.id_sexo;
```
## Power Query

Tabela **d_calendario** criada em linguagem M
``` M
let
    // Fonte de dados genérica (substitua esta linha pelo nome da tabela atual)
    FonteDados = f_fatos, 

    // Definindo datas mínima e máxima manualmente
    PrimeiraData = #date(2022, 1, 1),
    UltimaData = #date(2023, 10, 19),


    // Criação da lista de datas
		Fonte = List.Dates(PrimeiraData, Number.From(UltimaData) - Number.From(PrimeiraData) + 1 ,#duration(1,0,0,0)),

    // Conversão para tabela e renomeação da coluna
    TabelaDatas = Table.FromList(Fonte, Splitter.SplitByNothing(), null, null, ExtraValues.Error),
    TabelaRenomeada = Table.RenameColumns(TabelaDatas,{{"Column1", "Data"}}),
    
    // Transformar coluna "Data" em tipo data
    TabelaTipos = Table.TransformColumnTypes(TabelaRenomeada,{{"Data", type date}}),

    // Adicionando colunas auxiliares
    NomesMesesAbreviados = {"Jan", "Fev", "Mar", "Abr", "Mai", "Jun", "Jul", "Ago", "Set", "Out", "Nov", "Dez"},
    TabelaFinal = 
        Table.AddColumn(
            Table.AddColumn(
                Table.AddColumn(
                    Table.AddColumn(
                        Table.AddColumn(
                            Table.AddColumn(
                                Table.AddColumn(
                                    Table.AddColumn(
                                        Table.AddColumn(TabelaTipos, "Ano", each Date.Year([Data]), Int64.Type), 
                                        "Mês", each Date.Month([Data]), Int64.Type),
                                    "Dia", each Date.Day([Data]), Int64.Type),
                                "Nome do Mês", each NomesMesesAbreviados{[Mês]-1}, type text),
                            "Dia da Semana nome", each Date.DayOfWeekName([Data]), type text),
                        "Dia da Semana", each Date.DayOfWeek([Data]), Int64.Type),
                    "Número da Semana do Ano", each Date.WeekOfYear([Data]), Int64.Type),
                "Trimestre", each Number.RoundDown((Date.Month([Data])-1)/3) + 1, Int64.Type),
            "Nome do Trimestre", each Text.From([Trimestre]) & " Tri", type text)
in
    TabelaFinal
```

**Modelo de Relacionamentos**
<div align="center">
<img src = "https://github.com/JefersonOPacheco/DataInsights/assets/151678235/db9a82e5-8649-4796-b162-918ae0e92fb5" width="1000px" />
</div>

## Visualização de Dados com Power BI
Criação de dashboards e relatórios para visualizar e analisar os dados, focando em KPIs como quantidade de admissões/demissões, média salarial, headcount ativo, turnover, entre outros.

<div align="center">
<img src = "https://github.com/JefersonOPacheco/DataInsights/assets/151678235/07cc4acd-059d-4caf-b7c5-f702476639c2" width="1000px" />
</div>

### Overview
- Quantidade de admissões (admitidos no período).
``` DAX
Admissoes = 

CALCULATE(
    COUNTROWS(f_fatos),
    USERELATIONSHIP(d_calendario[Data], f_fatos[Data de Admissao])
)
```
- Quantidade de demissões (demitidos no período).
``` DAX
Demissoes = 

CALCULATE(
    COUNTROWS(f_fatos),
    f_fatos[id Situacao] = 2, --Situação 2 é usada para funcionarios demitidos
    USERELATIONSHIP(d_calendario[Data], f_fatos[Data de Rescisao])
)
```
- Média salárial e Headcount ativo.
``` DAX
MediaSalarial = 

CALCULATE(
    AVERAGE(f_fatos[Salario]),
    f_fatos[id Situacao] <> 2 --Diferente da situação 2, que se refere aos demitidos
)
```
- Total de má contratções e percentual de má contratações (má contratação são os colaboradores contratados e desligado por iniciativa do empregado ou empregador antés dos 90 dias de experiência).
``` DAX
MaContracoes = 

CALCULATE(
    [Admissoes],
    FILTER(
        f_fatos,
        DATEDIFF(f_fatos[Data de Admissao], f_fatos[Data de Rescisao], DAY) < 90 && 
        f_fatos[Data de Rescisao] <> BLANK() &&
        f_fatos[id Vinculo] <> 2
    )
)
```
- Quantidade de admissões e demissões por mês.
``` DAX

```
- Média da Taxa de turnover.
``` DAX
Media_Turnover_Mensal =
 
AVERAGEX(
    VALUES(d_calendario[Nome do Mês]),
    DIVIDE(
        ([Admissoes] + [Demissoes]) / 2,
        [Headcount]
    )
)
```
- Headcount por faixa etária.
> Uso a mesma dax de headcount, dessa vez no eixo Y coloco a faixa etária

- Quantidade e percentual de Headcount por gênero.
> Uso a mesma dax de headcount, dessa vez no eixo Y coloco o gênero

<div align="center">
<img src = "https://github.com/JefersonOPacheco/DataInsights/assets/151678235/b8779940-28fe-4b06-b5b7-5bbeb404583e" width="1000px" />
</div>

### Headcount
- Headcount por mensal.
- Tunover mensal.
- Headcount por centro de custo.

<div align="center">
<img src = "https://github.com/JefersonOPacheco/DataInsights/assets/151678235/7181835c-9e08-4573-abb3-dd2364fb7786" width="1000px" />
</div>

### Detalhamento
- Consegue visualizar no detalhe todas as informações em um formato de matriz.

## Considerações Finais
Este projeto destacou como a análise de dados pode transformar o RH, transformando dados brutos em insights valiosos para decisões estratégicas, marcando um passo significativo em direção a uma gestão de RH mais informada e eficaz.
