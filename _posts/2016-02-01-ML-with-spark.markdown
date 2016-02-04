---
layout: post
title:  "Machine Learning with Spark"
date:   2016-02-03
categories: machinelearning
---
I've been using spark for machine learning recently and pleasantly found it could simplify a lot of routines, such as data preprocessing. Here I am going to train a simple classifier from scratch. 

Data Preprocessing
=========
The input training file is a tsv with the first column as class label. We first transform the data into the a `Vectors.dense`, and then `LabeledPoint` that is used by spark. You may also use other [MLlib](http://spark.apache.org/docs/latest/mllib-guide.html) format such as `Vectors.sparse`. 

{%highlight bash%}
1	4121	177	78
1	16492	1148	53
1	852	31	17
1	720	21	11
0	720	177	70
0	772	169	104
0	772	28	18
0	772	30	21
0	1058	25	12
0	41933	812	213
{%endhighlight%}

{%highlight scala%}

import org.apache.spark.mllib.tree.RandomForest
import org.apache.spark.mllib.util.MLUtils
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf
import collection.mutable.HashMap
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.linalg.{Vector, Vectors}
object Feature {

  def main(args: Array[String]): Unit = {
	val sc = new SparkContext(new SparkConf().setAppName("Feature Extraction").setMaster("local"))
	val input = args(0)
	val output = args(1)
	val data = sc.textFile(args(0)).map{
			 line =>
			 	val fields = line.split("\t")  	
			 	
			 	// create a dense vector
			 	val dense = Vectors.dense(fields(1).toDouble, fields(2).toDouble, fields(3).toDouble)
			 	
			 	// form a labeled point
			 	LabeledPoint(fields(0).toDouble, dense)
		 	}
		 	.repartition(1)
	MLUtils.saveLabeledData(data, args(1))
  }
}

{%endhighlight%}

Train Model
========
Now we use MLlib to train a random forest model.
{%highlight scala%}
import org.apache.spark.mllib.tree.RandomForest
import org.apache.spark.mllib.util.MLUtils
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf
import org.apache.spark.mllib.feature.StandardScaler


object Train {

  def main(args: Array[String]): Unit = {
    
    val sc = new SparkContext(new SparkConf().setAppName("Train Model").setMaster("local[1]")) // For local test
    
    // Load and parse the data file.
    val data = MLUtils.loadLabeledData(sc, args(0))
    
    // Split the data into training and test sets (30% held out for testing)
    val splits = data.randomSplit(Array(0.7, 0.3))
    val (trainingData, testData) = (splits(0), splits(1))
    val scaler = new StandardScaler().fit(data.map(trainingData => trainingData.features))
    
    // Train a RandomForest model.
    // Empty categoricalFeaturesInfo indicates all features are continuous.
    val numClasses = 2
    val categoricalFeaturesInfo = Map[Int, Int]()
    val numTrees = 10 // Use more in practice.
    val featureSubsetStrategy = "auto" // Let the algorithm choose.
    val impurity = "gini"
    val maxDepth = 5
    val maxBins = 32
    
    val model = RandomForest.trainClassifier(trainingData, numClasses, categoricalFeaturesInfo,
      numTrees, featureSubsetStrategy, impurity, maxDepth, maxBins)
      
    // Evaluate model on test instances 
        
    val data1 = testData.map(x => (x.label, scaler.transform(x.features)))  
	  
    val labelAndPreds = data1.map { point =>
	    val prediction = model.predict(point._2)
	      (point._1, prediction)
	    }
    
    val testErr = labelAndPreds.filter(r => r._1 != r._2).count.toDouble / data1.count()
    
    
    val tp = labelAndPreds.filter(r=> r._1 == r._2 && r._1==1).coalesce(1).count.toDouble
    val tn = labelAndPreds.filter(r=> r._1 == r._2 && r._1==0).coalesce(1).count.toDouble
    val fp = labelAndPreds.filter(r=> r._1== 0 && r._2 == 1).coalesce(1).count.toDouble
    val fn = labelAndPreds.filter(r=> r._1== 1 && r._2 == 0).coalesce(1).count.toDouble
    
    val prec = tp /(tp + fp)
    val rec = tp/(tp + fn)
    
    println("-----------------")
    println("Learned classification forest model:\n" + model.toDebugString)
    println("Test Error = " + testErr)
    println("Prec = " + prec+"\tRecall = "+ rec)
    println("total data "+ data.count +"\ttp " + tp+" tn "+ tn +" fp " + fp +" fn " + fn)
    println("-----------------")
    
    
    // Save the model and scalar
    sc.parallelize(Seq(scaler), 1).saveAsObjectFile(args(1) + "scaler.model")
    sc.parallelize(Seq(model), 1).saveAsObjectFile(args(1) + "myRandomForestClassificationModel")
  }
}
{%endhighlight%}

Make Predictions
=======
Now we load new data, and apply our model on them. Make sure you use the scaler to transform the data.
{%highlight scala%}
import org.apache.spark.mllib.tree.RandomForest
import org.apache.spark.mllib.util.MLUtils
import org.apache.spark.SparkContext
import org.apache.spark.SparkContext._
import org.apache.spark.SparkConf
import org.apache.spark.mllib.regression.LabeledPoint
import org.apache.spark.mllib.linalg.{Vector, Vectors}
import org.apache.spark.mllib.feature.StandardScaler
import org.apache.spark.mllib.feature.StandardScalerModel

object Pred {

  def main(args: Array[String]): Unit = {
		 
	val sc = new SparkContext(new SparkConf().setAppName("Prediction").setMaster("local"))
		 					
	val testData = sc.textFile(args(0)).map{ 
			line =>
				val fields = line.split("\t")
				val dense = Vectors.dense(fields(0).toDouble, fields(1).toDouble, fields(2)toDouble)
				(LabeledPoint(0, dense), line)
			 					
		 	}
		 			
	// Load model and scaler
	val model = sc.objectFile[ org.apache.spark.mllib.tree.model.RandomForestModel](args(1) + "myRandomForestClassificationModel").first()
	val scaler = sc.objectFile[StandardScalerModel](args(1) + "scaler.model").first()
		 
	println("------------ Model Loaded ----------\n"+model.toDebugString)
		 
	val labelAndPreds = testData.map { 
			e =>
		   		val pred = model.predict(scaler.transform(e._1.features))
		   		(pred, e._2)
		 	}
		
	val out = labelAndPreds.map(e => e._1 + "\t" + e._2)
			.repartition(1)
			.saveAsTextFile(args(1) + "pred")
  }
		 
}

{%endhighlight%}

Done! We have just built a Hello World classifier. You can see that the process is fairly simple. Of course you would need some more work to transform your data into a tsv in the first place if it is not structured yet, thanks to spark the process could be much simpler than Java Hadoop application. Performance tuning will definitely be involved if you are working on large scaled data. 
