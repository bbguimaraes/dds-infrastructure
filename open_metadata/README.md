OpenMetadata
============

S3 storage
----------

### Create storage

- create the bucket in the Hetzner console
  - `https://console.hetzner.com/projects/<id>/buckets`
  - Object storage → Create bucket
- add S3 credentials
  - `https://console.hetzner.com/projects/<id>/buckets/<id>/overview`
  - S3 Credential → Generate Credentials

### Configure the AWS CLI

```
# .aws/config
[default]
region = <hetzner-location>
endpoint_url = https://<hetzner-location>.your-objectstorage.com
```

```
# .aws/credentials
[default]
aws_access_key_id = # …
aws_secret_access_key = # …
```

### Add a manifest

Add a manifest file describing the contents of the bucket at the root.

https://docs.open-metadata.org/latest/connectors/storage/s3#openmetadata-manifest

```console
$ mkdir tmp/
$ cat > tmp/openmetadata.json <<'EOF'
{"entries": [{"dataPath": "data", "structureFormat": "json"}]}
EOF
$ aws s3 sync tmp/ s3://<bucket-name>
```

### Create pipeline

https://docs.open-metadata.org/latest/connectors/storage/s3#metadata-ingestion

- Settings → Services → Databases → Add
- Storage services → S3
    - AWS access key ID / secret access key: as displayed in the "Generate
      credentials" page
    - AWS region: Hetzner location (e.g. `fsn1`)
    - Endpoint URL: `https://<location>.your-objectstorage.com`
    - Bucket names: Hetzner storage name
- verify configuration is correct with the "Test connection" button (CloudWatch
  errors are expected and harmless)
- Agents → Add agent → Add metadata agent

### Add data

```console
$ mkdir tmp/data
$ cat > tmp/data/test.json <<'EOF'
{"test_key":"test_value"}
EOF
$ aws s3 sync tmp/ s3://<bucket-name>
```

### Execute ingestion

Back on the storage service page:

`https://<host>/service/storageServices/<name>`

- Agents → "…" menu → Run

<details>
    <summary>example log</summary>

```
9407994e07f2
 INFO - ::group::Log message source details
*** Found local files:
***   * /opt/airflow/logs/dag_id=8f56fc12-83db-4972-99a5-ba8ded1cbbcf/run_id=manual__2025-09-09T07:19:49+00:00/task_id=ingestion_task/attempt=1.log
 INFO - ::endgroup::
[2025-09-09T07:19:51.017+0000] {local_task_job_runner.py:123} INFO - ::group::Pre task execution logs
[2025-09-09T07:19:51.037+0000] {taskinstance.py:2614} INFO - Dependencies all met for dep_context=non-requeueable deps ti=<TaskInstance: 8f56fc12-83db-4972-99a5-ba8ded1cbbcf.ingestion_task manual__2025-09-09T07:19:49+00:00 [queued]>
[2025-09-09T07:19:51.043+0000] {taskinstance.py:2614} INFO - Dependencies all met for dep_context=requeueable deps ti=<TaskInstance: 8f56fc12-83db-4972-99a5-ba8ded1cbbcf.ingestion_task manual__2025-09-09T07:19:49+00:00 [queued]>
[2025-09-09T07:19:51.043+0000] {taskinstance.py:2867} INFO - Starting attempt 1 of 1
[2025-09-09T07:19:51.055+0000] {taskinstance.py:2890} INFO - Executing <Task(CustomPythonOperator): ingestion_task> on 2025-09-09 07:19:49+00:00
[2025-09-09T07:19:51.066+0000] {standard_task_runner.py:72} INFO - Started process 7720 to run task
[2025-09-09T07:19:51.069+0000] {standard_task_runner.py:104} INFO - Running: ['airflow', 'tasks', 'run', '8f56fc12-83db-4972-99a5-ba8ded1cbbcf', 'ingestion_task', 'manual__2025-09-09T07:19:49+00:00', '--job-id', '12', '--raw', '--subdir', 'DAGS_FOLDER/8f56fc12-83db-4972-99a5-ba8ded1cbbcf.py', '--cfg-path', '/tmp/tmpfw9a3ig5']
[2025-09-09T07:19:51.070+0000] {standard_task_runner.py:105} INFO - Job 12: Subtask ingestion_task
[2025-09-09T07:19:51.131+0000] {task_command.py:467} INFO - Running <TaskInstance: 8f56fc12-83db-4972-99a5-ba8ded1cbbcf.ingestion_task manual__2025-09-09T07:19:49+00:00 [running]> on host 9407994e07f2
[2025-09-09T07:19:51.215+0000] {taskinstance.py:3134} INFO - Exporting env vars: AIRFLOW_CTX_DAG_OWNER='admin' AIRFLOW_CTX_DAG_ID='8f56fc12-83db-4972-99a5-ba8ded1cbbcf' AIRFLOW_CTX_TASK_ID='ingestion_task' AIRFLOW_CTX_EXECUTION_DATE='2025-09-09T07:19:49+00:00' AIRFLOW_CTX_TRY_NUMBER='1' AIRFLOW_CTX_DAG_RUN_ID='manual__2025-09-09T07:19:49+00:00'
[2025-09-09T07:19:51.216+0000] {taskinstance.py:732} INFO - ::endgroup::
[2025-09-09T07:19:51.246+0000] {server_mixin.py:74} INFO - OpenMetadata client running with Server version [1.8.6] and Client version [1.8.6.0]
[2025-09-09T07:19:51.633+0000] {test_connections.py:203} INFO - Running ListBuckets...
[2025-09-09T07:19:51.853+0000] {test_connections.py:203} INFO - Running GetMetrics...
[2025-09-09T07:19:53.109+0000] {test_connections.py:214} ERROR - GetMetrics-An error occurred (Unknown) when calling the ListMetrics operation: Unknown
[2025-09-09T07:19:53.110+0000] {test_connections.py:228} INFO - Test connection results:
[2025-09-09T07:19:53.110+0000] {test_connections.py:229} INFO - lastUpdatedAt=None status=<StatusType.Running: 'Running'> steps=[TestConnectionStepResult(name='ListBuckets', mandatory=True, passed=True, message=None, errorLog=None), TestConnectionStepResult(name='GetMetrics', mandatory=False, passed=False, message='Failed to fetch Cloudwatch AWS/S3 metrics, please validate if user has access to fetch these credentials', errorLog='An error occurred (Unknown) when calling the ListMetrics operation: Unknown')]
[2025-09-09T07:19:53.111+0000] {test_connections.py:242} WARNING - You might be missing metadata in: GetMetrics due to Failed to fetch Cloudwatch AWS/S3 metrics, please validate if user has access to fetch these credentials
[2025-09-09T07:19:55.050+0000] {metadata.py:620} WARNING - Failed fetching metric NumberOfObjects for bucket test, returning 0
[2025-09-09T07:19:56.276+0000] {metadata.py:620} WARNING - Failed fetching metric BucketSizeBytes for bucket test, returning 0
[2025-09-09T07:19:56.395+0000] {metadata.py:750} INFO - Looking for metadata template file at - s3://test/openmetadata.json
[2025-09-09T07:19:56.481+0000] {metadata.py:385} INFO - Extracting metadata from path data and generating structured container
[2025-09-09T07:19:56.564+0000] {metadata.py:686} INFO - File data/test.json was picked to infer data structure from.
[2025-09-09T07:19:56.969+0000] {logger.py:201} INFO - Workflow S3 Summary:
[2025-09-09T07:19:56.970+0000] {logger.py:201} INFO - Processed records: 1
[2025-09-09T07:19:56.970+0000] {logger.py:201} INFO - Updated records: 0
[2025-09-09T07:19:56.970+0000] {logger.py:201} INFO - Warnings: 0
[2025-09-09T07:19:56.970+0000] {logger.py:201} INFO - Errors: 0
[2025-09-09T07:19:56.971+0000] {logger.py:201} INFO - Success %: 100.0
[2025-09-09T07:19:56.971+0000] {logger.py:201} INFO - Workflow OpenMetadata Summary:
[2025-09-09T07:19:56.971+0000] {logger.py:201} INFO - Processed records: 1
[2025-09-09T07:19:56.971+0000] {logger.py:201} INFO - Updated records: 0
[2025-09-09T07:19:56.972+0000] {logger.py:201} INFO - Warnings: 0
[2025-09-09T07:19:56.974+0000] {logger.py:201} INFO - Errors: 0
[2025-09-09T07:19:56.974+0000] {logger.py:201} INFO - Success %: 100.0
[2025-09-09T07:19:56.974+0000] {logger.py:201} INFO - Workflow Success %: 100.0
[2025-09-09T07:19:56.978+0000] {logger.py:201} INFO - Workflow finished in time: 5.51s
[2025-09-09T07:19:56.978+0000] {python.py:240} INFO - Done. Returned value was: None
[2025-09-09T07:19:57.126+0000] {local_task_job_runner.py:266} INFO - Task exited with return code 0
[2025-09-09T07:19:57.146+0000] {taskinstance.py:3901} INFO - 0 downstream tasks scheduled from follow-on schedule check
[2025-09-09T07:19:57.148+0000] {local_task_job_runner.py:245} INFO - ::endgroup::
```

</details>
