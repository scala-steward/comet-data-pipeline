---
sidebar_position: 3
---

# Transform

Once your data is ingested, you may start to expose insights by joining them and / or create meaningful aggregates.

In the example below, we join the `sellers` and `orders` tables to compute the total amount sold by each seller.

We want to do it on parquet files and on BigQuery. We need to create 2 env files, one for environment.

## Parquet Job

Create the `env.FS.comet.yml` file in the `metadata` folder as follows:

```yaml
env:
  sink_type: "FS"
  engine: Spark
  sellers_view: "FS:${root_path}/datasets/accepted/hr/sellers"
  orders_view: "FS:${root_path}/datasets/accepted/sales/orders"
```

Create the YAML file that describe the job and name it `sellers_kpi.comet.yml`:

```yaml
transform:
  name: kpi
  engine: ${engine}
  views:
    sellers: "${sellers_view}"
    orders: "${orders_view}"
  tasks:
    - sql: "select seller_email, sum(amount) as sum from sellers, orders where sellers.id = orders.seller_id group by sellers.seller_email"
      name: byseller
      domain: sales_kpi
      dataset: byseller_kpi
      write: OVERWRITE
  area: business
```

Before executing the job, we set the `COMET_ENV` variable to `FS` to make sure variables are instantiated correctly:

````shell
export COMET_ENV=FS
$SPARK_HOME/bin/spark-submit target/scala-2.12/comet-assembly-VERSION.jar job --name sellers_kpi
````

## BigQuery Job
To execute the same request on BigQuery, we need to create views on the BigQuery tables where data were ingested. 
Create the `env.BQ.comet.yml` file in the `metadata` folder as follows:

```yaml
env:
  sink_type: "BQ"
  engine: BQ
  sellers_view: "BQ:select * from hr.sellers" # or simply sellers_view: hr.sellers
  orders_view: "BQ:select * from sales.orders" # or simply sellers_view: sales.orders
```

Before executing the job, we set the `COMET_ENV` variable to `BQ` to make sure variables are instantiated correctly:

````shell
export COMET_ENV=BQ
$SPARK_HOME/bin/spark-submit target/scala-2.12/comet-assembly-VERSION.jar job --name sellers_kpi
````

## Externalize SQL Requests

Most of the time you may want to use your favorite SQL Deve env (I personally use Jetbrains IntelliJ / DataGrip). 
You may externalize the SQL Code in a file named after the name of the job if you have only one task in the job file 
or after the name and the task for each task in the job file.

Create a SQL File with the name `ki.byseller.sql` (or simply `ki.sql` since we have only one task in the job file)
```sql
select seller_email, sum(amount) as sum from sellers, orders where sellers.id = orders.seller_id
group by sellers.seller_email
```

And leave the task.sql field blank in the YAML file. 

