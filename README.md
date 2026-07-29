# ENEM Spark Study

Estudo de processamento distribuído e análise de dados com PySpark a partir dos
microdados do ENEM 2025. O notebook explora a qualidade dos dados, transforma os
registros seguindo a arquitetura medalhão e analisa indicadores educacionais do
estado da Paraíba.

## O que este projeto faz

- lê os microdados brutos do ENEM 2025 em CSV;
- verifica schema, valores nulos e abstenções;
- filtra e transforma os registros de escolas da Paraíba;
- calcula nota média e faixa de desempenho dos candidatos;
- salva a camada Silver em Parquet, particionada por município de prova;
- analisa desempenho, presença, dependência administrativa e localização das
  escolas;
- discute escalabilidade, armazenamento e performance no Spark.

## Tecnologias

- Python 3.13
- Apache Spark / PySpark
- Jupyter Notebook
- Pandas e PyArrow
- [uv](https://docs.astral.sh/uv/) para gerenciamento do Python e das dependências

## Estrutura do projeto

```text
enem-spark-study/
├── data/
│   ├── bronze/                # CSV original do INEP
│   └── silver/                # dados transformados em Parquet
├── notebooks/
│   └── analysis.ipynb         # processamento e análises
├── Dicionário_Microdados_Enem_2025.xlsx
├── pyproject.toml
└── uv.lock
```

As pastas `data/bronze` e `data/silver` e seus arquivos não são versionados no
Git. Cada pessoa que executar o projeto deve criá-las localmente e baixar os
microdados.

## Como executar do zero

### 1. Instale os pré-requisitos

Você precisará ter instalado:

- [Git](https://git-scm.com/downloads);
- [uv](https://docs.astral.sh/uv/getting-started/installation/);
- Java JDK 17 ou superior, com a variável de ambiente `JAVA_HOME` configurada.

O Java é necessário porque o PySpark executa sobre a JVM. Verifique as
instalações com:

```bash
git --version
uv --version
java -version
```

### 2. Clone o repositório

```bash
git clone https://github.com/iagonmic/enem-spark-study.git
cd enem-spark-study
```

### 3. Instale o Python e as dependências

Na raiz do projeto, execute:

```bash
uv sync
```

O `uv` usará os arquivos `.python-version`, `pyproject.toml` e `uv.lock` para
instalar a versão correta do Python, criar o ambiente virtual `.venv` e
reproduzir as dependências usadas no estudo.

Não é necessário ativar o ambiente virtual manualmente: os próximos comandos
podem ser executados com `uv run`.

### 4. Crie as camadas de dados

No Linux ou macOS:

```bash
mkdir -p data/bronze data/silver
```

No PowerShell:

```powershell
New-Item -ItemType Directory -Force data/bronze, data/silver
```

### 5. Baixe e organize os microdados

Baixe o arquivo oficial do INEP:

**[Microdados do ENEM 2025 (ZIP)](https://download.inep.gov.br/microdados/microdados_enem_2025.zip)**

Depois:

1. extraia o arquivo ZIP;
2. localize `RESULTADOS_2025.csv` no conteúdo extraído;
3. copie esse arquivo para `data/bronze/RESULTADOS_2025.csv`.

Este estudo usa especificamente o arquivo **`RESULTADOS_2025.csv`**. Antes de
abrir o notebook, a estrutura deve estar assim:

```text
data/
├── bronze/
│   └── RESULTADOS_2025.csv
└── silver/
```

O CSV é grande e pode exigir alguns gigabytes livres em disco. O arquivo
`Dicionário_Microdados_Enem_2025.xlsx`, já incluído no repositório, descreve as
colunas e os códigos presentes nos microdados.

### 6. Abra o notebook

Na raiz do projeto, execute:

```bash
uv run jupyter notebook notebooks/analysis.ipynb
```

Execute as células em ordem. Durante o processamento, o notebook lê a camada
Bronze e grava os dados transformados em:

```text
data/silver/municipio_prova/
```

Essa saída usa o formato Parquet e é particionada pela coluna
`NO_MUNICIPIO_PROVA`. A escrita está configurada com modo `overwrite`, portanto
uma nova execução substitui a saída Silver anterior.

## Solução de problemas

### `JAVA_HOME` não está configurado

Confirme que um JDK 17 ou superior está instalado e que `JAVA_HOME` aponta para
a pasta do JDK. Feche e abra o terminal novamente após alterar a variável.

### Erros do Hadoop no Windows

Se o Spark apresentar erros relacionados ao Hadoop, à biblioteca nativa ou ao
`winutils.exe`, crie a pasta necessária no PowerShell:

```powershell
New-Item -ItemType Directory -Force C:\hadoop\bin
```

Baixe os arquivos `hadoop.dll` e `winutils.exe` da pasta
[hadoop-3.3.6/bin do repositório winutils](https://github.com/cdarlint/winutils/tree/master/hadoop-3.3.6/bin)
e coloque ambos em:

```text
C:\hadoop\bin\
```

A versão 3.3.6 do Hadoop é compatível com o Spark 4.2.0 utilizado neste estudo.
Em seguida, nas variáveis de ambiente do Windows:

1. crie a variável `HADOOP_HOME` com o valor `C:\hadoop`;
2. adicione `%HADOOP_HOME%\bin` à variável `Path`;
3. feche e abra novamente o terminal e o Jupyter para aplicar as alterações.

Para conferir a configuração em um novo PowerShell, execute:

```powershell
winutils.exe
```

### `Path does not exist: .../data/bronze/RESULTADOS_2025.csv`

Confira o nome e a localização do arquivo. O caminho esperado é exatamente:

```text
data/bronze/RESULTADOS_2025.csv
```

### Kernel ou dependências não aparecem no Jupyter

Feche o Jupyter, sincronize novamente o ambiente e reabra o notebook:

```bash
uv sync
uv run jupyter notebook notebooks/analysis.ipynb
```

## Licença

Distribuído sob a [GNU General Public License v3.0](LICENSE).
