# Design a Distributed Job Scheduler

## _Fun. / Non-Fun. Requirements_
### Functional Requirements
1. **Job Submission:**
     * Users should be able to submit jobs to the scheduler.
     * Support for different job types (e.g., one-time, recurring).
     * Ability to specify job execution time and frequency.
2. **Job Management:**
     * Users can view, update, or cancel scheduled jobs.
     * Support for job prioritization.
     * Ability to pause and resume jobs.
3. **Job Execution:**
    * Execute jobs at the specified time.
    * Support for distributed execution across multiple nodes.
    * Ensure job execution reliability and consistency.
4. **Job Monitoring and Logging:**
    * Provide real-time status updates for jobs.
    * Log job execution details and history.
    * Alert users on job failures or delays.
5. **Scalability:**
    * Handle a large number of concurrent job submissions and executions.
    * Support for horizontal scaling to accommodate increased load.
6. **Security:**
    * Authenticate and authorize users for job submission and management.
    * Ensure data privacy and integrity during job execution.
7. **Fault Tolerance:**
    * Automatically retry failed jobs. 
    * Ensure system availability in case of node failures.
8. **API Access:**
   * Provide RESTful APIs for job submission, management, and monitoring.

### Non-Functional Requirements

1. **Performance:**
   * Low latency in job scheduling and execution.
   * Efficient resource utilization to minimize execution time.
2. **Reliability:**
   * High availability with minimal downtime.
   * Ensure job execution even in the event of partial system failures.
3. **Scalability:**
   * Ability to scale horizontally to support increasing job loads.
   * Efficient load balancing across distributed nodes.
4. **Consistency:**
   * Ensure data consistency across distributed components.
   * Provide eventual consistency for job status updates.
5. **Resilience:**
   * System should recover gracefully from failures.
   * Implement redundancy to prevent data loss.
6. **Security:**
   * Secure communication channels (e.g., HTTPS).
   * Implement role-based access control for job management.
7. **Observability:**
   * Provide comprehensive logging and monitoring.
   * Enable easy tracking of job execution metrics and system health.
8. **Extensibility:**
   * Design the system to accommodate future feature additions.
   * Support for plugin-based architecture for custom job types.

## _Traffic Estimation and Data Calculation_
#### Assumptions
1. **User Base:**
   * 10,000 active users.
   * Each user submits an average of 5 jobs per day.
2. **Job Characteristics:**
   * Average job size: 1 KB metadata. 
   * Average job execution time: 5 minutes. 
   * 10% of jobs are recurring.
3. **System Load:**
   * Peak load occurs during business hours (9 AM - 5 PM). 
   * 70% of jobs are submitted during peak hours.
4. **Data Retention:**
   * Job logs retained for 30 days. 
   * Job metadata retained indefinitely.
#### Write Flow
1. Job Submission:
   * 50,000 job submissions per day. 
   * 35,000 submissions during peak hours. 
   * 15,000 submissions during off-peak hours.
2. Job Updates:
   * 20% of jobs are updated at least once. 
   * 10,000 job updates per day.
3. Job Execution Logs:
   * Each job generates 5 KB of logs. 
   * 250,000 KB (250 MB) of logs generated per day.
#### Read Flow
1. Job Status Checks:
   * Users check job status twice per job on average. 
   * 100,000 status checks per day.
2. Job History Retrieval:
   * 10% of users retrieve job history daily.
   * 1,000 job history retrievals per day.
3. Monitoring and Alerts:
   * System generates alerts for 1% of jobs.
   * 500 alerts per day.
#### Data Storage
1. Job Metadata:
   * 50,000 jobs x 1 KB = 50,000 KB (50 MB) per day.
   * 50 MB * 30 Day = 1.5 GB per month.
2. Job Logs:
   * 250 MB of logs per day.
   * 250 MB * 30 Day = 7.5 GB of logs per month.

**Total Storage Requirement:**
   * 1.5 GB (metadata) + 7.5 GB (logs) = 9 GB per month.
   * 9 GB * 12 Months = 108 GB per year.
3. **Database Growth:**
   * Assuming a 10% annual increase in user base and job submissions.
   * Plan for 120 GB storage capacity for the first year, with scalability options for future growth.

## _API Design_

1. **Submit Job**
    * Endpoint: POST /api/jobs
    * Request:
      ```json
      {
         "jobName": "string",
         "jobType": "one-time | recurring",
         "executionTime": "ISO 8601 timestamp",
         "frequency": "string (e.g., daily, weekly)",
         "priority": "low | medium | high",
         "payload": "object"
      }
      ```
    * Response:
      ```json
      {
         "jobId": "string",
         "status": "submitted"
      }
      ```

2. **Update Job**
    * Endpoint: PUT /api/jobs/{jobId}
    * Request:
      ```json
      {
         "jobName": "string",
         "executionTime": "ISO 8601 timestamp",
         "frequency": "string",
         "priority": "low | medium | high",
         "payload": "object"
      }
      ```
    * Response:
      ```json
      {
         "jobId": "string",
         "status": "updated"
      }
      ```
3. **Cancel Job**
    * Endpoint: DELETE /api/jobs/{jobId}
    * Request: None
    * Response:
      ```json
      {
         "jobId": "string",
         "status": "cancelled"
      }
      ```
4. **Get Job Status**
    * Endpoint: GET /api/jobs/{jobId}/status
    * Request: None
    * Response:
      ```json
      {
         "jobId": "string",
         "status": "pending | running | completed | failed",
         "lastUpdated": "ISO 8601 timestamp"
      }
      ```
5. **List Jobs**
    * Endpoint: GET /api/jobs
    * Request:
      * Query Parameters: status, type, priority, page, limit
    * Response:
      ```json
      [
         {
            "jobId": "string",
            "jobName": "string",
            "status": "string",
            "executionTime": "ISO 8601 timestamp"
         },
         ...
      ]
      ```

6. **Get Job Logs**
    * Endpoint: GET /api/jobs/{jobId}/logs
    * Request: None
    * Response:
      ```json
      {
         "jobId": "string",
         "logs": [
            {
               "timestamp": "ISO 8601 timestamp",
               "message": "string"
            },
            ...
         ]
      }
      ```
7. **Pause Job**
    * Endpoint: POST /api/jobs/{jobId}/pause
    * Request: None
    * Response:
      ```json
      {
         "jobId": "string",
         "status": "paused"
      }
      ```
8. **Resume Job**
    * Endpoint: POST /api/jobs/{jobId}/resume
    * Request: None
    * Response:
      ```json
      {
         "jobId": "string",
         "status": "resumed"
      }
      ```