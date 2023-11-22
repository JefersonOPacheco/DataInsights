# Análise de Dados em People Analytics com MySQL e Power BI

## Introdução
Em um mundo cada vez mais movido a dados, o campo de Recursos Humanos não pode se dar ao luxo de ficar à margem dessa revolução. People Analytics vai além de ser um termo da moda. É uma ferramenta estratégica que está redefinindo a gestão de talentos. Este artigo se concentra em um caso prático de People Analytics aplicado na GourmetGo, uma empresa fictícia de fast food inaugurada em 2022. Utilizamos MySQL para o armazenamento e gestão de dados. Power BI foi empregado para criar visualizações impactantes. Com essas ferramentas, abordamos todo o ciclo de vida dos dados. Criamos um banco de dados relacional e desenvolvemos indicadores-chave de desempenho (KPIs) centrados em headcount. Esses são componentes vitais para qualquer estratégia bem-sucedida de RH.

## Problema de Negócio
Inaugurada em 2022, a empresa GourmetGo experimentou um crescimento acelerado que destacou a necessidade imediata de um sistema robusto de gerenciamento de dados. O objetivo era não apenas armazenar informações dos colaboradores de forma eficiente, mas também rastrear indicadores-chave de desempenho (KPIs) relacionados ao headcount. Esse acompanhamento é crucial para uma gestão eficaz dos recursos humanos e para entender as dinâmicas operacionais nas diversas filiais da empresa.

## Objetivos
1. **Criar um Banco de Dados Relacional:** Desenvolver um banco de dados relacional que possa acomodar e gerenciar eficientemente as informações dos colaboradores e outros dados relevantes para a empresa GourmetGo.
2. **Estruturar as Tabelas:** Definir e implementar uma estrutura de tabelas otimizada que permita consultas eficazes e possibilite análises complexas, tendo em mente tanto os requisitos atuais quanto as necessidades futuras da empresa.
3. **Criar Indicadores de Acompanhamento Mensal:** Desenvolver uma série de KPIs (Indicadores-Chave de Desempenho) voltados para o headcount e outros aspectos pertinentes aos recursos humanos, permitindo um acompanhamento mensal e fornecendo insights valiosos para a tomada de decisão.

## Dados Analisados
Para a análise, utilizamos um conjunto de dados fictícios que representam informações dos colaboradores da empresa GourmetGo. Esses dados estão organizados em diversas tabelas inter-relacionadas, formando um modelo lógico coerente que facilita consultas e análises. O modelo lógico de relacionamento entre as tabelas é apresentado abaixo.

<div align="center">
<img src = "https://github.com/JefersonOPacheco/DataInsights/assets/151678235/6f9c4fee-da49-489c-bd8c-b398c2885448" width="1000px" />
</div>

## Ferramentas Utilizadas
- MySQL
- Python
- Power BI

## Desenvolvimento
### Configuração Inicial no MySQL
O primeiro passo foi criar um banco de dados específico no MySQL chamado rh_analytics. A query para isso é a seguinte:
``` SQL
CREATE DATABASE rh_analytics
DEFAULT CHARACTER SET utf8
DEFAULT COLLATE utf8_general_ci;

```
### Estruturação das Tabelas:
Após o banco de dados estar pronto, começamos a estruturar as tabelas necessárias. Elas abrangem diversos aspectos, como informações dos estados, empresas, locais de trabalho, cargos, cores de pele, vínculos empregatícios, situações de trabalho e detalhes de contratos.
### Query de Criação da Tabela Contratos
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

## Importação de Dados com Python
O próximo passo após a criação do banco de dados e das tabelas é a inserção dos dados. Utilizei Python e a biblioteca Pandas para importar dados de um arquivo Excel e inseri-los na tabela contratos.

### Código para Importação de Dados
O código Python abaixo ilustra o processo de importação dos dados:
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
