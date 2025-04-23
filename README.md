
# Guia de Comandos do Projeto

## 1. Instalando o Poetry

Se ainda não tiver o Poetry instalado, utilize o seguinte comando:

```bash
curl -sSL https://install.python-poetry.org | python3 -
```

Para verificar se a instalação foi concluída corretamente:

```bash
poetry --version
```

Para configurar o ambiente virtual e instalar as dependências:

```bash
poetry install
```

Se estiver utilizando a versão 2.0 ou superior, ative o ambiente com:

```bash
poetry env activate
```

## 2. Adicionando Bibliotecas com Poetry

Para incluir pacotes no seu ambiente, execute:

```bash
poetry add pyspark@3.5.0 delta-spark@3.0.0 ipykernel jupyterlab
```

## 3. Ajustando o Hadoop no Windows

Inclua as variáveis de ambiente no seu código ou terminal antes de criar a `SparkSession`:

```python
import os
os.environ['HADOOP_HOME'] = 'C:/hadoop'
os.environ['PATH'] = os.environ['HADOOP_HOME'] + '/bin;' + os.environ['PATH']
```

## 4. Inicializando o Spark com suporte ao Delta Lake

Configure sua sessão Spark com suporte ao Delta da seguinte forma:

```python
from pyspark.sql import SparkSession

spark = SparkSession.builder \
    .appName("Exemplo Delta Lake") \
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension") \
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog") \
    .getOrCreate()
```

Criando um DataFrame simples:

```python
dados = [("João", 25), ("Maria", 30), ("José", 35)]
colunas = ["Nome", "Idade"]
df = spark.createDataFrame(dados, colunas)
```

Gravando os dados em formato Delta:

```python
df.write.format("delta").mode("overwrite").save("C:/tmp/delta-teste")
```

## 5. Download do Winutils para Hadoop

Faça o download do `winutils.exe` e coloque em `C:/hadoop/bin`:

```bash
curl -L https://github.com/cdarlint/winutils/raw/master/hadoop-3.3.6/bin/winutils.exe -o /c/hadoop/bin/winutils.exe
```

Depois, configure as variáveis no terminal:

```bash
set HADOOP_HOME=C:/hadoop
set PATH=%HADOOP_HOME%/bin;%PATH%
```

## 6. Verificações Rápidas

Para checar as versões instaladas:

Versão do Python:

```bash
python --version
```

Versão do PySpark:

```python
import pyspark
print(pyspark.__version__)
```

Versão do Delta:

```python
import delta
print(delta.__version__)
```

## 7. Dicas para Resolver Problemas

Se aparecer o erro `java.lang.UnsatisfiedLinkError`, confira se `HADOOP_HOME` e `PATH` estão corretamente definidos.

## 8. Comandos Úteis Durante o Desenvolvimento

Ativar o ambiente:

```bash
poetry env activate
```

Rodar o Jupyter Lab:

```bash
poetry run jupyter lab
```

Instalar dependências do projeto:

```bash
poetry install
```

Adicionar novos pacotes:

```bash
poetry add <nome_do_pacote>
```

## 9. Preparando a Sessão Spark

```python
from pyspark.sql import SparkSession

spark = (
    SparkSession.builder
    .appName("DeltaLakeTest")
    .config("spark.sql.extensions", "io.delta.sql.DeltaSparkSessionExtension")
    .config("spark.sql.catalog.spark_catalog", "org.apache.spark.sql.delta.catalog.DeltaCatalog")
    .config("spark.hadoop.fs.file.impl", "org.apache.hadoop.fs.LocalFileSystem")
    .getOrCreate()
)
```

## 10. Criando uma Tabela Delta

Exemplo de criação e salvamento de um DataFrame:

```python
dados = [("João", 25), ("Maria", 30), ("José", 35)]
colunas = ["Nome", "Idade"]

df = spark.createDataFrame(dados, colunas)
df.write.format("delta").mode("overwrite").save("/tmp/delta/clientes")
```

## 11. Inserção de Novos Registros

Adicionando novos dados ao conjunto existente:

```python
novos_dados = [("Carlos", 40), ("Ana", 28)]
novos_df = spark.createDataFrame(novos_dados, colunas)

novos_df.write.format("delta").mode("append").save("/tmp/delta/clientes")
```

## 12. Atualizando Registros

```python
from delta.tables import DeltaTable
from pyspark.sql.functions import col

deltaTable = DeltaTable.forPath(spark, "/tmp/delta/clientes")

deltaTable.update(
    condition = col("Nome") == "João",
    set = { "Idade": "29" }
)
```

## 13. Removendo Registros

Excluindo entradas com base em uma condição:

```python
deltaTable.delete(
    condition = col("Nome") == "José"
)
```

## 14. Consultando os Dados

Exibindo os dados após alterações:

```python
df_resultado = spark.read.format("delta").load("/tmp/delta/clientes")
df_resultado.show()
```
