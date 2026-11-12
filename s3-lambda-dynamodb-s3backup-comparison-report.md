**AWS Lambda + DynamoDB + Scheduled S3 Backups\
Evidence-Based Project Report**

*Report based on README.md requirements and submitted screenshots*

1.  1\. Report Purpose and Method

This report evaluates the completed AWS lab against the requirements and
questions stated in the project README.md. The implementation evidence
is taken from the supplied screenshots. The report compares the expected
outcomes from the assignment with the observed results, then answers the
six required student questions.

1.  2\. Expected Architecture

The required architecture is: JSONPlaceholder API → Lambda #1 (import) →
DynamoDB → Lambda #2 (export) → Amazon S3. EventBridge Scheduler invokes
Lambda #2 approximately every two minutes, while CloudWatch provides
logs and metrics.

Lambda #1 imports 10 sample users and writes them to the
jsonplaceholder-users DynamoDB table.

Lambda #2 reads the DynamoDB table and creates timestamped JSON exports
in a private S3 bucket.

EventBridge Scheduler automatically invokes Lambda #2 using rate(2
minutes).

CloudWatch is used to verify execution, logs, invocations, errors,
duration, and related runtime behavior.

1.  3\. Requirements vs. Observed Results

  -------------------------- ------------------------------------------------------------- ------------------------------------------------------------------------------ ------------------------------------------------
  Requirement                Expected Result                                               Observed Screenshot Evidence                                                   Assessment
  DynamoDB table             jsonplaceholder-users with id as Number partition key         Screenshots 1 and 7 show the DynamoDB resource and stored records.             Consistent with requirement
  Private S3 bucket          Bucket exists as export destination                           Screenshot 2 shows the S3 bucket; screenshots 8 and 10 show backup output.     Consistent with requirement
  Lambda #1 permissions      Execution role includes DynamoDB PutItem plus basic logging   Screenshot 3 shows the Lambda #1 IAM role configuration.                       Evidence supports least-privilege write access
  Lambda #1 test             Successful import of 10 users                                 Screenshot 5 shows a successful import Lambda test.                            Successful
  DynamoDB population        Imported users visible in table                               Screenshot 7 shows records in the DynamoDB table.                              Successful
  Lambda #2 permissions      Read/Scan DynamoDB and PutObject to S3                        Screenshot 4 shows Lambda #2 configuration/permissions.                        Consistent with export role
  Lambda #2 manual export    Successful timestamped JSON backup in S3                      Screenshot 6 shows successful test; screenshot 8 shows manual backup output.   Successful
  Scheduler                  Enabled recurring rate(2 minutes) schedule                    Screenshot 9 shows the schedule enabled.                                       Successful
  Automatic backups          Two or more distinct backup files from scheduled runs         Screenshot 10 shows multiple backups generated over successive runs.           Successful
  CloudWatch observability   Logs and metrics for both functions                           Screenshots 11--14 show logs, metrics, invocations and error monitoring.       Successful evidence of monitoring
  -------------------------- ------------------------------------------------------------- ------------------------------------------------------------------------------ ------------------------------------------------

1.  4\. Analysis of the Required Questions

    1.  ## Question 1: What is the difference between Lambda\'s execution role and Scheduler\'s execution role?

Lambda\'s execution role is assumed by the Lambda service while the
function is running. It grants the code permission to perform actions
such as DynamoDB PutItem, DynamoDB Scan, S3 PutObject, and CloudWatch
logging. The EventBridge Scheduler execution role is different:
Scheduler assumes it when delivering the scheduled invocation, and it
needs permission to invoke the target Lambda function. Therefore, the
Scheduler role controls the trigger-to-Lambda action, while the Lambda
execution role controls what the function can do after it starts.

In short, Scheduler invokes Lambda function, and then Lambda function
does what it supposed to do.

**Comparison with results: Screenshot 3 provides evidence of a Lambda #1
execution role, screenshot 4 shows the Lambda #2
permissions/configuration, and screenshot 9 demonstrates the enabled
Scheduler. Together they support the separation of responsibilities
required by the assignment.**

1.  1.  ## Question 2: Why does running Lambda #1 twice not produce 20 DynamoDB users?

The imported users use id as the DynamoDB partition key. DynamoDB
PutItem writes an item with a specific primary key; if another PutItem
uses the same id, it replaces the existing item rather than creating a
second item with the same key. Because JSONPlaceholder returns the same
10 users with the same IDs, running the import twice results in
approximately 10 uniquely keyed records, not 20.

On the other word, each time Lambda#1 excutes, it overwrites the
existing datasets, not appened them. Therefore it always keep the same
user counts.

**Comparison with results: Screenshot 5 shows a successful import, while
screenshot 7 shows the populated DynamoDB table. The observed design
matches the expected overwrite behavior for repeated IDs.**

1.  1.  ## Question 3: Why do Lambda #2 backups have unique timestamps and request IDs?

Each export run is a separate Lambda invocation. The backup filename
includes a timestamp so each successful run can create a distinct S3
object instead of overwriting an earlier export. AWS also assigns each
Lambda invocation a unique request ID for tracing and troubleshooting.
Timestamps identify the backup version/time, while request IDs identify
the individual execution in logs and monitoring.

**Comparison with results: Screenshot 8 shows manual backup evidence and
screenshot 10 shows multiple distinct backups produced by repeated
scheduled executions, consistent with unique timestamp-based output.**

1.  1.  ## Question 4: What is the difference between print() logs and Invocations/Errors metrics?

print() output is application-level diagnostic information written to
CloudWatch Logs, such as messages describing saved users or exported
records. Invocations and Errors are CloudWatch/Lambda metrics that
aggregate execution outcomes over time. Metrics answer questions such as
how often a function ran or failed; logs provide detailed messages for
individual executions. Runtime START, END, and REPORT entries also
appear in logs and provide per-invocation context.

**Comparison with results: Screenshots 11 and 12 provide log evidence
for the two Lambda functions, while screenshots 13 and 14 show metrics
and invocation/error monitoring. This directly demonstrates the
distinction between detailed logs and aggregated metrics.**

1.  1.  ## Question 5: What would happen if the DynamoDB table were deleted but its JSON files remained in S3? Could the original indexes/configuration be automatically restored from JSON alone?

The JSON files would remain in S3 as exported data records, so the data
copy could still be read or used to repopulate a new table. However,
JSON alone is not a complete DynamoDB backup. It does not automatically
preserve and recreate the full table definition, secondary indexes,
capacity settings, Point-in-Time Recovery configuration, IAM policies,
alarms, encryption configuration, or other operational settings. A new
table and its configuration would have to be recreated before importing
the records unless a proper managed backup/export and restore workflow
were used.

**Comparison with results: The screenshots demonstrate JSON data export
to S3, but they do not demonstrate preservation of complete DynamoDB
configuration. This matches the README warning that the exercise is an
export, not a production-grade backup.**

1.  1.  ## Question 6: Why is an export every 2 minutes usually a poor production backup strategy for large DynamoDB tables?

For large or busy tables, scanning the entire table every two minutes
can consume significant read capacity, increase cost, create many S3
objects, and potentially exceed Lambda runtime or memory limits. A Scan
is not a transactionally consistent point-in-time snapshot, so a
changing table can produce an export representing different moments in
time. Frequent scans can also create operational load and complex
retention management. Production workloads normally require a backup
strategy based on recovery objectives, such as DynamoDB Point-in-Time
Recovery or managed DynamoDB Export to S3.

**Comparison with results: Screenshot 10 successfully proves that
frequent backups can be generated for this small lab, but the same
success also illustrates why object accumulation becomes a concern. The
two-minute schedule is appropriate for demonstrating automation, not as
a general production backup pattern.**

1.  5\. Screenshot Evidence

The screenshots below are included as the implementation evidence used
for the comparisons in this report.

1.  1.  ## Figure 1: 1-dynamodb-table.PNG

DynamoDB table creation/configuration evidence.

![](Pictures/100002010000064A000000D0DA6C5A24.png){width="17.272cm"
height="2.231cm"}

1.  1.  ## Figure 2: 2-s3-bucket.PNG

Private S3 bucket evidence.

![](Pictures/1000020100000419000000B130983B47.png){width="17.272cm"
height="2.914cm"}

1.  1.  ## Figure 3: 3-rw-db-iam-lamda-1-role.PNG

Lambda #1 execution role and permission evidence.

![](Pictures/10000201000003BD0000018791F69A54.png){width="17.272cm"
height="7.056cm"}

1.  1.  ## Figure 4: 4-lambda-2-rdb-rw-S3-jasonbackup.PNG

Lambda #2 configuration and read/write permission evidence.

![](Pictures/100002010000039E000001B439D1BCC7.png){width="17.272cm"
height="8.132cm"}

1.  1.  ## Figure 5: 5-lambda-1-import-jason-test-success.PNG

Successful Lambda #1 import test.

![](Pictures/10000201000006B900000357CF4F77E1.png){width="17.272cm"
height="8.58cm"}

1.  1.  ## Figure 6: 6-lambda-2-export-dynamodb-s3-test-success.PNG

Successful Lambda #2 export test.

![](Pictures/10000201000006AA00000359662B9F0A.png){width="17.272cm"
height="8.677cm"}

1.  1.  ## Figure 7: 7-dynamodb-table.PNG

Imported records visible in DynamoDB.

![](Pictures/100002010000074B0000035232B2D323.png){width="17.272cm"
height="7.863cm"}

1.  1.  ## Figure 8: 8-manual-backup.PNG

Manual JSON backup evidence in S3.

![](Pictures/10000201000006320000019A5CAA41E5.png){width="17.272cm"
height="4.464cm"}

1.  1.  ## Figure 9: 9-dbbackup-schedule-enabled.PNG

Enabled EventBridge Scheduler evidence.

![](Pictures/100002010000076A00000350A2F4D428.png){width="17.272cm"
height="7.715cm"}

1.  1.  ## Figure 10: 10-auto-backups-in-every-2-minutes.PNG

Multiple automatic backups generated on the recurring schedule.

![](Pictures/100002010000064D00000361EB6ED368.png){width="17.272cm"
height="9.262cm"}

1.  1.  ## Figure 11: 11-lambda-1-cloudwatch-logs.PNG

CloudWatch application/runtime logs for Lambda #1.

![](Pictures/100002010000075800000317D45B4034.png){width="17.272cm"
height="7.267cm"}

1.  1.  ## Figure 12: 12-lambda-2-cloudwatch-logs.PNG

CloudWatch application/runtime logs for Lambda #2.

![](Pictures/1000020100000749000003552B3FB708.png){width="17.272cm"
height="7.899cm"}

1.  1.  ## Figure 13: 13-lambda-1-cloudwatch-invocations-errorlogs.PNG

Invocation and error monitoring for Lambda #1.

![](Pictures/100002010000075E000002D2C999741C.png){width="17.272cm"
height="6.613cm"}

1.  1.  ## Figure 14: 13-lambda-1-cloudwatch-invocations.PNG

Invocation monitoring for Lambda #1.

![](Pictures/10000201000006970000031D552AF602.png){width="17.272cm"
height="8.16cm"}

1.  1.  ## Figure 15: 13-lambda-1-cloudwatch-metrics.PNG

CloudWatch/Lambda metrics for Lambda #1.

![](Pictures/10000201000006BE00000335E2886127.png){width="17.272cm"
height="8.214cm"}

1.  1.  ## Figure 16: 14-lambda-2-cloudwatch-invocations.PNG

Invocation monitoring for Lambda #2.

![](Pictures/100002010000069C0000034210ADFBF3.png){width="17.272cm"
height="8.513cm"}

1.  1.  ## Figure 17: 14-lambda-2-cloudwatch-metrics.PNG

CloudWatch/Lambda metrics for Lambda #2.

![](Pictures/10000201000006AA0000034B208990BB.png){width="17.272cm"
height="8.534cm"}

1.  6\. Overall Conclusion

The submitted evidence is consistent with the main goals of the
assignment: a serverless pipeline imported user data, stored it in
DynamoDB, exported it to S3, and automated repeated exports through
EventBridge Scheduler. The CloudWatch screenshots provide observability
evidence for both Lambda functions. The results also support the
assignment\'s key lesson that frequent JSON Scan exports are useful for
demonstrating data movement but should not be confused with a complete,
transactionally consistent production backup and disaster-recovery
solution.

1.  7\. Final Comparison Summary

Expected lab outcome: 10 imported users → successful DynamoDB storage →
successful manual JSON export → enabled two-minute schedule → repeated
S3 backup files → CloudWatch evidence. Observed evidence: the supplied
screenshots show each of these stages, including successful Lambda
tests, populated DynamoDB data, manual and repeated S3 backups, an
enabled schedule, and CloudWatch logs/metrics. Therefore, the observed
results align with the core expected outcomes of the assignment.
