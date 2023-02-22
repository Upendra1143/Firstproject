My Sample data: https://www.dropbox.com/sh/uqr26kza6vmxzy8/AAAuy_DzT0OVF1BxPjJz5qoFa?dl=1

Download dataset: https://www.diva-gis.org/gdata

2.	Download couple of countries data.
India, US, Vietnam etc ( 5 Countries data)

3.	Keep this data in HDFS/Local

4.	Load the data into the Spark program
https://databricks.com/notebooks/geospark-notebook.html
https://databricks.com/blog/2019/12/05/processing-geospatial-data-at-scale-with-databricks.html


build.sbt
scalaVersion := "2.11.8"
libraryDependencies += "org.datasyslab" % "geospark" % "1.3.1"
libraryDependencies += "org.datasyslab" % "geospark-sql_2.3" % "1.3.1"
libraryDependencies += "org.apache.spark" %% "spark-core" % "2.4.0"
libraryDependencies += "org.apache.spark" %% "spark-hive" % "2.4.0"
libraryDependencies += "mysql" % "mysql-connector-java" % "8.0.27"

Write Your Spark Program.
import org.apache.spark.sql.SparkSession
import org.datasyslab.geospark.formatMapper.shapefileParser.ShapefileReader
import org.datasyslab.geosparksql.utils.{Adapter, GeoSparkSQLRegistrator}

object ProcessData {
  def main(args: Array[String]): Unit = {

    val spark = SparkSession
      .builder()
      .master("local")
      .appName("SparkAndHive")
      .config("spark.sql.warehouse.dir", "/tmp/spark-warehouse2")
      .enableHiveSupport()
      .getOrCreate()

    GeoSparkSQLRegistrator.registerAll(spark.sqlContext)

    
    val spatialRDD = ShapefileReader.readToGeometryRDD(spark.sparkContext, "src/main/resources/IND_rds")

    val rawSpatialDf = Adapter.toDf(spatialRDD,spark)
    rawSpatialDf.createOrReplaceTempView("rawSpatialDf")

    spark.sql("select * from rawSpatialDf").show

    //cache to speed up queries
    //rawSpatialDf.repartition(spark.sparkContext.defaultParallelism).cache.count
  }
}






1.Write a Program to read all the subfolders of 
a.	roads(rds) of all the country into a single rdd or dataframe 

b.	rails(rrd) of all the country into a single rdd or dataframe 
and write the data into MySQL DB
You should read all the folders and subfolders, Group all the data by the type and push the data to your Database.
2. Once you push the data to DB. Calculate the total count of roads, railways that have been captured in the dataset, country wise.

Select country_code,count(*) from roads group by country_code;


3.Your Database should be created Automatically.
                Maps_9_oct_2022_18_08_00 
1.	Roads table
2.	Railroads table
Refer this to know how to create database programmatically in Java.
https://kodejava.org/creating-mysql-database-programmatically-in-java/
