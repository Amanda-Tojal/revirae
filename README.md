# Análises para o Projeto Revirae  

Este repositório contém scripts e instruções para análises de resistência antimicrobiana no âmbito do Projeto Revirae.  

## Objetivo  
Realizar análises para identificar padrões de resistência antimicrobiana em amostras, utilizando dados do CZID e gerando visualizações como dendogramas e gráficos de barra.  

## Requisitos  
Certifique-se de ter os seguintes itens instalados antes de executar os scripts:  
- **R** (com os pacotes necessários especificados nos scripts `.R`)  
- **Python 3** (com Jupyter Notebook e bibliotecas como `pandas`, `seaborn`, entre outras)  
- Editor de planilhas (como Excel ou Google Sheets)  

---

## Instruções 

## 1.Análises de Qualidade
### Passo 1: Download do Arquivo
Faça o download do arquivo `Sample Overview (sample QC metrics)` a partir do **CZID**.

### Passo 2: Execute o arquivo Numero_reads.R
Será gerado como resultado um gráfico com o número total de reads e o número que passou pelo filtro de qualidade.

---

## 2.Organismos detectados
### Passo 1: Download do Arquivo 
Faça o download do arquivo `Combined Sample Taxon Results (NT r total reads)` a partir do **CZID**.

### Passo 2: Execute o arquivo 01_top-generation.R com as devidas alterações
Ao final da execução, será gerado o arquivo `new_output.csv`, utilizado nas próximas etapas.

### Passo 3: Execute o arquivo 02_pieplot_relatorio.R com as devidas alterações
Ao final da execução, serão gerados gráficos para cada uma das amostras analisadas.

### Passo 4: Edições no Arquivo
Faça as alterações necessárias no arquivo `new_output.csv` para incluir informações de tempo, estado, local, protocolo...

### Passo 5: Execute o arquivo 03_graficos_por_especie_patogenica.R com as devidas alterações


---

## 3.Análises de Resistência Antimicrobiana  
### Passo 1: Download do Arquivo  
Faça o download do arquivo `combined_amr_results.csv` a partir do **CZID**.

### Passo 2: Organização das Colunas  
Use o script `01_organizador.R` para reorganizar as colunas que contêm múltiplos valores, gerando como resultado o arquivo **`drug_class.csv`**.  

### Passo 3: Edições no Arquivo
Faça as alterações necessárias no arquivo `drug_class.csv` para incluir informações de tempo, estado, local, protocolo...
O arquivo pode conter espaçamentos desnecessários no início de alguns valores. Utilize a fórmula abaixo em seu editor de planilhas para corrigir:  
```excel  
=SE(ESQUERDA(A1;1)=" ";DIREITA(A1;NÚM.CARACT(A1)-1);A1)  
```  
Certifique-se de salvar o arquivo corrigido para as etapas seguintes.  




### Passo 4: Geração do Dendrograma  
Execute o script `02_heatmap.ipynb` para criar o dendrograma com base nos dados processados.  
- **Entrada**: Arquivo CSV ajustado.  
- **Saída**: Dendrograma visualizando os padrões de resistência.  

### Passo 5: Criação de Gráficos de Barra  
Utilize o script `03_graficos_por_especie_patogenica.R` para gerar gráficos de barra agrupados por amostra e espécie patogênica.  
- **Entrada**: Arquivo processado na etapa anterior.  
- **Saída**: Gráficos detalhados por espécie patogênica.  

---

## Estrutura do Repositório  

```plaintext  
|-- 01_organizador.R          # Script para organizar colunas com múltiplos valores.  
|-- 02_heatmap.ipynb          # Notebook para gerar o dendrograma.  
|-- 03_graficos_por_especie_patogenica.R  # Script para criar gráficos de barra.  
|-- combined_amr_results.csv  # Arquivo de entrada (não incluso, baixe do CZID).  
```  

---
***

## BANCO DE DADOS
O script da criação do banco de dados com o nome 'acmelab_amb' 
```
CREATE DATABASE `acmelab_amb` /*!40100 DEFAULT CHARACTER SET utf8mb4 COLLATE utf8mb4_0900_ai_ci */ /*!80016 DEFAULT ENCRYPTION='N' */;
```
Após a criação do database foi gerado no total 5 tabelas, a qual essas tabelas são nomeadas de 'ngs' 'amr', 'czid', 'amostra' e 'corrida', sendo ligações entre tabelas de N para 1. A seguir segue o script de criação de cada tabela e o tipo de ligação entre elas. 
```
# criaçao da tabela 'ngs' com duas chaves estrageiras 
-- acmelab_amb.ngs definition

CREATE TABLE `ngs` (
  `ID_NGS` int NOT NULL AUTO_INCREMENT,
  `ID_AMOSTRA` int DEFAULT NULL,
  `ID_CORRIDA` int NOT NULL,
  `SAMPLE_NAME` text NOT NULL,
  `PROTOCOL` text NOT NULL,
  `TOTAL_READS` bigint NOT NULL,
  `PASSED_FILTERS_CZID` bigint NOT NULL,
  `PASSED_FILTERS_PERCENT_CZID` float NOT NULL,
  PRIMARY KEY (`ID_NGS`),
  KEY `ID_AMOSTRA` (`ID_AMOSTRA`),
  KEY `ID_CORRIDA` (`ID_CORRIDA`),
  CONSTRAINT `ngs_ibfk_1` FOREIGN KEY (`ID_AMOSTRA`) REFERENCES `amostra` (`ID_AMOSTRA`) ON DELETE CASCADE,
  CONSTRAINT `ngs_ibfk_2` FOREIGN KEY (`ID_CORRIDA`) REFERENCES `corrida` (`ID_CORRIDA`) ON DELETE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=159 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

# criaçao da tabela 'amr' com uma chaves estrageiras de N para 1 para 'ngs'
-- acmelab_amb.amr definition

CREATE TABLE `amr` (
  `ID_AMR` int NOT NULL AUTO_INCREMENT,
  `ID_NGS` int DEFAULT NULL,
  `NUM_READS` float NOT NULL,
  `GENE_FAMILY` text NOT NULL,
  `DRUG_CLASS` text NOT NULL,
  `RESISTANCE_MECHANISM` text NOT NULL,
  PRIMARY KEY (`ID_AMR`),
  KEY `ID_NGS` (`ID_NGS`),
  CONSTRAINT `amr_ibfk_1` FOREIGN KEY (`ID_NGS`) REFERENCES `ngs` (`ID_NGS`) ON DELETE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=8857 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

# criaçao da tabela 'czid' com uma chaves estrageiras de N para 1 para 'ngs'
-- acmelab_amb.czid definition

CREATE TABLE `czid` (
  `ID_CZID` int NOT NULL AUTO_INCREMENT,
  `ID_NGS` int DEFAULT NULL,
  `ORGANISMO` text NOT NULL,
  `ESKAPE` enum('SIM','NAO') NOT NULL,
  `NUMERO_READS` bigint NOT NULL,
  `RPM` float NOT NULL,
  PRIMARY KEY (`ID_CZID`),
  KEY `ID_NGS` (`ID_NGS`),
  CONSTRAINT `czid_ibfk_1` FOREIGN KEY (`ID_NGS`) REFERENCES `ngs` (`ID_NGS`) ON DELETE CASCADE
) ENGINE=InnoDB AUTO_INCREMENT=3161 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

# criaçao da tabela 'amostra' com uma chaves estrageiras de 1 para N para 'ngs'
-- acmelab_amb.amostra definition

CREATE TABLE `amostra` (
  `ID_AMOSTRA` int NOT NULL AUTO_INCREMENT,
  `CODIGO_INTERNO` text NOT NULL,
  `DATA_COLETA` date DEFAULT NULL,
  `SEMANA_COLETA` int DEFAULT NULL,
  `DATA_ENVIO` date DEFAULT NULL,
  `UF` varchar(2) DEFAULT NULL,
  `CIDADE` text,
  `LOCALIDADE` text,
  PRIMARY KEY (`ID_AMOSTRA`)
) ENGINE=InnoDB AUTO_INCREMENT=148 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

# criaçao da tabela 'corrida' com uma chaves estrageiras de 1 para N para 'ngs'
-- acmelab_amb.corrida definition

CREATE TABLE `corrida` (
  `ID_CORRIDA` int NOT NULL AUTO_INCREMENT,
  `NOME_CORRIDA` text,
  `LOTE` int DEFAULT NULL,
  `DATA_SEQUENCIAMENTO` date DEFAULT NULL,
  `ILLUMINA_RUN_ID` text,
  `SEQUENCIADOR` enum('Miseq','Nextseq2000') DEFAULT NULL,
  `Q30` float DEFAULT NULL,
  `PF` float DEFAULT NULL,
  PRIMARY KEY (`ID_CORRIDA`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;
```


