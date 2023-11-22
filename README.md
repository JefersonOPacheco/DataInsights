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
-  **contratos**
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
-  **empresas**
``` SQL
CREATE TABLE IF NOT EXISTS empresas(

) DEFAULT CHARSET=utf8;
```
-  **locais**
``` SQL
CREATE TABLE IF NOT EXISTS locais(

) DEFAULT CHARSET=utf8;
```
-  **estados**
``` SQL
CREATE TABLE IF NOT EXISTS estados(

) DEFAULT CHARSET=utf8;
```
-  **situacao**
``` SQL
CREATE TABLE IF NOT EXISTS situacao(

) DEFAULT CHARSET=utf8;
```
-  **cores_pele**
``` SQL
CREATE TABLE IF NOT EXISTS cores_pele(

) DEFAULT CHARSET=utf8;
```
-  **vinculo**
``` SQL
CREATE TABLE IF NOT EXISTS vinculo(

) DEFAULT CHARSET=utf8;
```
-  **cargos**
``` SQL
CREATE TABLE IF NOT EXISTS cargos(

) DEFAULT CHARSET=utf8;
```
-  **sexo**
``` SQL
CREATE TABLE IF NOT EXISTS sexo(

) DEFAULT CHARSET=utf8;
```

## Importação de Dados com Python
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

## Visualização de Dados com Power BI
Criação de dashboards e relatórios para visualizar e analisar os dados, focando em KPIs como quantidade de admissões/demissões, média salarial, headcount ativo, turnover, entre outros.

## Considerações Finais
Este projeto destacou como a análise de dados pode transformar o RH, transformando dados brutos em insights valiosos para decisões estratégicas, marcando um passo significativo em direção a uma gestão de RH mais informada e eficaz.
