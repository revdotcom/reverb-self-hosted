
# Knowledge Requirements

This guide is written for developers and is best understood by those with previous knowledge of the following technologies:

- Docker
- REST API
- Redis
- Postman 

### Remarks
We recommend using the On Prem solution with an Intel architecture.

# Downloading the Image

The images are under the public Docker Hub repository, `revdotcom/revai-onprem`.

```
$ docker pull revdotcom/revai:<TAG_VERSION>
```

## Available Tags

The On Prem deployment is split into three images: `Workers` and `Gateway`.
As of this documentation, the following tags are the latest available tags:
- open-gateway-3.15.0
- open-workers-3.21.0
- open-en-workers-3.21.0

If you encounter any issues when downloading please reach out to our team by email at `support@rev.ai`.

# Environment Variables
## Gateway Environment Variables

### Required Gateway Environment Variables

| Variable | Type | Description |
|---|---|---|
| `Initialization__AccessToken` | string | Your Rev AI account access token. A valid token must be provided in order for the container to authenticate. |
| `Redis__Endpoints` | string | Endpoints for the Redis instance. |
| `Redis__KeyPrefix` | string | Prefix to given to all keys stored into Redis for this application to maintain key isolation across shared environments. |
| `Redis__Database` | number | Which database Redis should connect to. Should be the same as the `RedisConfig__Database` value provided to the `Workers`. |
| `Revspeech__CompletionQueue` | string | Key prefix for the Redis-based queue that the `Gateway` pulls jobs from. |

#### Queue Environment Configuration

At least one of the following `Worker` submission queue environment variable is required. This ensures that the `Gateway` is able to submit transcription jobs.

| Variable | Type | Default | Description |
|---|---|---|---|
| `Revspeech__SubmissionQueue` | string | null | Key prefix for the Redis-based queue that the `Gateway` submits English jobs to. Should be configured the same as the `InboundQueueName` environment variable in the English `Workers` deployment. |

### Optional Environment Variables

| Variable | Type | Default | Description |
|---|---|---|---|
| `Initialization__CallbackUrl` | string | null | Callback URL for `POST` request, made when the container initialization succeeds or fails. |
| `Initialization__Attempts` | number | 5 | The number of attempts made to reach the initialization callback before failure. |
| `Initialization__RetryDelayMs` | number | 1000 | The amount of time in milliseconds between each attempt to POST to the initialization callback. |
| `Billing__Endpoint` | string | `https://api.rev.ai/external/v2open/billing` | The provided endpoint is responsible for forwarding the billing request to `https://api.rev.ai/external/v2open/billing`. |
| `Logging__DefaultLogLevel` | number | 2 | The verbosity of the Docker logs. <br/> 0 = TRACE <br/> 1 = DEBUG <br/> 2 = INFO <br/> 3 = WARN <br/> 4 = ERROR |
| `Logging__LogTemplate` | string | null | Default = Json format, non-indented. <br/> Sample text format: `[\{Timestamp:HH:mm:ss} \{Level:u3}] {Message:lj} {NewLine} {Properties}{NewLine}`. |
| `Redis__EnableTls` | boolean | false | Whether Redis requests are required to be sent via SSL. |
| `Redis__Password` | string | null | Password used to connect to the provided Redis instance. |
| `Revspeech__MaxTranscriptionTimeSeconds` | number | 7200 | The max number of seconds a transcription job can take to perform the transcription step. Must be between 300 and 7200, and less than `Revspeech__MaxJobLifetimeSeconds`. |
| `Revspeech__MaxJobLifetimeSeconds` | number | 8100 | The max job lifetime in seconds for the job to complete its worflow. Must be between 600 and 82800, and greater than `Revspeech__MaxTranscriptionTimeSeconds`. |
| `Revspeech__MediaAuthorizationHeader` | string | null | Authorization header value used when accessing the `media_url` provided. Should be configured the same as the `MediaAuthorizationHeader` environment variable in the `Worker` deployment. If provided, the value is added as an `Authorization` header when downloading the provided `media_url` during transcription. |
| `Revspeech__EnableMediaUrlManualRedirect` | bool | false | Handles media url redirects manually by parsing the returned `Location` header from the provided `media_url` to `ffprobe`. By default, `ffprobe` passes along all headers, including `Authorization` headers, so if the redirect url needs to have headers stripped (as in the case of signed Azure blob urls), then set this flag to `true`. `Runner__MediaAuthorizationHeader` will only be passed to the initial call to `media_url` to retrieve the redirect `Location` header instead. |
| `Metrics__Queue__LoggingIntervalSeconds` | int | 5 | The amount of time in seconds between each publishing of Prometheus metrics. Must be between 1 and 300, inclusive. |
| `ShutdownTimeoutSeconds` | number | 300 | The number of seconds after shutdown has been initiated that background tasks wait to finish. Bear in mind that Docker has its own shutdown timeout defaulted to 10 seconds, so that will be needed to increase whenever `ShutdownTimeoutSeconds` is increased. |
| `Storage__Prefix` | string | "gateway" | Optional prefix path to give to all Storage uploads. Allows for environment isolation. Cannot start nor end with a '/'. |

## Workers Environment Variables (CPU)

### Required Environment Variables

| Variable | Type | Description |
|---|---|---|
| `AccessToken` | string | Your Rev AI account access token. A valid token must be provided in order for the container to authenticate. |
| `RedisConfig__Endpoints` | string | Endpoints for the Redis instance. |
| `RedisConfig__KeyPrefix` | string | Prefix to given to all keys stored into Redis for this application to maintain key isolation across shared environments. |
| `RedisConfig__Database` | number | Which database Redis should connect to. Should be the same as the `Redis__Database` value provided to the `Gateway`. |
| `InboundQueueName` | string | Key prefix for the Redis-based queue that the worker instance pulls job from. Should be configured the same as the `Revspeech__SubmissionQueue` environment variable in the `Gateway` deployment. |

### Optional Environment Variables

| Variable | Type | Default | Description |
|---|---|---|---|
| `InitializationCallbackUrl` | string | null | Callback URL for `POST` request, made when the container initialization succeeds or fails. |
| `Runner__CountThreads` | number | 1 | The number of workers available for the container to process job chunks. Each worker can process one chunk at a time so the higher the number of workers the more chunks can be processed in parallel. |
| `Runner__LogLevel` | number | 2 | The verbosity of the Docker logs. <br/> 0 = TRACE <br/> 1 = DEBUG <br/> 2 = INFO <br/> 3 = WARN <br/> 4 = ERROR |
| `InitializationAttempts` | number | 5 | The number of attempts made to reach the initialization callback before failure. |
| `InitializationRetryDelayMs` | number | 1000 | The amount of time in milliseconds between each attempt to POST to the initialization callback. |
| `MediaAuthorizationHeader` | string | null | Authorization header value used when accessing the `media_url` provided. Should be configured the same as the `Revspeech__MediaAuthorizationHeader` environment variable in the `Gateway` deployment. If provided, the value is added as an `Authorization` header when downloading the provided `media_url` during transcription. |
| `OnPremBillingEndPoint` | string | `https://api.rev.ai/external/v2open/billing` | The provided endpoint is responsible for forwarding the billing request to `https://api.rev.ai/external/v2open/billing`. |
| `Runner__LogTemplate` | string | null | Default = Json format, non-indented. <br/> Sample text format: `[\{Timestamp:HH:mm:ss} \{Level:u3}] {Message:lj} {NewLine} {Properties}{NewLine}`. |
| `Runner__LogLevelOverrides__<namespace>` | number | null | Namespace specific log-level overrides. <br/> 0 = TRACE <br/> 1 = DEBUG <br/> 2 = INFO <br/> 3 = WARN <br/> 4 = ERROR |
| `RedisConfig__EnableTls` | boolean | false | Whether Redis requests are required to be sent via SSL. |
| `RedisConfig__Password` | string | null | Password used to connect to the provided Redis instance. |
| `ShutdownTimeoutSeconds` | number | 300 | The number of seconds after shutdown has been initiated that background tasks wait to finish. Bear in mind that Docker has its own shutdown timeout defaulted to 10 seconds, so that will be needed to increase whenever `ShutdownTimeoutSeconds` is increased. |
| `ASPNETCORE_URLS` | string | `http://+:80` | ASP .Net Core environment variable to control the port which the application runs under |
| `StorageConfig__Prefix` | string | "workers" | Optional prefix path to give to all Storage uploads. Allows for environment isolation. Cannot start nor end with a '/'. |
| `MetricsConfig__Prometheus__Summary__Rtf__QuantileEpsilonPairsString` | string | `0.5,0.01;0.9,0.01;0.99,0.01` | Optional string representation of Prometheus Summary quantile and epsilon pairs to record Prometheus Summary quantiles on RTF with. It should be semicolon-delimited, with a comma separating the quantile and epsilon values, ie. '0.5,0.01;0.9,0.01;0.99,0.01' represents the quantile/epsilon pairs [0.5, 0.01] [0.9, 0.01] [0.99, 0.01]. |
| `MetricsConfig__Prometheus__Summary__SourceFileDownloadTimeSeconds__QuantileEpsilonPairsString` | string | `0.5,0.01;0.9,0.01;0.99,0.01` | Optional string representation of Prometheus Summary quantile and epsilon pairs to record Prometheus Summary quantiles on source file download time in seconds with. It should be semicolon-delimited, with a comma separating the quantile and epsilon values, ie. '0.5,0.01;0.9,0.01;0.99,0.01' represents the quantile/epsilon pairs [0.5, 0.01] [0.9, 0.01] [0.99, 0.01]. |
| `MetricsConfig__Prometheus__Summary__SourceFileSizeKb__QuantileEpsilonPairsString` | string | `0.5,0.01;0.9,0.01;0.99,0.01` | Optional string representation of Prometheus Summary quantile and epsilon pairs to record Prometheus Summary quantiles on source file size in kb with. It should be semicolon-delimited, with a comma separating the quantile and epsilon values, ie. '0.5,0.01;0.9,0.01;0.99,0.01' represents the quantile/epsilon pairs [0.5, 0.01] [0.9, 0.01] [0.99, 0.01]. |

# Networking

The `Gateway` and `Workers` require an open network connection for authentication and initialization.

If instead you would prefer to whitelist specific URLs, the containers require communication with the following:

- `https://api.rev.ai/external/v2open/initialize`
- `https://api.rev.ai/external/v2open/billing`

## Initialization

The `POST` request to `/initialize` is a required licensing request to Rev.ai's servers. It verifies the On Prem containers' license against your Rev.ai account. The request contains the following information:

`access_token` - Customer's token provided to verify that they are signed up to use On Prem and to tell the container which account to charge to.

`container_key` - A unique container key identifying the On Prem container image hardcoded into the container itself.

`container_version` - The version of the container that is sending the initialization request, this is equivalent to the tag of the container image.

Example request:

```
Authorization: Bearer <YOUR_ACCOUNT_TOKEN>

{
    "container_key":"UnIqUeKeY",
    "container_version":"gateway-0.1.0"
}
```

Log Message on Successful Initialization:
```
2021-09-14 06:56:11.568	 Initialization response user token is "v3|fV5_-QZjdRV-DWdRb-EUBtk7TqruQkQvVSmhg4jPU7XYkc79qxFzzevUzsEVfsPTNfQB6g"
```

**We maintain there will be no customer data ever in this request**

## Billing

Both the `Gateway` and `Workers` make a `POST` request to `/billing`. The billing requests from the `Workers` are used for logging purposes by the Rev AI team for accounting. The billing request from the `Gateway` contains the actual duration of the overall job. The requests contain the following:

**Headers**

`x-rev-signature` - Signature which verifies the integrity of the request.

`x-rev-billing-type`- Billing request type, can be `api` or `worker`.

**Body**

`duration` - A numeric value for that either contains the total duration in seconds of the job or a duration for accounting purposes.

`reference_id` - A universally unique identifier.

`request_sent_on` - A date time stamp of when this request was made.

`revaiapi_endpoint` - An http url that the customer can use to forward the request to if they wish to implement a sidecar to inspect this billing request before it reaches Rev AI's server.

`user_token` - A string in the following format: `v3|someString`. See initialization log message above for example.

`language` - Language code of the job. Currently defaults to `en`.

`metadata` - Metadata provided by the customer on the job. Only present in the `Gateway` billing request

`job_details` - Serialized metadata object provided by the `Gateway` about the job. Only present in the `Gateway` billing request.

Request example:

```
x-rev-signature: Example+hash+token=
x-rev-billing-type: api
{
    "reference_id": "example-1111-1111-1111",
    "duration": 5.87,
    "request_sent_on": "2020-03-30T04:29:59.7351045Z",
    "revaiapi_endpoint": "https://api.rev.ai/external/v2open/billing",
    "user_token": "v3|someuniquestring",
    "language": "en",
    "metadata": "My custom metadata",
    "job_details": "\{ \"type\": \"async\" \}"
}
```

### Response

The response from Rev AI confirms the billing request. On the `Gateway` side, this clears the container to release the transcript.

`x-rev-signature` - Signature which verifies the integrity of the request.

`reference_id` - A universally unique identifier.

Response example:

```
x-rev-signature: Example+hash+token=
{
    "reference_id": "example-1111-1111-1111",
}
```

# Resource Requirements

## Gateway Resources

The `Gateway` acts as an API server and does workflow processing of each job. A minimum of 1 core and 4GB of memory is required, however we recommend the `Gateway` to be run on 4 physical cores and 16GB of memory.

## Workers Resources

### CPU and Memory Requirements

The number of resources required varies depending on the number of workers each container has been initialized with using the `Runner__CountThreads` variable. The number of workers dictates the number of chunks that can be transcribed in parallel. Each job that is submitted to the `Gateway` is divided into 3 minute chunks. i.e. if the submitted job is 5 minutes long, there will be 2 chunks for workers to work on. Having 2 workers will enable parallelized processing.

**The Rev AI recommended configuration is to run 4 workers on a AWS C5.2xlarge equivalent machine (4 physical cores, 16 GB memory). All resources must be provided to the container. Any other configuration should be tested against production workloads by the customer to confirm that it is stable.**

As a base, using a single speech-to-text worker, the container requires at least 5 GB of RAM. The table below provides information on the *MINIMUM* memory requirements for up to 4 workers. In the event you wish to run a non-standard configuration, you can use this a baseline. However it is guaranteed the image will require more memory than the baseline amount to process the job. A buffer of at least 2x the minimum memory is recommended.

| Worker Count | Minimum Memory |
| ---          | ---            |
| 1 | 5 GB |
| 2 | 6 GB |
| 3 | 7 GB |
| 4 | 8 GB |
| x | (1x + 4) GB |

### Additional Memory Requirements When Using Custom Vocabularies

When custom vocabularies are provided, each worker require about 60MB of additional memory to perform transcription. This additional memory is expected to stay roughly constant, regardless of the input custom vocabulary size.

### Disk Requirements

The workers will download the provided `media_url` into local storage before performing transcription. While files are cleaned up after processing, it is recommended to have sufficient disk space attached to each container to accomodate the number of parallel workers (`Runner__CountThreads`) and possible parallel jobs that the container can be running at any given time.

**Rev AI attaches a 100GB volume mount to each container we run with Runner__CountThreads=4.**

## Redis Requirements

Intermediate transcript files and job data are stored in the provided Redis instance for a period of 24 hours. The Redis instance must be provisioned with enough memory to handle the number of jobs over a 24 hour period. Users can manually clear the space in Redis by deleting jobs once they complete and have received their transcripts.

**~1350 hours of transcribed audio can be stored in a 16GB Redis instance over a 24 hour period.**

# Starting the Containers

There are multiple ways to start the containers: `docker run`, `docker-compose` or `kubernetes`. In this guide we will provide examples for `docker run`.

## Starting From the Docker Run Command

From the command line run the following:

```
# Gateway
$ docker run -p <YOUR_HOST_PORT:80> -e Initialization__AccessToken=<YOUR_REV_AI_ACCESS_TOKEN> -e Redis__Endpoints=<YOUR_REDIS_ENDPOINT> -e Redis__KeyPrefix=<YOUR_KEY_PREFIX> -e  Redis__Database=<YOUR_REDIS_DATABASE> -e Revspeech__SubmissionQueue=revspeech-submission -e ReWhisper__Transcription__DefaultTranscriber__SubmissionQueueUrl=revwhisper-submission -e Revdiarization__SubmissionQueue=revdiarization-submission -e Revspeech__CompletionQueue=revspeech-completion
revdotcom/revai:gateway-<TAG_VERSION>

# Workers
$ docker run -e AccessToken=<YOUR_REV_AI_ACCESS_TOKEN> -e RedisConfig__Endpoints=<YOUR_REDIS_ENDPOINT> -e RedisConfig__KeyPrefix=<YOUR_KEY_PREFIX> -e  RedisConfig__Database=<YOUR_REDIS_DATABASE> -e InboundQueueName=revspeech-submission revdotcom/revai:workers-<TAG_VERSION>
```

## Container User
For the sake of security, the containers by default run under non-root users predefined by the image. For certain operating systems, non-root users by default are not able to bind port 80 for `localhost`, which the application requires by default.

If network or OS configurations prevent this, you can change the default application port by providing the following environment variable: `--emv ASPNETCORE_URLS=http://+:<custom_port>` on `docker run`. Optionally you can also run the docker as root user: `-u root` 

## File Storage

Due to the nature of having persistent files over a 24 hour period, an object based file storage system is required to be configured as transcripts and other intermediate outputs from the `Workers` are stored and referenced to. Examples of cloud-based file storages include S3 and Azure blob storage. The file storage needs to support the following:

- Uploading of files/objects referenced via a key/path structure
- Retrieving of files/objects referenced via a key/path structure
- Deleting of files/objects referenced via a key/path structure

The `Gateway` and `Workers` by default have a Redis-backed file storage system implementation built in.

**Blob Storage Prefix and Lifecycle**

Each image will store their files with a given Blob prefix. The `Gateway` will have its files in the `<Blob_Container_Name>` container with the `/gateway` as the path prefix to all Blobs. `Workers` will have `/workers` as it's prefix. By default, these files are not cleaned up automatically. **It is highly recommended to put a lifecycle rule on the Blob Container to auto-delete up these Blob prefixes.**

The `Gateway` and `Workers` by default have a Redis-backed database implementation built in. **Note this database's items expire after 24 hours.**

## Message Queue

Communication between the two services is done via a messaging queue. The queue and messages supports the following:

- Receive Message
- Delete Message
- Set Message Invisibility
- Refresh Message Invisibility
- Message Expiration
- Best effort ordering

The `Gateway` and `Workers` by default have a Redis-backed message queue system implementation built in.

## License Expiry

[TODO]

# Using On Prem

## When Are the Containers Ready?

Upon startup the `Gateway` and `Workers` containers will attempt to authenticate with Rev AI by sending a `POST` request to the `https://api.rev.ai/external/v2open/initialize` endpoint. After the container receives a response, if the `Initialization__CallbackUrl` or `InitializationCallbackUrl` are provided, the containers will then send a POST request to that url. This `POST` contains the initialization status.

Successful `POST` body:
```
{ "success": true }
```

Failed `POST` body:
```
{ "success": false }
```

## Submitting a Job for Transcription

Once both the `Gateway` and `Workers` images are running, you can submit a file by making a `POST` request to `/speechtotext/v1/jobs` endpoint of the `Gateway` container with the proper parameters in the request body. Once the job is submitted, the user can query the status of the job and transcript via the API exposed in the `Gateway`. Outlined in the section titled `API Reference` is an overview of the available endpoints and features for On Prem V2. As On Prem V2 is meant to keep near feature parity with the Cloud API, the precise API and model documentation can be found in the [Cloud API Reference](https://rev.ai/docs).

## Gateway Workflow

When the job is created, it is stored into the Redis instance and is available to be queried for 24 hours before expiring (if not deleted). During this process, the job status can be:
`in_progress`, `transcribed` or `failed`. The job will be placed into a workflow where it will be processed for transcription. During this workflow phase, any unexpected failures will result in a retry, until the 2 hour transcription expiration is reached. At this point, if the job has not yet completed, the job will be failed with `internal_processing`.

## Audio Duration Detection

Internally, the `Gateway` uses [Ffprobe](https://ffmpeg.org/ffprobe.html) and the audio's metadata to determine the length of the audio. **On Prem V2 has a audio duration maximum of 15 hours, and a minimum of 2 seconds.**. A corrupted or malformed audio where the audio duration cannot be detected may result in the job being failed prematurely.

## Chunking

On Prem uses chunking to perform transcription. Chunking means the `Workers` will process 3 minute segments of the provided audio and parallelize the transcription step across these 3 minute chunks. The resultant individual transcripts are then combined into a singluar transcript. This drastically reduces the RTF of transcription. You can expect to recieve an hour long audio file to be transcribed in about 3-5 minutes so long as there are enough `Workers` to handle all the individual chunks.

```
# Log message from the Gateway when submitting chunks
2021-09-14 07:33:53.437	Submitting 4 chunks
```

## Message Queue

Before jobs are picked up by the `Workers`, the messages are submitted to a message queue. `Workers` read from the queue, however the jobs are not removed from the queue right away. They are instead marked as invisibile until the `Workers` finish processing the job. During the processing of the message, the `Workers` will refresh this invisibility, as it expires in 5 minutes otherwise. Once the `Workers` finish processing the job, they delete the job from the message queue. In the case that the `Workers` crash or an unexpected failure occurs, the message invisibility will expire in 5 minutes and the message be picked up by another `Workers` instance if available.

```
# Log message from the Workers regarding message refresh
2021-09-14 06:57:50.000	Refreshed the message "cec74fc6-8561-4a78-bf14-92af5560c527" from inbound queue

```

## Speech-to-Text Workers

Each `Workers` application maintains set stateless speech-to-text `worker` threads. The number of `worker` threads can be configured by setting `Runner__CountThreads`.

The speech-to-text `worker` performs the actual speech-to-text processing. The `media_url` provided is first downloaded and then transcoded using [Ffmpeg](https://ffmpeg.org/ffmpeg.html). Once processing, the job will continue either to failure or success. In the case of terminal failures, like invalid media or empty audio duration, the failure will propagate back to the `Gateway` and the entire job will be failed. In the case of retriable failures, e.g instance shutdown, `worker` will mark the job for a retry so another `worker` can process it. In either sucess or failure case, the `worker` submit a response message back to the `Gateway's` configured `Revspeech__CompletionQueue`.

On successful transcription, the `worker` will upload their results to file storage.

```
# Sample log output for a sucessful job, aggregated by RevRequestId

2021-09-14 06:56:11.568	Message received with delay 6.29 seconds
2021-09-14 06:56:11.578	Received SQS message
2021-09-14 06:56:11.600	skipping connection for inactive worker "postprocessing"
2021-09-14 06:56:11.601	Claiming worker id: "Ctm-968bb075-a1ba-42c0-af80-9cfc5181dae4" "3"
2021-09-14 06:56:11.602	skipping connection for inactive worker "secondpd"
2021-09-14 06:56:11.602	Acquired "ctm" worker with WorkerId: "Ctm-968bb075-a1ba-42c0-af80-9cfc5181dae4_3"
2021-09-14 06:56:11.605	Created transcription log: "/tmp/tmpjOh7j6.tmp"
2021-09-14 06:56:11.636	Refreshed the message "ddac8e67-07d8-41b7-b535-3e5f22548212" from inbound queue
2021-09-14 06:56:11.679	"Download" 19631KB file too fast to measure
2021-09-14 06:56:11.833	"Copy file stream" 19631KB in 0.1536749: 127744 KB/sec
2021-09-14 06:56:11.845	Process has been started
2021-09-14 06:56:11.926	Found EOF in ffprobe stdout
2021-09-14 06:56:11.938	Process has been started
2021-09-14 06:56:12.013	Process has been started
2021-09-14 06:56:12.034	Found EOF in ffprobe stdout
2021-09-14 06:56:12.043	Starting CV handling "completed" in 0.0 sec
2021-09-14 06:56:12.043	Starting ctm
2021-09-14 06:56:12.043	Tracking job completion time, estimated duration: 135 seconds
2021-09-14 06:57:37.195	Received workerResponse with status Success
2021-09-14 06:57:37.195	Ctm ran for 85 seconds for 180 seconds audio
...
2021-09-14 06:57:37.358	Sending response SQS message
2021-09-14 06:57:37.358	Job completed
```

```
# Log when the job is canceled due to shutdown and scheduled for retry

2021-09-14 06:57:37.195 Job execution was canceled
```

## Webhooks/Callbacks

If a `callback_url` is provided, the `Gateway` workflow will `POST` to the `callback_url` when the entire job has been processed. The callback will be retried for 24 hours in the case of non-2xx responses. A `410` response from the `callback_url` will also indicate that the callback should not be retried.

### Callback Body

```
# Successful Job Callback Body
{
    "job": {
    "id": "Umx5c6F7pH7r",
    "status": "transcribed",
    "created_on": "2018-05-05T23:23:22.29Z",
    "callback_url": "https://www.example.com/callback",
    "duration_seconds": 356.24,
    "media_url": "https://www.rev.ai/FTC_Sample_1.mp3"
    }
}
```

```
# Failed Job Callback Body
{
    "job": {
    "id": "Umx5c6F7pH7r",
    "status": "failed",
    "created_on": "2018-05-05T23:23:22.29Z",
    "callback_url": "https://www.example.com/callback",
    "failure": "internal_processing",
    "failure_detail": "Unexpected error has occured."
    }
}

```

```
# Log when callback is scheduled and completed

2021-09-14 07:08:25.759	scheduling webhook invocation: "2f219fee-5c9e-4093-a9ef-62550422b979"/"rWRa02HX2dtcZ"
...
2021-09-14 7:08:26.492	Sent callback, success: True, server returned 200
```

## Prometheus Metrics

The On Prem containers publish the following metrics to their `/metrics` endpoint, which can be used by [Prometheus](https://prometheus.io/docs/introduction/overview/).

### Gateway

##### Queue Metrics

The following Redis-based Queue metrics are queried and published on a interval defined by the `Gateway` variable `Metrics__Queue__LoggingIntervalSeconds`. Decreasing this value will causing the metrics to be more frequently updated at the cost of higher load on the Redis instance and `Gateway`.

- `Gateway` queue total items (`gateway_total_item_count`)
- `Gateway` queue invisible (in progress) items (`gateway_invisible_item_count`)
- English `Workers` queue total items (`workers_total_item_count`)
- English `Workers` queue invisible (in progress) items (`workers_invisible_item_count`)

#### Gateway External Request Failure Metrics

    - Number of billing request failures (`gateway_billing_request_failure_{statusCodeInt}`)
    - Number of billing response verification failures (`gateway_billing_response_verification_failure`)
    - Number of failures from invoking the initialization callback after a successful initialization (`gateway_initialization_success_callback_failure_{statusCodeInt}`)
    - Number of failures from invoking the initialization callback after a failed initialization (`gateway_initialization_failure_callback_failure_{statusCodeInt}`)
    - Number of failures from invoking the job completion webhook (`gateway_job_completion_webhook_failure_{statusCodeInt}`)

These metrics that contain `statusCodeInt` are labeled with `status_code_3xx`, `status_code_4xx`, and `status_code_5xx` failures based on their failure codes

### Workers

#### Worker Count Metrics

The number of speech-to-text workers initialized are recorded by the `Workers` containers. They are as follows:

- Number of CTM workers initialized (`worker_count_ready_ctm`)
- Number of diarization workers initialized (`worker_count_ready_diarization`)
- Number of post processing workers initialized (`worker_count_ready_postprocessing`)
- Total number of worker sets initialized. Every job requires 1 worker set to process (`worker_count_ready_set`)

#### Worker Transcription Failure Metrics

- Number of transcoding failures (`workers_job_failure_transcoding`)
- Number of transcribing failures (`workers_job_failure_transcribing`)
- Number of duration exceeded failures (`workers_job_failure_duration_exceeded`)
- Number of invalid audio channel failures (`workers_job_failure_invalid_audio_channel`)
- Number of internal processing failures (`workers_job_failure_internal_processing`)
- Number of invalid languages failures (`workers_job_failure_invalid_languages`)

#### Worker Jobs In Progress Metrics
- Number of jobs in progress (`jobs_in_progress`)
- Number of Ctm jobs in progress (`ctm_jobs_in_progress`)
- Number of Second Pass Diarization jobs in progress (`secondpd_jobs_in_progress`)
- Number of Itn jobs in progress (`itn_jobs_in_progress`)

### Worker Ctm Rtf Summary Metric
- Quantiles of Ctm rtf values (`workers_ctm_job_rtf`, `workers_ctm_job_rtf_count`, `workers_ctm_job_rtf_sum`)

### Worker Source File Download Metrics
- Quantiles of source file download time in seconds (`source_file_download_time_seconds`, `source_file_download_time_seconds_count`, `source_file_download_time_seconds_sum`)
- Quantiles of source file size in kb (`source_file_size_kb`, `source_file_size_kb_count`, `source_file_size_kb_sum`)

#### Worker External Request Failure Metrics

- Number of billing request failures (`workers_billing_request_failure_{statusCodeInt}`)
- Number of billing response verification failures (`workers_billing_response_verification_failure`)

#### Instance Unhealthy Metric

- A value of > 0 indicates that this instance has encountered an un-recoverable failure and should be restarted (`workers_is_unhealthy`)

These metrics that contain `statusCodeInt` are labeled with `status_code_3xx`, `status_code_4xx`, and `status_code_5xx` failures based on their failure codes


# Troubleshooting

## Job Failures

Job failures can be categorized into expected, unexpected, and terminal failures. Of the following failure types:

**Expected**

These failures can occur when the provided media is in an invalid state to be transcribed, whether due to the file
being either too short, too long, or a non-audio file.

- `duration_exceeded`
- `duration_too_short`
- `invalid_media`
- `empty_media`

**Unexpected**

These failures occur during the normal flow of processing. These are usually due to misconfiguration, but are not deemed
terminal as most of the process is still functioning. An investigation into the logs of the containers is usually required.

- `transcription`
- `billing`

**Terminal**

These failures indicate a greater, system-wide issue with the `Workers` containers, usually unrecoverable. A restart
of the container plus additional investigation is recommended. The `Workers` healthcheck endpoint will return 500 when this type of
error is encountered.

- `internal_processing`

## Logging

By default, the `Gateway`, `Streaming` and `Workers` log to standard output (stdout) in [Serilog](https://serilog.net/) format. It is highly recommended to capture the standard output and save them for troubleshooting.

```
# Note these logs samples are formatted JSON for readability. The application will output non-formatted JSON logs
{
    "Timestamp": "2021-09-14T17:47:06.6900487+00:00",
    "Level": "Information",
    "MessageTemplate": "Starting {serviceName}",
    "RenderedMessage": "Starting \"inboundQueueProcessor\"",
    "Properties": {
        "serviceName": "inboundQueueProcessor",
        "SourceContext": "Rev.Common.AspNetCore.Infrastructure.IntervalTaskHostedService`1[[Rev.Infrastructure.InboundQueue.IInboundQueueProcessor, Rev.Infrastructure.InboundQueue, Version=3.1.0.0, Culture=neutral, PublicKeyToken=null]]",
        "ThreadId": 4
    }
}
{
    "Timestamp": "2021-09-14T19:48:08.2238552+00:00",
    "Level": "Information",
    "MessageTemplate": "Starting ctm",
    "RenderedMessage": "Starting ctm",
    "Properties": {
        "SourceContext": "Rev.Revspeech.Transcription.TranscriptionJob`2[[Rev.Revspeech.Transcription.Requests.QueueTranscriptionJobRequest, Revspeech, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null],[Rev.Revspeech.Transcription.Responses.QueueTranscriptionJobResponse, Revspeech, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null]]",
        "estimatedCtmDurationSeconds": 135,
        "originalFileName": "SCHADE & MICHALUK audio_only.m4a",
        "jobId": "chunks.w.15873054.t.5493676.63c3471d-da64-41e3-881e-02bea6fe3e81.7",
        "sequenceId": null,
        "messageId": "ac4524ea-a1ef-41b2-a6c8-08b40dcb93ac",
        "RevRequestId": "2692232a-4e13-41ed-b22d-15eb3a540d2c",
        "RevRequestIdParent": "2692232a-4e13-41ed-b22d-15eb3a540d2c",
        "RevRequestIdRoot": "2692232a-4e13-41ed-b22d-15eb3a540d2c",
        "ThreadId": 15
    }
}
```

### Rev Request Id

Some of the log messages contain a series of `RevRequestId`. The `RevRequestId` track the context across each job execution. One can filter by the `RevRequestId` to get all of the logs for a specific job.

### Log Level

All log messages come with a log `Level` property. The ones exposed are `Info`, `Warning`, and `Error`. `Info` level logs are general level log messages that help trace application execution. `Warning` logs are logged when there is something that is potentially of concern and should be looked at. `Error` logs are unexpected and signal unexpected application state.

### Log Level Overrides

It is possible to override the level at which logs are printed, via the `Runner__LogLevelOverrides__<namespace>` property. By providing a specific namespace, you can override the log level for that specific namespace.
You can find the `<namespace>` on the log itself, under the `SourceContext` property

Examples:
```
Runner__LogLevelOverrides__Rev.Common.AspNetCore.Diagnostics.MemoryDiagnosticsService=3
Runner__LogLevelOverrides__Rev.Revspeech.Workers.WorkerProcessMonitoringHostedService=3
Runner__LogLevelOverrides__Rev.Revspeech.Workers.WorkerReadLimiterManager=3
Runner__LogLevelOverrides__Rev.Common.AspNetCore.Diagnostics.SwapMonitoringHostedService=3
```

## Gateway

### Submission

When a `POST` request is made, a job is created and is scheduled to be executed in a workflow. You should see this log message appear, as well as a successful 200 response.

```
# Sample Log Message
2021-09-14 12:28:33.011	Adding to seqqueue "revspeechapi"
```

```
# 200 Response
{
    "id": "Umx5c6F7pH7r",
    "status": "in_progress",
    "created_on": "2018-05-05T23:23:22.29Z",
    "type": "async"
}
```

If the endpoint returns a 500, you should check the logs for any exceptions thrown.

```
2021-09-13 07:33:13.700	Middleware uncaught exception "Exception of type 'ArgumentException'"
```

It is possible in the rare case that the job could be created but not scheduled during this step, A 200 success response is the indicator whether the job was successfully submitted and is processing. After 24 hours, the job will expire from storage.

### Workflow

The transcription job is scheduled to run through a series of steps before and after transcription. This is collectively know as the job `workflow`. The `workflow` has the following steps in this order:

1. Checking for transcription expiration
2. Preliminary Duration Detection
3. Transcription
4. Billing
5. Callback

Steps can be skipped depending on the state of the job. For example, if the job is already scheduled to be transcribed, a subsequent `workflow` will not submit the job for transcription. `Workflow` executions can fail at any step for any reason, however after a 10 minute delay, a scheduled `workflow` will be retried until 24 hours or the job is expired, whichever comes first.

```
2021-09-13 08:51:38.281	callback execution failed
```

### Processing Results

When the `Workers` finish processing, the intermediate files are sent to the `Gateway` via the queue (`Revspeech__CompletionQueue`) to be combined and processed into a final transcript. During this step, if the `Gateway` fails to read from the queue or fails to process the intermediate files, the item in the queue will follow invisibility timeout and be avaialble to be attempted to be processed again in 5 minutes. Once transcript is processed, another workflow is scheduled and follows the workflow processing rules.

```
2021-09-13 08:45:38.281 Read from the queue failed
...
2021-09-13 08:47:38.281 Failed while assemblying the chunked job result
...
2021-09-13 08:50:38.281 Exception while publishing the result
```

## Workers

### Processing Failures (Expected)

The `Workers` containers are a stateless application. Most failures will result in a failure message being returned to the `Gateway` and the job subsequently failed. There are expected errors like invalid media or invalid audio duration. In the expected cases, the failure log message `"Transcription failed"` will be logged at the `Info` level.

### Processing Failure (Unexpected)

In the case of unexpected failure when the `Workers` containers fail to return a response to the queue, the queue message being processed will have its invisibility timeout and be available to process again. A common message to see in this case is `"Message received with delay"`. A delay of of 300 seconds is expected as the invisibility timeout is set to 5 minutes. If the delay continues to grow larger and larger, this indicates that the message is continually failing to be processed unexpectedly.

```
2021-09-14 01:10:50.443	Message received with delay 300.1 seconds
```

## Streaming

### Processing Failures

The `Streaming` containers are a stateless application. All failures will result in a non 1000 close code as detailed above. Some errors such as `4002: Bad Request` are expected. To investigate any other failures we
recommend you find the RevRequestId for your streaming request and look at any associated logs from the `Streaming` and `Gateway` containers. Finding this RevRequestId is easiest done finding the associated API invocation log for your request.
These logs will look like:

```
2022-10-03 13:45:14.520	Invoked "GET" "https://api.rev.ai/speechtotext/v1/stream?access_token=abcd*&content_type=audio/x-raw;layout=interleaved;rate=48000;format=S16LE;channels=1&metadata=test", returned 101
```

The RevRequestId from this log can then be used to find all other logs associated with the stream from both the `Gateway` and `Streaming` containers.

### Job Logs

All jobs processed by the `Workers` also write logs to a tmp file on disk in addition to stdout. The location of the job log for the file can be found via the log messages. This job log can be useful for further debugging if needed.

```
2021-09-14 06:56:11.605	Created transcription log: "/tmp/tmpjOh7j6.tmp"
```

# Cloud Differences

## Gateway Sandbox
For integration testing purposes, we have the `Gateway` sandbox, which can be used to test the `Gateway` without the `Workers` running. The sandbox will be deployed as a seperate Docker image with the tag:
- sandbox-gateway-2.1.4

This `Gateway` sandbox will provide the same functionality as the regular `Gateway`, but will quickly return a dummy transcript regardless of audio provided. Jobs created by the `Gateway` sandbox will go through the same workflow, except the actual transcription step. Furthermore, all billing requests will by default be sent to a seperate billing endpoint:
- `https://api.rev.ai/external/v2/sandbox/billing`

## API

On Prem V2 tries to keep its API interface as similar to Rev AI's Cloud API as possible, however there are a few differences:

- No Access Token required per request. Instead Access Token is provided on container startup.
- `media_url` must be provided for submission. `multiform/form-data` submission does not exist.
- Jobs persist for a maximum of 24 hours instead of 30 days
- Maximum audio length is 15 hours instead of 17
- Allowed languages are restricted to those listed in `API Reference -Jobs` section [here](https://revai-onprem.redoc.ly/#operation/SubmitTranscriptionJob!path=language&t=request)
- `delete_after_seconds` is not supported in On Prem V2
- New job `failure` type: `billing_failure`
- There is no `GET /account` endpoint for On Prem V2

## Workflow

The workflow of a On Prem V2 has a few key differences compared to Rev AI's Cloud solution:

- On Prem V2 performs preliminary duration detection using ffprobe on the provided `media_url`. If the duration metadata cannot
be accurately detected from the `media_url`, the job will not be chunked correctly which may increase turn around time
- Transcript is not available until billing completes
- Maximum time the job can spend in the workflow is 2 hours and 15 minutes before it fails with `internal_processing`

## Streaming API

On Prem V2 tries to keep its API interface as similar to Rev AI's Cloud API as possible, however there are a few differences:

- No Access Token required per request. Instead Access Token is provided on container startup.
- Jobs are not created for Streaming requests and no information about the stream is persisted to any storage service
- Only allowed language is English
- Custom Vocabularies are not supported



## Logging

By default, the `Gateway` and `Workers` log to standard output (stdout) in [Serilog](https://serilog.net/) format. It is highly recommended to capture the standard output and save them for troubleshooting.

```
# Note these logs samples are formatted JSON for readability. The application will output non-formatted JSON logs
{
    "Timestamp": "2021-09-14T17:47:06.6900487+00:00",
    "Level": "Information",
    "MessageTemplate": "Starting {serviceName}",
    "RenderedMessage": "Starting \"inboundQueueProcessor\"",
    "Properties": {
        "serviceName": "inboundQueueProcessor",
        "SourceContext": "Rev.Common.AspNetCore.Infrastructure.IntervalTaskHostedService`1[[Rev.Infrastructure.InboundQueue.IInboundQueueProcessor, Rev.Infrastructure.InboundQueue, Version=3.1.0.0, Culture=neutral, PublicKeyToken=null]]",
        "ThreadId": 4
    }
}
{
    "Timestamp": "2021-09-14T19:48:08.2238552+00:00",
    "Level": "Information",
    "MessageTemplate": "Starting ctm",
    "RenderedMessage": "Starting ctm",
    "Properties": {
        "SourceContext": "Rev.Revspeech.Transcription.TranscriptionJob`2[[Rev.Revspeech.Transcription.Requests.QueueTranscriptionJobRequest, Revspeech, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null],[Rev.Revspeech.Transcription.Responses.QueueTranscriptionJobResponse, Revspeech, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null]]",
        "estimatedCtmDurationSeconds": 135,
        "originalFileName": "SCHADE & MICHALUK audio_only.m4a",
        "jobId": "chunks.w.15873054.t.5493676.63c3471d-da64-41e3-881e-02bea6fe3e81.7",
        "sequenceId": null,
        "messageId": "ac4524ea-a1ef-41b2-a6c8-08b40dcb93ac",
        "RevRequestId": "2692232a-4e13-41ed-b22d-15eb3a540d2c",
        "RevRequestIdParent": "2692232a-4e13-41ed-b22d-15eb3a540d2c",
        "RevRequestIdRoot": "2692232a-4e13-41ed-b22d-15eb3a540d2c",
        "ThreadId": 15
    }
}
```

### Rev Request Id

Some of the log messages contain a series of `RevRequestId`. The `RevRequestId` track the context across each job execution. One can filter by the `RevRequestId` to get all of the logs for a specific job.

### Log Level

All log messages come with a log `Level` property. The ones exposed are `Info`, `Warning`, and `Error`. `Info` level logs are general level log messages that help trace application execution. `Warning` logs are logged when there is something that is potentially of concern and should be looked at. `Error` logs are unexpected and signal unexpected application state.

### Log Level Overrides

It is possible to override the level at which logs are printed, via the `Runner__LogLevelOverrides__<namespace>` property. By providing a specific namespace, you can override the log level for that specific namespace.
You can find the `<namespace>` on the log itself, under the `SourceContext` property

Examples:
```
Runner__LogLevelOverrides__Rev.Common.AspNetCore.Diagnostics.MemoryDiagnosticsService=3
Runner__LogLevelOverrides__Rev.Revspeech.Workers.WorkerProcessMonitoringHostedService=3
Runner__LogLevelOverrides__Rev.Revspeech.Workers.WorkerReadLimiterManager=3
Runner__LogLevelOverrides__Rev.Common.AspNetCore.Diagnostics.SwapMonitoringHostedService=3
```

## Gateway

### Submission

When a `POST` request is made, a job is created and is scheduled to be executed in a workflow. You should see this log message appear, as well as a successful 200 response.

```
# Sample Log Message
2021-09-14 12:28:33.011	Adding to seqqueue "revspeechapi"
```

```
# 200 Response
{
    "id": "Umx5c6F7pH7r",
    "status": "in_progress",
    "created_on": "2018-05-05T23:23:22.29Z",
    "type": "async"
}
```

If the endpoint returns a 500, you should check the logs for any exceptions thrown.

```
2021-09-13 07:33:13.700	Middleware uncaught exception "Exception of type 'ArgumentException'"
```

It is possible in the rare case that the job could be created but not scheduled during this step, A 200 success response is the indicator whether the job was successfully submitted and is processing. After 24 hours, the job will expire from storage.

### Workflow

The transcription job is scheduled to run through a series of steps before and after transcription. This is collectively know as the job `workflow`. The `workflow` has the following steps in this order:

1. Checking for transcription expiration
2. Preliminary Duration Detection
3. Transcription
4. Billing
5. Callback

Steps can be skipped depending on the state of the job. For example, if the job is already scheduled to be transcribed, a subsequent `workflow` will not submit the job for transcription. `Workflow` executions can fail at any step for any reason, however after a 10 minute delay, a scheduled `workflow` will be retried until 24 hours or the job is expired, whichever comes first.

```
2021-09-13 08:51:38.281	callback execution failed
```

### Processing Results

When the `Workers` finish processing, the intermediate files are sent to the `Gateway` via the queue (`Revspeech__CompletionQueue`) to be combined and processed into a final transcript. During this step, if the `Gateway` fails to read from the queue or fails to process the intermediate files, the item in the queue will follow invisibility timeout and be avaialble to be attempted to be processed again in 5 minutes. Once transcript is processed, another workflow is scheduled and follows the workflow processing rules.

```
2021-09-13 08:45:38.281 Read from the queue failed
...
2021-09-13 08:47:38.281 Failed while assemblying the chunked job result
...
2021-09-13 08:50:38.281 Exception while publishing the result
```

## Workers

### Processing Failures (Expected)

The `Workers` containers are a stateless application. Most failures will result in a failure message being returned to the `Gateway` and the job subsequently failed. There are expected errors like invalid media or invalid audio duration. In the expected cases, the failure log message `"Transcription failed"` will be logged at the `Info` level.

### Processing Failure (Unexpected)

In the case of unexpected failure when the `Workers` containers fail to return a response to the queue, the queue message being processed will have its invisibility timeout and be available to process again. A common message to see in this case is `"Message received with delay"`. A delay of of 300 seconds is expected as the invisibility timeout is set to 5 minutes. If the delay continues to grow larger and larger, this indicates that the message is continually failing to be processed unexpectedly.

```
2021-09-14 01:10:50.443	Message received with delay 300.1 seconds
```

## Workflow

The workflow of a On Prem has a few key differences compared to Rev AI's Cloud solution:

- On Prem performs preliminary duration detection using ffprobe on the provided `media_url`. If the duration metadata cannot
be accurately detected from the `media_url`, the job will not be chunked correctly which may increase turn around time
- Transcript is not available until billing completes
- Maximum time the job can spend in the workflow is 2 hours and 15 minutes before it fails with `internal_processing`



# On Prem API Endpoints

### Paths

#### `/healthcheck`

- **GET**
  - **Summary**: Healthcheck
  - **Operation ID**: `Healthcheck`
  - **Description**: 
    Returns a 200 when the `Gateway` application is healthy and ready to receive requests. This endpoint is also available in the `Workers` application.
  - **Tags**: `API Reference - Healthcheck`
  - **Responses**:
    - **200**: Healthcheck Success

#### `/speechtotext/v1/jobs/{id}`

- **Parameters**:
  - `$ref: 'shared.yaml#/parameters/JobId'`

- **GET**
  - **Summary**: Get Job By Id
  - **Operation ID**: `GetJobById`
  - **Description**: Returns information about a transcription job.
  - **Tags**: `API Reference - Jobs`
  - **Responses**:
    - **200**: Transcription Job Details
      - **Content**: `application/json`
      - **Schema**: `$ref: '#/components/schemas/AsyncTranscriptionJob'`
      - **Examples**:
        - **New Job**: `$ref: '#/components/examples/NewJob'`
        - **Transcribed Job**: `$ref: '#/components/examples/TranscribedJob'`
        - **Failed Job**: `$ref: '#/components/examples/FailedJob'`
    - **404**: `$ref: 'shared.yaml#/responses/JobNotFound'`

- **DELETE**
  - **Summary**: Delete Job by Id
  - **Operation ID**: `DeleteJobById`
  - **Description**: 
    Deletes a transcription job. All data related to the job, such as input media and transcript, will be permanently deleted. A job can only be deleted once it's completed (either with success or failure).
  - **Tags**: `API Reference - Jobs`
  - **Responses**:
    - **204**: Job was successfully deleted.
    - **404**: `$ref: 'shared.yaml#/responses/JobNotFound'`
    - **409**: Bad Request
      - **Content**: `application/problem+json`
      - **Schema**: `$ref: 'shared.yaml#/schemas/InvalidStateDetails'`
      - **Examples**:
        - **In Progress Job**:
          ```json
          {
            "allowed_values": ["transcribed", "failed"],
            "current_value": "in_progress",
            "type": "https://rev.ai/api/v1/errors/invalid-job-state",
            "title": "Job is in invalid state",
            "detail": "Job is in invalid state to be deleted",
            "status": 409
          }
          ```

#### `/speechtotext/v1/jobs`

- **GET**
  - **Summary**: Get List of Jobs
  - **Operation ID**: `GetListOfJobs`
  - **Description**: 
    Gets a list of transcription jobs submitted within the last 24 hours in reverse chronological order up to the provided `limit` number of jobs per call. **Note:** Jobs older than 24 hours will not be listed. Pagination is supported via passing the last job `id` from a previous call into `starting_after`.
  - **Tags**: `API Reference - Jobs`
  - **Parameters**:
    - `$ref: 'shared.yaml#/parameters/JobListLimit'`
    - `$ref: 'shared.yaml#/parameters/JobListStartingAfter'`
  - **Responses**:
    - **200**: List of Rev AI Transcription Jobs
      - **Content**: `application/json`
      - **Schema**:
        - `type`: `array`
        - `items`: `$ref: '#/components/schemas/AsyncTranscriptionJob'`
    - **400**: Bad Request
      - **Content**: `application/json`
      - **Schema**: `$ref: 'shared.yaml#/schemas/BadRequestProblemDetails'`
      - **Examples**:
        - **Limit Above Max Value**: `$ref: 'shared.yaml#/responses/BadLimitResponse'`
        - **Invalid Job Id**: `$ref: 'shared.yaml#/responses/InvalidStartingAfterResponse'`

- **POST**
  - **Summary**: Submit Transcription Job
  - **Operation ID**: `SubmitTranscriptionJob`
  - **Description**: 
    Starts an asynchronous job to transcribe speech-to-text for a media file. Media files can be specified by including a public URL to the media in the transcription job `options`.
  - **Tags**: `API Reference - Jobs`
  - **Request Body**:
    - **Description**: Transcription Job Options
    - **Required**: `true`
    - **Content**: `application/json`
    - **Schema**: `$ref: '#/components/schemas/SubmitJobMediaUrlOptions'`
  - **Responses**:
    - **200**: Transcription Job Details
      - **Content**: `application/json`
      - **Schema**: `$ref: '#/components/schemas/AsyncTranscriptionJob'`
      - **Example**: `$ref: '#/components/examples/NewJob'`
    - **400**: Bad Request
      - **Content**: `application/problem+json`
      - **Schema**: `$ref: shared.yaml#/schemas/BadRequestProblemDetails`
      - **Example**:
        ```json
        {
          "parameter": {
            "media_url": ["The media_url field is required"]
          },
          "type": "https://www.rev.ai/api/v1/errors/invalid-parameters",
          "title": "Your request parameters didn't validate",
          "status": 400
        }
        ```

#### `/speechtotext/v1/jobs/{id}/transcript`

- **Parameters**:
  - `$ref: 'shared.yaml#/parameters/JobId'`

- **GET**
  - **Summary**: Get Transcript By Id
  - **Operation ID**: `GetTranscriptById`
  - **Description**: 
    Returns the transcript for a completed transcription job. The transcript can be returned as either JSON or plaintext format. Transcript output format can be specified in the `Accept` header. Returns JSON by default.
  - **Tags**: `API Reference - Transcript`
  - **Parameters**:
    - `$ref: '#/components/parameters/acceptTranscript'`
  - **Responses**:
    - **200**: Rev AI API Transcript
      - **Content**:
        - `application/vnd.rev.transcript.v1.0+json`
        - **Schema**: `$ref: 'shared.yaml#/schemas/Transcript'`
        - **Examples**:
          - Skip Diarization: `false` & Skip Punctuation: `false`:
            ```json
            {
              "monologues": [
                {
                  "speaker": 1,
                  "elements": [
                    { "type": "text", "value": "Hello", "ts": 0.5, "end_ts": 1.5, "confidence": 1 },
                    { "type": "punct", "value": " " },
                    { "type": "text", "value": "World", "ts": 1.75, "end_ts": 2.85, "confidence": 0.8 },
                    { "type": "punct", "value": "." }
                  ]
                }
              ]
            }
            ```
        - **406**: Invalid Transcript Format
        - **404**: Job Not Found

