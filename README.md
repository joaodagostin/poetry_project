```markdown
# Comandos Utilizados no Projeto

## 1. Instalação do Poetry

Para instalar o Poetry, caso ainda não tenha feito isso, execute o seguinte comando:

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

Para conferir a versão instalada do Poetry, use:

```bash
poetry --version
```

Para configurar o ambiente virtual e baixar as dependências:

```bash
poetry install
``` 

Para ativar o ambiente virtual (caso esteja utilizando a versão 2.0 ou superior):

```bash
poetry env activate
```

## 2. Instalando Dependências com Poetry

Para adicionar pacotes ao projeto, execute:

```bash
poetry add pyspark@3.5.0 delta-spark@3.0.0 ipykernel jupyterlab
```

## 3. Configuração do Hadoop no Windows

Adicione estas linhas ao seu script Python (antes de criar a sessão Spark) ou no terminal:

```python
import os
os.environ['HADOOP_HOME'] = 'C:/hadoop'
os.environ['PATH'] = os.environ['HADOOP_HOME'] + '/bin;' + os.environ['PATH']
```

## 4. Configuração do Spark com Delta

Para inicializar o Spark com o Delta, configure a `SparkSession` com as seguintes opções:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("Exemplo Delta Lake") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()
```

Crie um DataFrame de exemplo:

```python
dados = [("João", 25), ("Maria", 30), ("José", 35)]
colunas = ["Nome", "Idade"]
df = spark.createDataFrame(dados, colunas)
```

Para salvar em formato Delta:

```python
df.write.format("delta").mode("overwrite").save("C:/tmp/delta-teste")
```

## 5. Configuração do Hadoop com Winutils

Baixe o `winutils.exe` para o diretório `C:/hadoop/bin`:

```bash
curl -L https://github.com/cdarlint/winutils/raw/master/hadoop-3.3.6/bin/winutils.exe -o /c/hadoop/bin/winutils.exe
```

Para configurar o caminho no terminal:

```bash
set HADOOP_HOME=C:/hadoop
set PATH=%HADOOP_HOME%/bin;%PATH%
```

## 6. Comandos para Diagnóstico

Verifique a versão do Python:

```bash
python --version
```

Verifique a versão do PySpark (no Python):

```python
import pyspark
print(pyspark.__version__)
```

Verifique a versão do Delta (no Python):

```python
import delta
print(delta.__version__)
```

## 7. Solução de Problemas

Caso enfrente o erro `java.lang.UnsatisfiedLinkError`, verifique se as variáveis de ambiente `HADOOP_HOME` e `PATH` estão configuradas corretamente.

## 8. Comandos Úteis no Ambiente de Desenvolvimento

Ativar o ambiente do Poetry:

```bash
poetry env activate
```

Para rodar o Jupyter Lab:

```bash
poetry run jupyter lab
```

Instalar as dependências:

```bash
poetry install
```

Adicionar pacotes:

```bash
poetry add <package_name>
```

## 9. Preparação da Sessão Spark

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("DeltaLakeTest")
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension")
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")
    .config("spark.hadoop.fs.file.impl", "org.apache.hadoop.fs.LocalFileSystem")  # Correção importante aqui
    .getOrCreate()
)
```

## 10. Criando uma Tabela Delta

Exemplo de dados para o DataFrame:

```python
dados = [("João", 25), ("Maria", 30), ("José", 35)]
colunas = ["Nome", "Idade"]

# Criando o DataFrame
df = spark.createDataFrame(dados, colunas)

# Salvando a tabela Delta
df.write.format("delta").mode("overwrite").save("/tmp/delta/clientes")
```

## 11. Inserir Dados

Novos dados para inserir:

```python
novos_dados = [("Carlos", 40), ("Ana", 28)]
novos_df = spark.createDataFrame(novos_dados, colunas)

# Inserindo dados na tabela Delta
novos_df.write.format("delta").mode("append").save("/tmp/delta/clientes")
```

## 12. Atualizar Dados

```python
from delta.tables import DeltaTable
from pyspark.sql.functions import col

# Carregar a tabela Delta existente
deltaTable = DeltaTable.forPath(spark, "/tmp/delta/clientes")

# Atualizando os dados
deltaTable.update(
    condition = col("Nome") == "João",  # Condição para a atualização
    set = { "Idade": "29" }  # Novo valor para a coluna
)
```

## 13. Deletar Dados

Removendo registros com base em uma condição:

```python
deltaTable.delete(
    condition = col("Nome") == "José"
)
```

## 14. Verificar os Dados Após as Alterações

```python
# Lendo os dados da tabela Delta
df_resultado = spark.read.format("delta").load("/tmp/delta/clientes")
df_resultado.show()
```
```
