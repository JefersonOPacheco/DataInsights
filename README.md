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

## Criação da View para Indicadores
Após a configuração do banco de dados, tabelas e a inserção dos dados, o próximo passo foi a criação de uma view. Essa view, nomeada folha_mensal_view, serve como um recurso essencial para facilitar a construção de indicadores de RH.

### Query para Criação da View
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

## Visualização de Dados com Power BI
Após a preparação e importação dos dados para o banco rh_analytics, o próximo passo foi a visualização desses dados usando o Power BI. Utilizei a view folha_mensal_view para criar diversos dashboards e relatórios.

### Conexão ao Banco de Dados
O Power BI oferece uma forma intuitiva de se conectar ao MySQL. Selecionei o banco de dados rh_analytics e especificamente a view folha_mensal_view para começar a criar as visualizações.

### Dashboards e Relatórios
Neste link você consegue visualizar o Dashboard por completo:

[https://app.powerbi.com/view?r=eyJrIjoiNzliZDYxOWYtMDdhNS00NzkyLTliZDUtZTM5MDNkZWE1MmFjIiwidCI6IjM5MDViZGRjLWU4NWMtNGMwNC1hOWJmLTRkMDgwZWQ1MjZmZCJ9](url)

<div align="center">
<img src = "https://github.com/JefersonOPacheco/DataInsights/assets/151678235/07cc4acd-059d-4caf-b7c5-f702476639c2" width="1000px" />
</div>

### Overview
- Quantidade de admissões (admitidos no período).
- Quantidade de demissões (demitidos no período).
- Média salárial e Headcount ativo.
- Total de má contratções e percentual de má contratações (má contratação são os colaboradores contratados e desligado por iniciativa do empregado ou empregador antés dos 90 dias de experiência).
- Quantidade de admissões e demissões por mês.
- Média da Taxa de turnover.
- Headcount por faixa etária.
- Quantidade e percentual de Headcount por gênero.

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
O projeto demonstrou como uma abordagem orientada a dados pode revolucionar as operações de RH. Desde a estruturação de um banco de dados relacional até a visualização interativa dos KPIs em Power BI, conseguimos transformar dados brutos em insights valiosos. Esses insights não apenas ajudam na gestão eficaz do headcount, mas também fornecem uma base sólida para decisões estratégicas em Recursos Humanos. Portanto, a implementação de People Analytics na empresa GourmetGo não é apenas um avanço tecnológico, mas um passo significativo em direção a uma gestão de RH mais informado e eficaz.
