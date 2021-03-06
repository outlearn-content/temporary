<!--
{
"name" : "lab3",
"version" : "0.1",
"title" : "Lab 3: Pig - Risk Factor",
"description" : "Use Pig to compute Driver Risk Factor",
"freshnessDate" : 2015-07-23,
"homepage" : "http://hortonworks.com/",
"canonicalSource" : "http://hortonworks.com/hadoop-tutorial/hello-world-an-introduction-to-hadoop-hcatalog-hive-and-pig/#section_9",
"license" : "All Rights Reserved"
}
-->

<!-- @section -->

## Use Pig to compute Driver Risk Factor

#### **Introduction:**

In this tutorial you will be introduced to Apache Pig. In the earlier section of lab you learned how to load data into HDFS and then manipulate it using Hive. We are using the Truck sensor data to better understand risk associated with every driver. This section will teach you to compute risk using Apache Pig.

**Prerequisite:**

The tutorial is a part of series of hands on tutorial to get you started on HDP using Hortonworks sandbox. Please ensure you complete the prerequisites before proceeding with this tutorial.

*   Step-0 (Hortonworks sandbox set up)
*   Lab1: Loading sensor data into HDFS
*   Lab2: Data Manipulation with Apache Hive
*   Allow yourself around one hour to complete this tutorial.

**Outline:**

*   Pig basics
*   Step-3.1: Define Table schema
*   Step-3.2: Create Pig Script
*   Step-3.3: Quick Recap
*   Step-3.4: Execute Pig Script on Tez
*   Suggested readings

**Pig Basics:**

Pig is a high level scripting language that is used with Apache Hadoop. Pig enables data workers to write complex data transformations without knowing Java. Pig’s simple SQL-like scripting language is called Pig Latin, and appeals to developers already familiar with scripting languages and SQL.

Pig is complete, so you can do all required data manipulations in Apache Hadoop with Pig. Through the User Defined Functions(UDF) facility in Pig, Pig can invoke code in many languages like JRuby, Jython and Java. You can also embed Pig scripts in other languages. The result is that you can use Pig as a component to build larger and more complex applications that tackle real business problems.

Pig works with data from many sources, including structured and unstructured data, and store the results into the Hadoop Data File System.

Pig scripts are translated into a series of MapReduce jobs that are run on the Apache Hadoop cluster.

<!-- @section -->

## Step 3.1: Define table schema

Now we have refined the truck data to get the average mpg for each truck. The next task is to compute the risk factor for each driver which is the total miles driven/abnormal events. We can get the event information from the geolocation table.

![Lab3_1](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_1.png)

If we look at the truck_mileage table, we we have the driverid and the number of miles for each trip. To get the total miles for each driver, we can group those records by driverid and then sum the miles.

1.  We will start by creating a table named driver_mileage that is created from a query of the columns we want from truck_mileage. The following query groups the records by driverid and sums the miles in the select statement. Execute this query in a new Worksheet:

— Create table DriverMileage from existing truck_mileage data

```sql
CREATE TABLE DriverMileage
STORED AS ORC
AS
SELECT driverid, sum(miles) totmiles
FROM truck_mileage
GROUP BY driverid;
```

2\. View the data generated by the script by clicking the **Load sample data** icon in the Database Explorer next to drivermileage. The results should look like:

![Lab3_2](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_2.png)

3\. Next, you will use Pig to compute the risk factor of each driver. Before we can run the Pig code, one of the requirements for the HCatStorer() class is that the table must already exist in Hive. The Pig code expects the following structure for a table named riskfactor. Execute the following DDL command:

— Create table avg_mileage from existing trucks_mileage data

```sql
CREATE TABLE riskfactor (driverid string,events bigint,totmiles bigint,riskfactor float)STORED AS ORC;
```

4\. Verify the riskfactor table was created successfully. It will be empty now, but you will populate it from a Pig script. You are now ready to compute the risk factor using Pig. Let’s take a look at Pig and how to execute Pig scripts from within Ambari.

<!-- @task, "text" : "Complete Step 3.1."-->

<!-- @section -->

## Step 3.2: Create Pig Script

In this tutorial we create and run a Pig script. We will use the Ambari Pig User View. Let’s get started…

****a. Log in to Ambari Pig User Views****

To get to the Ambari Pig User View, click on the User Views icon at top right and select **Pig**:

![Screen-Shot-2015-07-21-at-10.12.41-AM](http://hortonworks.com/wp-content/uploads/2015/07/Screen-Shot-2015-07-21-at-10.12.41-AM.png)

This will bring up the Ambari Pig User View interface. Your Pig View does not have any scripts to display, so it will look like the following:

![Lab3_4](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_4.png)

On the left is a list of your scripts, and on the right is a composition box for writing scripts. A special feature of the interface is the Pig helper at the bottom. The Pig helper will provide us with templates for the statements, functions, I/O statements, HCatLoader() and Python user defined functions. At the very bottom are status areas that will show the results of our script and log files.

The following screenshot shows and describes the various components and features of the Pig User View:

![Lab3_5](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_5.png)

#### **b. Create a New Script**

Let’s enter a Pig script. Click the **New Script** button in the upper-right corner of the view:

![Lab3_6](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_6.png)

Name the script **riskfactor.pig**, then click the **Create** button:

![Lab3_7](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_7.png)

#### **c. Load Data in Pig using Hcatalog**

We are going to use HCatalog to load data into Pig. HCatalog allows us to share schema across tools and users within our Hadoop environment. It also allows us to factor out schema and location information from our queries and scripts and centralize them in a common repository. Since it is in HCatalog we can use the HCatLoader() function. Pig makes it easy by allowing us to give the table a name or alias and not have to worry about allocating space and defining the structure. We just have to worry about how we are processing the table.

*   We can use the Pig helper at the bottom of the screen to give us a template for the line. Click on **Pig helper -> HCatalog->load template**
*   The entry **%TABLE%** is highlighted in red for us. Type the name of the table which is geolocation.
*   Remember to add the **a =** before the template. This saves the results into a. Note the **‘=’** has to have a space before and after it.
*   Our completed line of code will look like:

```
a = LOAD 'geolocation' using org.apache.hive.hcatalog.pig.HCatLoader();
```

Copy-and-paste the above Pig code into the riskfactor.pig window.

**d. Filter your data set**

The next step is to select a subset of the records so that we just have the records of drivers for which the event is not normal. To do this in Pig we use the Filter operator. We tell Pig to Filter our table and keep all records where event !=“normal” and store this in b. With this one simple statement Pig will look at each record in the table and filter out all the ones that do not meet our criteria.

*   We can use Pig Help again by clicking on **Pig helper->Relational Operators->FILTER template**
*   We can replace **%VAR%** with **“a”** (hint: tab jumps you to the next field)
*   Our **%COND%** is “**event !=’normal’;** ” (note: single quotes are needed around normal and don’t forget the trailing semi-colon)
*   Complete line of code will look like:

```
b = filter a by event != 'Normal';
```

Copy-and-paste the above Pig code into the riskfactor.pig window.

**e. Iterate your data set**

Now that we have the right set of records we can iterate through them. We use the “foreach” operator on the grouped data to iterate through all the records. We would also like to know how many times a driver has a non normal event associated with him. to achieve this we add ‘1’ to every row in the data set.

*   Pig helper ->Relational Operators->FOREACH template will get us the code
*   Our **%DATA%** is **b** and the second **%NEW_DATA%** is “**driverid,event,(int) ‘1’ as occurance;**”
*   Complete line of code will look like:

```
c = foreach b generate driverid, event, (int) '1' as occurance;
```

Copy-and-paste the above Pig code into the riskfactor.pig window:

**f. Calculate the total non normal events for each driver**

The group statement is important because it groups the records by one or more relations. In this case we would like to group by driver id and iterate over each row again to sum the non normal events.

*   **Pig helper ->Relational Operators->GROUP %VAR% BY %VAR%** template will get us the code
*   First **%VAR%** takes **“c”** and second **%VAR%** takes “**driverid;**”
*   Complete line of code will look like:

```
d = group c by driverid;
```

Copy-and-paste the above Pig code into the riskfactor.pig window.

*   Next use Foreach statement again to add the occurance.

```
e = foreach d generate group as driverid, SUM(c.occurance) as t_occ;
```

**g. Load drivermileage table and perform a join operation**

In this section we will load drivermileage table into Pig using Hcatlog and perform a join operation on driverid. The resulting data set will give us total miles and total non normal events for a particular driver.

*   Load drivermileage using HcatLoader()

```
g = LOAD 'drivermileage' using org.apache.hive.hcatalog.pig.HCatLoader();
```

*   **Pig helper ->Relational Operators->JOIN %VAR% BY** template will get us the code
*   Replace **%VAR%** by ‘**e**’ and after **BY** put ‘**driverid, g by driverid;**’
*   Complete line of code will look like:

```
h = join e by driverid, g by driverid;
```

Copy-and-paste the above two Pig codes into the riskfactor.pig window.

**h. Compute Driver Risk factor**

In this section we will associate a driver risk factor with every driver. Driver risk factor will be calculated by dividing total miles travelled by non normal event occurrences.

*   We will use Foreach statement again to compute driver risk factor for each driver.
*   Use the following code and paste it into your Pig script.

```
final_data = foreach h generate $0 as driverid, $1 as events, $3 as totmiles, (float) $3/$1 as riskfactor;
```

*   As a final step store the data into a table using Hcatalog.

store final_data into ‘riskfactor’ using

Here is the final code and what it will look like once you paste it into the editor.

— Geolocation has data stored in ORC format

```
a = LOAD 'geolocation' using org.apache.hive.hcatalog.pig.HCatLoader();
b = filter a by event != 'Normal';
c = foreach b generate driverid, event, (int) '1' as occurance;
d = group c by driverid;
e = foreach d generate group as driverid, SUM(c.occurance) as t_occ;
g = LOAD 'drivermileage' using org.apache.hive.hcatalog.pig.HCatLoader();
h = join e by driverid, g by driverid;
final_data = foreach h generate $0 as driverid, $1 as events, $3 as totmiles, (float) $3/$1 as riskfactor;
store final_data into 'riskfactor' using org.apache.hive.hcatalog.pig.HCatStorer();
```

![Lab3_8](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_8.png)

Save the file riskfactor.pig by clicking the **Save** button in the left-hand column.

<!-- @task, "text" : "Complete Step 3.2."-->

<!-- @section -->

## Step 3.3: Quick Recap

Before we execute the code, let’s review the code again:

*   The line a= loads the geolocation table from HCatalog.
*   The line b= filters out all the rows where the event is not ‘Normal’.

*   Then we add a column called occurrence and assign it a value of 1.
*   We then group the records by driverid and sum up the occurrences for each driver.
*   At this point we need the miles driven by each driver, so we load the table we created using Hive.
*   To get our final result, we join by the driverid the count of events in e with the mileage data in g.
*   Now it is real simple to calculate the risk factor by dividing the miles driven by the number of events

You need to configure the Pig Editor to use HCatalog so that the Pig script can load the proper libraries. In the Pig arguments text box, enter –**useHCatalog** and click the **Add** button:

![Lab3_9](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_9.png)

The **Arguments** section of the Pig View should now look like the following:
![Lab3_10](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_10.png)

<!-- @task, "text" : "Complete Step 3.3."-->

<!-- @section -->

## Step 3.4: Execute Pig Script on Tez

1\.  You are now ready to execute the script. Click Execute on Tez checkbox and finally hit the blue **Execute** button to submit the job. Pig job will be submitted to the cluster. This will generate a new tab with a status of the running of the Pig job and at the top you will find a progress bar that shows the job status.

![Lab3_11](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_11.png)

2\.  Wait for the job to complete. The output of the job is displayed in the **Results** section. Your script does not output any result – it stores the result into a Hive table – so your Results section will be empty.

![Lab3_12](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_12.png)

![Lab3_13](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_13.png)

Click on the **Logs** twisty to see what happened when your script ran. This is where you will see any error messages. The log may scroll below the edge of your window so you may have to scroll down.

3\.  Go back to the Ambari Hive User View and browse the data in the riskfactor table to verify that your Pig job successfully populated this table. Here is what is should look like:

![Lab3_14](http://hortonworks.com/wp-content/uploads/2015/07/Lab3_14.png)

At this point we now have our truck miles per gallon table and our risk factor table. The next step is to pull this data into Excel to create the charts for the visualization step.

<!-- @task, "text" : "Complete Step 3.4."-->
