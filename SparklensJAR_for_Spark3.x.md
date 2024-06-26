#### Problem: 
Latest JARs in Maven Central repo support Spark 2.X and doesn't work with Spark 3.X. Here are modifications you need to make to run on Spark 3.X. 

#### Steps to run Sparklens on Spark 3.X:
1.	Install sbt: 0.13.18
2.	Clone from the repo: qubole/sparklens: Qubole Sparklens tool for performance tuning Apache Spark (github.com)
3.	Change directory: cd sparklens
4. Changing plugins.sbt:
Comment out addSbtPlugin (addSbtPlugin("org.spark-packages" % "sbt-spark-package" % "0.2.4"))
    ```
    addSbtPlugin("com.eed3si9n" % "sbt-assembly" % "0.12.0")
    
    resolvers += "Spark Package Main Repo" at "https://dl.bintray.com/spark-packages/maven"
    
    // addSbtPlugin("org.spark-packages" % "sbt-spark-package" % "0.2.4")
    ```
6. In build.sbt file, comment out spName, sparkVersion, spAppendScalaVersion as they were using the ':=' operator which is used for setting keys in earlier sbt version. Declare these three as variables in this context. Comment out the line where it is using 'sparkVersion.version' to get the version of Spark. However, 'sparkVersion' is a String and does not have a 'version' property. Hence, it has been replaced with 'sparkVersion'. 
      ````
    name := "sparklens"
    organization := "com.qubole"
    
    scalaVersion := "2.12.0"
    
    crossScalaVersions := Seq("2.10.6", "2.12.0")
    
    // spName := "qubole/sparklens"
    
    // sparkVersion := "2.0.0"
    
    // spAppendScalaVersion := true
    
    val spName = "qubole/sparklens"
    
    val sparkVersion = "3.0.0"
    
    val spAppendScalaVersion = true
    
    
    // libraryDependencies += "org.apache.spark" %% "spark-core" % sparkVersion.version % "provided"
    
    libraryDependencies += "org.apache.spark" %% "spark-core" % sparkVersion % "provided"
    
    libraryDependencies += "org.apache.spark" %% "spark-sql" % "3.0.0"
    
    libraryDependencies +=  "org.apache.hadoop" % "hadoop-client" % "2.6.5" % "provided"
    
    libraryDependencies += "org.apache.httpcomponents" % "httpclient" % "4.5.6" % "provided"
    
    libraryDependencies += "org.apache.httpcomponents" % "httpmime" % "4.5.6" % "provided"
    
    test in assembly := {}
    
    testOptions in Test += Tests.Argument("-oF")
    
    scalacOptions ++= Seq("-target:jvm-1.7")
    
    javacOptions ++= Seq("-source", "1.7", "-target", "1.7")
    
    publishMavenStyle := true
    
    
    licenses += ("Apache-2.0", url("http://www.apache.org/licenses/LICENSE-2.0"))
    
    credentials += Credentials(Path.userHome / ".ivy2" / ".sbtcredentials")
    
    
    pomExtra :=
      <url>https://github.com/qubole/sparklens</url>
      <scm>
        <url>git@github.com:qubole/sparklens.git</url>
        <connection>scm:git:git@github.com:qubole/sparklens.git</connection>
      </scm>
      <developers>
        <developer>
          <id>iamrohit</id>
          <name>Rohit Karlupia</name>
          <url>https://github.com/iamrohit</url>
        </developer>
        <developer>
          <id>beriaanirudh</id>
          <name>Anirudh Beria</name>
          <url>https://github.com/beriaanirudh</url>
        </developer>
        <developer>
          <id>mayurdb</id>
          <name>Mayur Bhosale</name>
          <url>https://github.com/mayurdb</url>
        </developer>
      </developers>
    
    ```
7. Change the scala version to 2.12.0 and spark version to make Sparklens work on support 3.X. Add spark-sql 3.0.0 library dependency (libraryDependencies += "org.apache.spark" %% "spark-sql" % "3.0.0")
8. In QuboleJobListener.scala (src\main\scala\com\qubole\sparklens\QuboleJobListener.scala), change attemptId to attemptNumber().
9. In the HDFSConfigHelper.scala (src\main\scala\com\qubole\sparklens\helper\HDFSConfigHelper.scala), SparkHadoopUtil class has been changed to a private class in Spark 3. Modify this as shown below:

      ```
      import org.apache.hadoop.conf.Configuration
      import org.apache.spark.SparkConf
      import org.apache.spark.deploy.SparkHadoopUtil
      import org.apache.spark.sql.SparkSession
      
      object HDFSConfigHelper {
        def getHadoopConf(sparkConfOptional: Option[SparkConf]): Configuration = {
          if (sparkConfOptional.isDefined) {
            val spark = SparkSession.builder.config(sparkConfOptional.get).getOrCreate()
            spark.sparkContext.hadoopConfiguration
          } else {
            val spark = SparkSession.builder.getOrCreate()
            spark.sparkContext.hadoopConfiguration
          }
        }
      }
      ```
 
10. Run "sbt compile" to compile the revised code:
10.Run "sbt package" to package the compiled code to sparklens JAR (target\scala-2.12\sparklens_2.12-0.3.2.jar). 
