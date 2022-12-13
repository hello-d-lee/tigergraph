# In the Google Cloud platform, navigate to Vertex AI Workbench to managed notebooks and create a new notebook. Launch the new notebook.

# you'll want to select either PySpark local kernel or Serverless Spark. install required dependencies:
```python
!pip install pyTigerGraph --user --quiet
!pip install pyspark[sql] --user --quiet
```

# import libraries
```python
import pyTigerGraph as tg
import json
import pandas as pd
import getpass
```

# specify credentials for your tigergraph instance
```python
host = "https://ce.i.tgcloud.io"
graphname = "social"
username = [your username]
version = "3.0.5" #@param ["3.0.5", "3.0.0", "2.6.2"] {allow-input: true}
useCert = True #@param {type:"boolean"}
```

# enter the password and secrets
```python
password = getpass.getpass()
secret = getpass.getpass()
```

# create the connection
```python
conn = tg.TigerGraphConnection(host=host, graphname=graphname, username=username, password=password, gsqlSecret=secret)
token = conn.getToken(secret, setToken=True)
print(token)
```

# now get some data
```python
def getLoadedStats(limit=5):
  numPeople = conn.getVertexCount("person")

  people = conn.getVertices("person", limit=limit)

  print(f"The re are currently {numPeople} people, edges connecting them")
  print(f"Our people are: {json.dumps(people, indent=2)}")

getLoadedStats()
```

# your results may be different depending on what you are trying to do
```python
results = conn.getVerticesById("person", "Tom")
print(json.dumps(results, indent=2))
```

# now use pyspark to read the data into a spark dataframe
# Import the required classes and functions
```python
from pyspark.sql import SparkSession, Row

# Initialize a SparkSession
spark = SparkSession.builder.appName("tigergraph-solution").getOrCreate()

# Convert the query result to a list of Row objects
rows = [Row(**x) for x in results]

# Create a DataFrame from the list of Row objects
df = spark.createDataFrame(rows)

# Print the schema of the DataFrame
df.printSchema()

print(df)

# you can also check the schema this way to check the shape of the data
print(df.schema)

# and look at the dataframe
df.select("*").show()

# next you'll want to output this dataframe to a json file - can either write to local directory or to a google cloud storage bucket
# local directory
df.write.json("./test1.json")

# google cloud storage bucket - change to your unique name
df.write.json("gs://tigergraph-bq/test1.json")
```

# now go to your cloud storage bucket, you should see a new folder created and the json files inside 
