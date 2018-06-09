# Watchdog Alerting/Monitoring

## Introduction
HySDS core functionalities include monitoring and alerting services. For exampole, it can  monitor if a queue is inactive (not processing the waiting jobs)  or absence of a successful completion of a particular job type within certain period of time and send alert through email or slack channel which  allows users to take necessary action if needed.

## Queue Monitoring
It monitors if any queue is inactive or a job is hanging there for long time.

### Source code
check_queue_periodicity.py

### Command
python check_queue_periodicity.py <rabbitmq_account>:<port_no> periodicity_time  -s <slack_channel> -e <email_address>

### How it works
* User passes Rabbit MQ account information through command line
* Using Rabbit MQ API call, get all the queue informations and filter out all the unwanted queues such as any queue starts with ‘celery’ or any queue for which there is no message waiting in the queue
* For the queues selected, get the following information
* * How many jobs are waiting in the queue (‘message_ready’)
* * How many jobs that are presently running (‘unack’)
* If jobs_waiting>0 and jobs_running=0, send an alert for “possibly queue inactive”
* If jobs_running>0, query Elasticsearch  to find out the information about last job started. If job started longer than periodicity time (passed by user through command line, presently set for 4 hours), send an alert about ‘possible job stacked’


## Job Mionitoring
It monitors if a particular job type does not have any successful job completion within a ceratin period of time

### Source Code 
check_job_periodicity_combined.py

### Command 
python check_job_periodicity_combined.py <job_type> <periodicity> -s <slack_channel> -e <email_address>


### Jobs presently monitoring
* job-sling
* job-s1_orbit_crawler
* job-s1_calibration_crawler
* job-csk_scrape
* job-csk_incoming
* job-spyddder-extract (for CSK)

### How it works
* User passes job type (with release information) and time limit  through command line
* Check for “Successfully Completed” jobs: Query Elastic search, for information about the last job successfully completed.
* * If the last job successfully completed time is within the time range, NO alert is sent. The program exits.
* * If no information is found and if the job type is NOT ‘csk-incoming’ or ‘csk extract’, add in the error report that “No successfully completed job found” and go to next step “Check for Failed Jobs”.
* * If the last job “successfully completed” time is beyond the time range and if the job type is NOT ‘csk-incoming’ or ‘csk extract’, record the last successfully completed jobs end time in the error report and go to next step “Check for Failed Jobs”.
* * If the job-type is ‘csk-incoming’ or ‘csk extract’, just go to next step “Check for Failed Jobs”. (“csk-incoming” and “csk-extract” job does not run unless there is a new csk dataset available, so it is not an error)
* Check for “Failed” jobs: Query Elastic search, for information about the last job failed.
* * If no information is found and if the job type is NOT ‘csk-incoming’ or ‘csk extract’, add in the error report that “No failed job found” for that job type.
* * If the job type is ‘csk-incoming’ or ‘csk extract’ and the last job failed time is BEYOND the time range, no alert is sent and the program exists.
* * Otherwise, get the information about the last job that failed, add to the error report (This is true for job type ‘csk-incoming’ and ‘csk extract’ too as long as the last job failed time is WITHIN the time range)
* Send the alert: Create a detailed error report containing all the messages added to it and send the alert to the slack channel.

