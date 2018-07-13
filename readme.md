# airflow-test
[Apache Airflow](https://airflow.apache.org/) is a platform to programmatically author, schedule, and monitor workflows. When workflows are defined as code, they become more maintainable, versionable, testable, and collaborative.

## Introduction
Apache Airflow is quickly becoming the de-facto standard to do ETL-batch processing and is most often combined with [Apache Spark](https://spark.apache.org/) to do data transformations. Apache Airflow solves the dependency problem where one job depends on another job, where a job consists of several tasks that depend on one another. Combined with a powerful scheduler and executor, a simple programming language and a powerful DSL, you get a very powerful batch processing system to process data coming from various sources, in various formats, that must be normalized, transformed and exported to other systems.

## Architecture
At its core, Apache Airflow is simply a queuing system built on top of a metadata database. The database stores the state of queued tasks and a scheduler uses these states to prioritize how other tasks are added to the queue. This functionality is orchestrated by four primary components:

- __Metadata Database:__ this database stores information regarding the state of tasks. Database updates are performed using an abstraction layer implemented in `SQLAlchemy`. This abstraction layer cleanly separates the function of the remaining components of Airflow from the database. The following databases are supported: `sqlite`, `mysql` and `postgres`.
- __Scheduler:__ The Scheduler is a process that uses DAG definitions in conjunction with the state of tasks in the metadata database to decide which tasks need to be executed, as well as their execution priority. The Scheduler is generally run as a service. 
- __Executor:__ The Executor is a message queuing process that is tightly bound to the Scheduler and determines the worker processes that actually execute each scheduled task. There are different types of Executors, each of which uses a specific class of worker processes to execute tasks. For example, the `LocalExecutor` executes tasks with parallel processes that run on the same machine as the Scheduler process. Other Executors, like the `CeleryExecutor` execute tasks using worker processes that exist on a separate cluster of worker machines. The `SequentialExecutor` will only run one task instance at a time, can be used for debugging. It is also the only executor that can be used with sqlite since sqlite doesn't support multiple connections. Since we want airflow to work out of the box, it defaults to this SequentialExecutor alongside sqlite as you first install it.
- __Workers:__ These are the processes that actually execute the logic of tasks, and are determined by the Executor being used.

## Concepts
The concepts are described [here](https://airflow.apache.org/concepts.html)

## Python
A small study project on Python from a Java/Scala developers point of view can be found [here](https://github.com/dnvriend/study-python).

## Developing DAGs
...

## Testing DAGs
see: https://bcb.github.io/airflow/testing-dags

Developing Airflow dags involves writing unit tests for the individual tasks, and then manually running the whole dag from start to finish. To test an operator’s method in a unit test, you need to create three objects, a dag, a task (the operator we’re testing), and a TaskInstance. Here we test MyOperator.execute.

```
class TestMyOperator(TestCase):

    def test_execute(self):
        dag = DAG(dag_id='foo', start_date=datetime.now())
        task = MyOperator(dag=dag, task_id='foo')
        ti = TaskInstance(task=task, execution_date=datetime.now())
        result = task.execute(ti.get_template_context())
        self.assertEqual(result, 'foo')
```

## Hide globals in a DAG definition file
see: https://bcb.github.io/airflow/hide-globals-in-dag-definition-file

Airflow has a fairly strange way of registering DAGs and tasks. They’re put into the global namespace of the DAG definition file.

```
dag = DAG(dag_id='foo', start_date=start_date)
MyOperator(dag=dag, task_id='foo')
```

Airflow then comes along and finds them.

When importing that file however, as you do when unit testing, it’s not ideal to have those global objects created.

The solution is to protect that code by preceding it with a predicate:

```
if __name__.startswith('unusual_prefix'):
    dag = DAG(dag_id='foo', start_date=start_date)
    MyOperator(dag=dag, task_id='foo')
```

Airflow will still find your DAG as normal, however that code won’t be executed when the module is imported.

## Airflow's execute context
see: https://bcb.github.io/airflow/execute-context

I often forget the contents of the context dict, and it’s not well documented.

It can be found in `airflow/models.py`, in the `TaskInstance.get_template_context` method.

```
return {
    'END_DATE': ds,
    'conf': configuration,
    'dag': task.dag,
    'dag_run': dag_run,
    'ds': ds,
    'ds_nodash': ds_nodash,
    'end_date': ds,
    'execution_date': self.execution_date,
    'latest_date': ds,
    'macros': macros,
    'params': params,
    'run_id': run_id,
    'tables': tables,
    'task': task,
    'task_instance': self,
    'task_instance_key_str': ti_key_str,
    'test_mode': self.test_mode,
    'ti': self,
    'tomorrow_ds': tomorrow_ds,
    'tomorrow_ds_nodash': tomorrow_ds_nodash,
    'ts': ts,
    'ts_nodash': ts_nodash,
    'yesterday_ds': yesterday_ds,
    'yesterday_ds_nodash': yesterday_ds_nodash,
}
```

As a side note, you can generate the context from a TaskInstance object.

```
ti = TaskInstance(task=task, execution_date=datetime.now())
task.execute(context=ti.get_template_context())
```

## Managed Apache Airflow
- [Google Cloud Composer](https://cloud.google.com/composer/) is a fully managed workflow orchestration service that empowers you to author, schedule, and monitor pipelines that span across clouds and on-premises data centers. Built on the popular Apache Airflow open source project and operated using the Python programming language, Cloud Composer is free from lock-in and easy to use.
- [astronomer.io](https://www.astronomer.io/pricing/): Astronomer makes it easy to deploy and manage your own Apache Airflow cluster, so you can get straight to writing workflows.
- [Qubole](https://www.qubole.com/blog/airflow-as-a-service-on-qds-generally-available/): We are excited to announce that Airflow as a service on Qubole Data Service (QDS) is GA and joins the family of Hadoop 1, Hadoop 2, Spark, Presto, and HBase offered as a service on QDS. 

## Sources
- [Apache Airflow](https://airflow.apache.org/)
- [How Quizlet uses Apache Airflow to execute complex data processing pipelines](https://medium.com/tech-quizlet/going-with-the-flow-how-quizlet-uses-apache-airflow-to-execute-complex-data-processing-pipelines-1ca546f8cc68)
- [Using Apache Airflow to build reusable ETL on AWS Redshift](https://sonra.io/2018/01/01/using-apache-airflow-to-build-a-data-pipeline-on-aws/)
- [Automated Model Building with EMR, Spark, and Airflow](https://www.agari.com/identity-intelligence-blog/automated-model-building-emr-spark-airflow/)

## Videos
- [A Pratctical Introduction to Airflow - Matt Davis](https://www.youtube.com/watch?v=cHATHSB_450)
- [Modern ETL-ing with Python and Airflow (and Spark) - Tamara Mendt](https://www.youtube.com/watch?v=tcJhSaowzUI)
- [Developing elegant workflows in Python code with Apache Airflow - Michał Karzyński](https://www.youtube.com/watch?v=XJf-f56JbFM)
- [Elegant data pipelining with Apache Airflow - Bolke de Bruin](https://www.youtube.com/watch?v=neuh_2_zrt8)