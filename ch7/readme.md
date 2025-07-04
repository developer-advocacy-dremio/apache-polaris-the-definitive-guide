# Chapter 7 - Using Apache Polaris with Spark


## Complete Code Snippet for Section 7.1

```py
import pyspark
from pyspark.sql import SparkSession

# Define sensitive variables
POLARIS_URI = 'http://polaris:8181/api/catalog'
POLARIS_CATALOG_NAME = 'polariscatalog'
POLARIS_CREDENTIALS = '<principal_clientId>:<principal_clientSecret>'
POLARIS_SCOPE = 'PRINCIPAL_ROLE:ALL'

# Set up Spark configuration
conf = (
    pyspark.SparkConf()
        .setAppName('PolarisSparkApp')
        # Add necessary JARs
        .set('spark.jars.packages', 'org.apache.iceberg:iceberg-spark-runtime-3.5_2.12:1.5.2,org.apache.hadoop:hadoop-aws:3.4.0')
        # Enable Iceberg SQL extensions
        .set('spark.sql.extensions', 'org.apache.iceberg.spark.extensions.IcebergSparkSessionExtensions')
        # Configure the Polaris catalog
        .set('spark.sql.catalog.polaris', 'org.apache.iceberg.spark.SparkCatalog')
        .set('spark.sql.catalog.polaris.warehouse', POLARIS_CATALOG_NAME)  # Adjust based on your setup
        .set('spark.sql.catalog.polaris.catalog-impl', 'org.apache.iceberg.rest.RESTCatalog')
        .set('spark.sql.catalog.polaris.uri', POLARIS_URI)
        .set('spark.sql.catalog.polaris.credential', POLARIS_CREDENTIALS)
        .set('spark.sql.catalog.polaris.scope', POLARIS_SCOPE)       
        .set('spark.sql.catalog.polaris.token-refresh-enabled', 'true')
)

# Start the Spark session
spark = SparkSession.builder.config(conf=conf).getOrCreate()
print("Spark session configured for Polaris is running.")

# Test the connection
print("Listing all available namespaces:")
spark.sql("SHOW NAMESPACES IN polaris").show()

# Create a new namespace
spark.sql("CREATE NAMESPACE IF NOT EXISTS polaris.db").show()

# List tables in a namespace
spark.sql("SHOW TABLES IN polaris.db").show()
```