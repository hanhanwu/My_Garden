### Customized Airflow Pipeline

Airflow came out 2 years later than Luigi, but it is more popular now because of its scalability, visualization and flexibility in building the workflows.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/notes/airflow_dag.png" width="766" height="79" />
</p>


#### Airflow Setup (Local Mac)

In order to use the Airflow ecosystem, it takes more effort than Luigi setup. Let's share Lady H.'s notes. She has tried both docker setup and local setup on Windows and Mac, but found the easiest way was to setup on Mac or Linux locally. Here's how did she get Airflow run on Mac:

* To install Airflow, just need to follow the steps [here][1]
  * Better to take a note of which python version used by your Airflow, so that you can find its site packages later 
* After the installation succeeded, each time
  * Just type `airflow standalone` through your terminal, to start Airflow
  * Then you can get access to http://localhost:8080/home, which is the interface of all the DAGs (workflows)
  * To create your own DAG
    * Make sure you have defined AIRFLOW_HOME during the installation stage, like this `export AIRFLOW_HOME=~/airflow`
    * Find file `airflow.cfg` in your AIRFLOW_HOME folder, in this file, make sure to specify DAGs folder like this `dags_folder = ~/airflow/dags`, this is where you will add new DAGs as .py files
    * When creating your DAG in the .py file, make sure to define the `dag_id`, this will be the file name shown in the DAG list
    * To check whether your DAG has been added to the DAG list, type `airflow dags list` through a terminal and you will see all the DAGs. If you want to find DAGs through keywords, you can type `airflow dags list | grep [key_word]`

For example, Lady H. was trying to search for DAGs with "super" in their names:

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/customized_pipeline/dag_list.png" width="1149" height="48" />
</p>

  * If your DAG isn't shown in the list
    * Check whether all the imported packages were installed in the site packages of the python used by your Airflow 
    * Check whether there is any error shown at the top of http://localhost:8080/home, if so, fix it

Once your DAG appeared in the list, you can run it through the localhost interface:

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/customized_pipeline/trigger_airflow_dag.png" width="1000" height="155" />
</p>

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/notes/windows_local_airflow.png" width="766" height="79" />
</p>

If you know how to setup Airflow in other ways, such as using docker or on Windows system, feel free to [show and tell us  details][2]! 💝

#### Super Mini Airflow Pipeline

The learning curve of Airflow is steeper than Luigi, to help your learning, Lady H. decided to exhibit a super mini Airflow pipeline that covers the key learning points.

This super mini pipeline only has 2 tasks, data spliting task followed by model training task. This whole workflow is a DAG and can be defined within a .py file.

🌻 [Check Airflow super_mini_pipeline.py DAG >>][3]

<p align="center">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/customized_pipeline/correct_airflow_flow.png" width="326" height="82" />
</p>

At the very beginning of this DAG, you will import different python packages, besides built-in python and airflow packages, other packages (such as `lightgbm`, `sklearn`) need to be installed in the site packages of the python used by airflow, otherwise your DAG won't appear in the DAG list.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/customized_pipeline/airflow_code1.png" width="448" height="245" />
</p>

Then you need to specifiy the parameters of this DAG. Make sure `dag_id` and `tags` are unique for each DAG.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/customized_pipeline/airflow_code2.png" width="566" height="49" />
</p>

Now it's the core part where you need to define each task. As we can see, the logic of data spliting and model training is defined in functions, and each task is defined as a `PythonOperator`, whose id is specified in its `task_id` and its `python_callable` matches to the function name of this task, `op_kwargs` stores the (key, value) pairs of the function parameters. 

For example, `split_data_task` has its logic defined in function `split_data()`, therefore, its `python_callable` is "split_data". Meanwhile, function `split_data()` has user configurable parameter `label`, in this case the value of `label` is "species", so `'label':'species'` appears in the `op_kwargs` of this task.

You must also have noticed the `xcom_push()` and `xcom_pull()` used in the code. In this DAG, they are used to transfer the data between tasks:

1. In `split_data()` function, `xcom.push()` is used to push the absolute path of the data output. <b>Remember, airflow only accepts the absolute path</b>.
2. In order to load the split data, `train_model()` function uses `xcom_pull()` to locate the pushed data through `task_ids` and the `key` specified in `xcom_push()`. In this example, `task_ids` points to the `task_id` of `split_data_task`.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/customized_pipeline/airflow_code3.png" width="1000" height="600" />
</p>

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/notes/airflow_xcom.png" width="766" height="79" />
</p>

After seeing your DAG appeared in the DAG list, you can run it, and may get errors in a certain task. For example, as shown below,  data spliting task succeeded so it's marked as green and model training task failed so it's marked in red:

<p align="center">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/customized_pipeline/airflow_error_flow.png" width="333" height="86" />
</p>

To check the error log, you can click the failed task and find the log, it will indicate which lines of code caused the errors.

<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/customized_pipeline/airflow_error_log.png" width="1000" height="430" />
</p>

After trials and errors, finally you will get green lights on every task of the DAG 🥳, and from the Tree view, you can see an overview of all the historical attempts.

<p align="center">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/Garden_Market_images/customized_pipeline/airflow_flow.png" width="510" height="126" />
</p>

#
<p align="left">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/follow_us.png" width="120" height="50" />
</p>

[Keep going >>][4]

<p align="right">
<img src="https://github.com/lady-h-world/My_Garden/blob/main/images/going_back.png" width="60" height="44" />
</p>

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;[<< Looking back][5]
 

[1]:https://airflow.apache.org/docs/apache-airflow/stable/start/local.html
[2]:https://github.com/lady-h-world/My_Garden/discussions/categories/show-and-tell
[3]:https://github.com/lady-h-world/My_Garden/blob/main/code/garden_market/airflow_pipeline/super_mini_pipeline.py
[4]:https://github.com/lady-h-world/My_Garden/blob/main/reading_pages/Garden_Market/customized_pipeline6.md
[5]:https://github.com/lady-h-world/My_Garden/blob/main/reading_pages/Garden_Market/customized_pipeline4.md
