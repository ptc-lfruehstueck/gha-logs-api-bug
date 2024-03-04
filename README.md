# Bug in "Download job logs for a workflow run" REST API endpoint

The endpoint `/repos/{owner}/{repo}/actions/jobs/{job_id}/logs`
is documented as: `Gets a redirect URL to download a plain text file of logs for a workflow job.`

As the request URL contains the Job ID, I assume this endpoint is supposed to return the logs for this particular job.

However, it returns the logs for the latest attempt of that job in the same workflow run (which is a different job id)

## Example:
The [test-workflow.yml](https://github.com/ptc-lfruehstueck/gha-logs-api-bug/blob/master/.github/workflows/test-workflow.yml) in this repo has a job that prints its own ID, the workflow run ID and the run attempt number.

[This workflow run (8140576594)](https://github.com/ptc-lfruehstueck/gha-logs-api-bug/actions/runs/8140576594) has 3 run attempts:
* [Attempt 1](https://github.com/ptc-lfruehstueck/gha-logs-api-bug/actions/runs/8140576594/attempts/1) = Job ID 22246027899
* [Attempt 2](https://github.com/ptc-lfruehstueck/gha-logs-api-bug/actions/runs/8140576594/attempts/2) = Job ID 22246056671
* [Attempt 3](https://github.com/ptc-lfruehstueck/gha-logs-api-bug/actions/runs/8140576594/attempts/3) = Job ID 22246067732

When using the REST API to query logs for attempt 1, the logs for attempt 3 are returned instead:
```bash
# Get Job ID for attempt 1:
gh api /repos/ptc-lfruehstueck/gha-logs-api-bug/actions/runs/8140576594/attempts/1/jobs | jq -r '.jobs[0].id'
# Result: 22246027899

# Get logs for this job:
gh api /repos/ptc-lfruehstueck/gha-logs-api-bug/actions/jobs/22246027899/logs | grep This | grep -v echo
# Result: 2024-03-04T12:59:08.7103197Z This is job 22246067732 from workflow run 8140576594 attempt 3
```
