# Knowledge Requirements

This guide is written for developers and is best understood by those with previous knowledge of the following technologies:

- Docker
- REST API
- Redis
- Postman (for asynchronous transcription)
- Websockets (for live transcription)

### Remarks
We recommend using the On Prem solution with an Intel architecture.

# Obtaining a Rev AI Access Token
Go to the [Rev AI](https://rev.ai) website and create an account. Once logged in, nagivate to the `Access Token` section on the side bar and generate a new token.
<span style="color: red;">Please store this token in a safe location, as you will need it to run the on prem containers</span>. You can also regenerate a new token and delete the old token via the website if you were to lose your token.

# Downloading Images

The images are under the public Docker Hub repository `revdotcom/reverb-self-hosted`.

```
$ docker pull revdotcom/reverb-self-hosted:<TAG_VERSION>
```

## Available Tags

The On Prem deployment is split into multiple images: `Gateway`, `Workers`, and `Streaming`.
The latest images are taged with `latest`, i.e.:
- `open-gateway-latest`
- `open-workers-latest` and `open-en-workers-latest`
- `open-streaming-latest` and `open-en-streaming-latest`

For asynchronous transcription, only `Gateway` and `Workers` images are needed. For live transcription, only `Gateway` and `Streaming` images are needed.

If you encounter any issues when downloading please reach out to our team by email at [support@rev.ai](mailto:support@rev.ai).

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

Following `Worker` submission queue environment variable is required. This ensures that the `Gateway` is able to submit transcription jobs.

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
| `MetricsConfig__Prometheus__Summary__Rtf__QuantileEpsilonPairsString` | string | `0.5,0.01;0.9,0.01;0.99,0.01` | Optional string representation of Prometheus Summary quantile and epsilon pairs to record Prometheus Summary quantiles on RTF with. It should be semicolon-delimited, with a comma separating the quantile and epsilon values, i.e. '0.5,0.01;0.9,0.01;0.99,0.01' represents the quantile/epsilon pairs [0.5, 0.01] [0.9, 0.01] [0.99, 0.01]. |
| `MetricsConfig__Prometheus__Summary__SourceFileDownloadTimeSeconds__QuantileEpsilonPairsString` | string | `0.5,0.01;0.9,0.01;0.99,0.01` | Optional string representation of Prometheus Summary quantile and epsilon pairs to record Prometheus Summary quantiles on source file download time in seconds with. It should be semicolon-delimited, with a comma separating the quantile and epsilon values, i.e. '0.5,0.01;0.9,0.01;0.99,0.01' represents the quantile/epsilon pairs [0.5, 0.01] [0.9, 0.01] [0.99, 0.01]. |
| `MetricsConfig__Prometheus__Summary__SourceFileSizeKb__QuantileEpsilonPairsString` | string | `0.5,0.01;0.9,0.01;0.99,0.01` | Optional string representation of Prometheus Summary quantile and epsilon pairs to record Prometheus Summary quantiles on source file size in kb with. It should be semicolon-delimited, with a comma separating the quantile and epsilon values, i.e. '0.5,0.01;0.9,0.01;0.99,0.01' represents the quantile/epsilon pairs [0.5, 0.01] [0.9, 0.01] [0.99, 0.01]. |

## Streaming Environment Variables

### Required Environment Variables

| Variable | Type | Description |
|---|---|---|
| `OnPrem__AccessToken` | string | Your Rev AI account access token. A valid token must be provided in order for the container to authenticate. |
| `OnPrem__Redis__Endpoints` | string | Endpoints for the Redis instance. |
| `OnPrem__Redis__KeyPrefix` | string | Prefix for all keys stored into Redis for this application to maintain key isolation across shared environments. |
| `OnPrem__Redis__Database` | number | Database for Redis. Should be the same as the `Redis__Database` value provided to the `Gateway`. |
| `OnPrem__InstanceIp` | string | IP address of the container running the streaming Docker image. `Gateway` containers must be able to make requests to this IP address. |

### Optional Environment Variables

| Variable | Type | Default | Description |
|---|---|---|---|
| `OnPrem__InitializationCallbackUrl` | string | null | Callback URL for `POST` request, made when the container initialization succeeds or fails. |
| `OnPrem__InitializationAttempts` | number | 5 | The number of attempts made to reach the initialization callback before failure. |
| `OnPrem__InitializationRetryDelayMs` | number | 1000 | The amount of time in milliseconds between each attempt to POST to the initialization callback. |
| `OnPrem__Redis__EnableTls` | boolean | false | Whether Redis requests are required to be sent via SSL. |
| `OnPrem__Redis__Password` | string | null | Password used to connect to the provided Redis instance. |
| `Workers__NumWorkers` | number | 1 | The number of workers available for the container to process audio streams. Each worker can process one stream at a time; the higher the number of workers, the more streams that can be processed in parallel. |
| `ASPNETCORE_URLS` | string | `http://+:80` | ASP .Net Core environment variable to control the port which the application runs under |

# Networking

The images require an open network connection for authentication and initialization.

If instead you would prefer to whitelist specific URLs, the containers require communication with the following:

- `https://api.rev.ai/external/v2open/initialize`
- `https://api.rev.ai/external/v2open/billing`

## Initialization

The `POST` request to `/initialize` is a required licensing request to Rev AI's servers. It verifies the On Prem containers' license against your Rev AI account. The request contains the following information:

`access_token` - Customer's access token.

`container_key` - A unique container key identifying the On Prem container image hardcoded into the container itself.

`container_version` - The version of the container that is sending the initialization request, this is equivalent to the tag of the container image.

Example request:

```
Authorization: Bearer <YOUR_ACCOUNT_TOKEN>

{
    "container_key":"UnIqUeKeY",
    "container_version":"open-gateway-0.1.0"
}
```

Log Message on Successful Initialization:
```
2021-09-14 06:56:11.568	 Initialization response user token is "v3|fV5_-QZjdRV-DWdRb-EUBtk7TqruQkQvVSmhg4jPU7XYkc79qxFzzevUzsEVfsPTNfQB6g"
```

**We maintain there will be no customer data ever in this request**

## Billing

All images make a `POST` request to `/billing`. The billing requests from the `Workers` are used for logging purposes by the Rev AI team for accounting. The billing request from the `Gateway` contains the actual duration of the overall job. The requests contain the following:

**Headers**

`x-rev-signature` - Signature which verifies the integrity of the request.

`x-rev-billing-type`- Billing request type, can be `api` or `worker`.

**Body**

`duration` - A numeric value for that either contains the total duration in seconds of the job or a duration for accounting purposes.

`reference_id` - A universally unique identifier.

`request_sent_on` - A date time stamp of when this request was made.

`revaiapi_endpoint` - An http url that the customer can use to forward the request to if they wish to implement a sidecar to inspect this billing request before it reaches Rev AI's server.

`user_token` - A string in the following format: `v4|someString`. See initialization log message above for example.

`language` - Language code of the job.

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
    "user_token": "v4|someuniquestring",
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

## Streaming Resources

### CPU and Memory Requirements

The number of resources required varies depending on the number of workers each container has been initialized with using the `Workers__NumWorkers` variable. The number of workers dictates the number of streams that can be transcribed in parallel. Each stream started with `Gateway` will create a connection between the gateway and a `Streaming` container with an available worker.

**The Rev AI recommended configuration is to run 4 workers on a AWS c6i.2xlarge equivalent machine (4 physical cores, 16gb memory). All resources must be provided to the container. Any other configuration should be tested by the customer that it is stable.**

As a base, using a single streaming worker, the container requires at least 5GB of RAM. The table below provides information on the *MINIMUM* memory requirements for up to 4 workers. In the event you wish to run a non-standard configuration, you can use this a baseline. However it is guaranteed the image will require more memory than the baseline amount to process the job. A buffer of at least 2x the minimum memory is recommended.

| Worker Count | Minimum Memory |
| ---          | ---            |
| 1 | 5 GB |
| 2 | 6 GB |
| 3 | 7 GB |
| 4 | 8 GB |
| x | (1x + 4) GB |

## Redis Requirements

Intermediate transcript files and job data are stored in the provided Redis instance for a period of 24 hours. The Redis instance must be provisioned with enough memory to handle the number of jobs over a 24 hour period. Users can manually clear the space in Redis by deleting jobs once they complete and have received their transcripts.

**~1350 hours of transcribed audio can be stored in a 16GB Redis instance over a 24 hour period.**

# Starting the Containers

There are multiple ways to start the containers: `docker run`, `docker-compose` or `kubernetes`. In this guide we will provide examples for `docker run` and `docker-compose`.

## Starting From the Docker Run Command

From the command line run the following:

```
# Gateway
$ docker run -p <YOUR_HOST_PORT:80> -e Initialization__AccessToken=<YOUR_REV_AI_ACCESS_TOKEN> -e Redis__Endpoints=<YOUR_REDIS_ENDPOINT> -e Redis__KeyPrefix=<YOUR_KEY_PREFIX> -e  Redis__Database=<YOUR_REDIS_DATABASE> -e Revspeech__SubmissionQueue=revspeech-submission -e Revspeech__CompletionQueue=revspeech-completion
revdotcom/reverb-self-hosted:open-gateway-<TAG_VERSION>

# Workers
$ docker run -e AccessToken=<YOUR_REV_AI_ACCESS_TOKEN> -e RedisConfig__Endpoints=<YOUR_REDIS_ENDPOINT> -e RedisConfig__KeyPrefix=<YOUR_KEY_PREFIX> -e  RedisConfig__Database=<YOUR_REDIS_DATABASE> -e InboundQueueName=revspeech-submission revdotcom/reverb-self-hosted:open-workers-<TAG_VERSION>

# Streaming
$ docker run -e OnPrem__AccessToken=<YOUR_REV_AI_ACCESS_TOKEN> -e OnPrem__Redis__Endpoints=<YOUR_REDIS_ENDPOINT> -e OnPrem__Redis__KeyPrefix=<YOUR_KEY_PREFIX> -e  OnPrem__Redis__Database=<YOUR_REDIS_DATABASE> -e OnPrem__InstanceIp=<THIS_CONTAINER_IP> revdotcom/reverb-self-hosted:open-streaming-<TAG_VERSION>
```

## Starting From Docker Compose

See the instructions in the [docker-compose](/onprem-api/local-environment/docker-compose) README.

## Container User
For the sake of security, the containers by default run under non-root users predefined by the image. For certain operating systems, non-root users by default are not able to bind port 80 for `localhost`, which the application requires by default.

If network or OS configurations prevent this, you can change the default application port by providing the following environment variable: `--emv ASPNETCORE_URLS=http://+:<custom_port>` on `docker run`. Optionally you can also run the docker as root user: `-u root` 

## Providing InstanceIP

In order for streams to function, `Gateway` container(s) must be able to form WebSocket connections with `Streaming` containers. Internally we do this by keeping track of IP addresses
at which the `Gateway` container(s) can request to form a connection with each `Streaming` container. These IP addresses are highly dependent on the container orchestration system
(or lack thereof) that you use. Therefore we ask that you provide each `Streaming` container with its own IP address accessible to the `Gateway` container(s) in the form of an environment variable
set on container startup. An example for Kubernetes is shown below;
```
#Kubernetes: Downward API example for getting Pod IP
spec:
  containers:
    - name: streaming-worker
      image: <IMAGE NAME>
      imagePullPolicy: Always
      ports:
        - containerPort: 80
      resources:
        requests:
          cpu: <CPU>
          memory: <MEMORY>
      env:
        - name: OnPrem__InstanceIp
          valueFrom:
            fieldRef:
              fieldPath: status.podIP
```
This takes advantage of the Kubernetes [Downward API](https://kubernetes.io/docs/concepts/workloads/pods/downward-api/) to set this environment variable for each started pod.

## File Storage

Due to the nature of having persistent files over a 24 hour period, an object based file storage system is required to be configured as transcripts and other intermediate outputs from the `Workers` are stored and referenced to. Examples of cloud-based file storages include S3 and Azure blob storage. The file storage needs to support the following:

- Uploading of files/objects referenced via a key/path structure
- Retrieving of files/objects referenced via a key/path structure
- Deleting of files/objects referenced via a key/path structure

The `Gateway` and `Workers` by default have a Redis-backed file storage system implementation built in. `Streaming` does not store any files.

**Blob Storage Prefix and Lifecycle**

Each image will store their files with a given Blob prefix. The `Gateway` will have its files in the `<Blob_Container_Name>` container with the `/gateway` as the path prefix to all Blobs. `Workers` will have `/workers` as it's prefix. By default, these files are not cleaned up automatically. **It is highly recommended to put a lifecycle rule on the Blob Container to auto-delete up these Blob prefixes.**

The `Gateway` and `Workers` by default have a Redis-backed database implementation built in. **Note this database's items expire after 24 hours.**

## Database

On Prem also introduces the concept of the `job` data model. To have these models persist for 24 hours, a NoSQL-styled document database is required. Examples of cloud solutions similar to the database provided include `DynamoDb` and `MongoDb` The required operations and functionalities for the document database are the basic CRUD operations:

- Create Object
- Update Object
- Get Object(s)
- Delete Objects
- Indexing Objects
- Primary Key

The `Gateway` and `Workers` have a Redis-backed database implementation built in. **Note this database's items expire after 24 hours.**

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

The `Gateway`, `Streaming` and `Workers` containers have a 24 hour expiration limit. 24 hours after the container has started, a `sigterm` signal is sent to the application. During this `sigterm` period, the application prevents new workflows/jobs to be processed and waits for all processing workflows/jobs to be completed. Once completed, the container is shutdown.

# Using On Prem

## When Are the Containers Ready?

Upon startup the containers will attempt to authenticate with Rev AI by sending a `POST` request to the `https://api.rev.ai/external/v2open/initialize` endpoint. After the container receives a response, if the `Initialization__CallbackUrl` or `InitializationCallbackUrl` are provided, the containers will then send a POST request to that url. This `POST` contains the initialization status.

Successful `POST` body:
```
{ "success": true }
```

Failed `POST` body:
```
{ "success": false }
```

## Submitting a Job for Asynchronous Transcription

Once both the `Gateway` and `Workers` images are running, you can submit a file by making a `POST` request to `/speechtotext/v1/jobs` endpoint of the `Gateway` container with the proper parameters in the request body. Once the job is submitted, the user can query the status of the job and transcript via the API exposed in the `Gateway`. As On Prem is meant to keep near feature parity with the Cloud API, the precise API and model documentation can be found in the [Cloud API Reference](https://rev.ai/docs).

## Gateway Workflow

When the job is created, it is stored into the Redis instance and is available to be queried for 24 hours before expiring (if not deleted). During this process, the job status can be:
`in_progress`, `transcribed` or `failed`. The job will be placed into a workflow where it will be processed for transcription. During this workflow phase, any unexpected failures will result in a retry, until the 2 hour transcription expiration is reached. At this point, if the job has not yet completed, the job will be failed with `internal_processing`.

## Audio Duration Detection

Internally, the `Gateway` uses [Ffprobe](https://ffmpeg.org/ffprobe.html) and the audio's metadata to determine the length of the audio. **On Prem has a audio duration maximum of 15 hours, and a minimum of 2 seconds.**. A corrupted or malformed audio where the audio duration cannot be detected may result in the job being failed prematurely.

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

## Starting a Transcription Stream

Once both the `Gateway` and `Streaming` images are running, you can begin a stream by making a WebSocket request to `/speechtotext/v1/stream` endpoint of the `Gateway` container with the proper parameters in the request body. Because this endpoint is a WebSocket endpoint it is not documented in the list of HTTP endpoints below, please refer to our public documentation for inforamation about the streaming protocol. As On Prem is meant to keep near feature parity with the Cloud API, the precise API and model documentation can be found in the [Cloud API Reference](https://docs.rev.ai/api/streaming/).

## Initial Connection

Once a WebSocket connection is opened with the `Gateway` it will attempt to find an available worker on a `Streaming` instance. Assuming this is successful the client will receive a text WebSocket messsage which looks like
```
# Successful connection message
{
    "type": "connected"
}
```

Receiving this message means the `Gateway` is ready to begin receiving binary data from the client.

## During the Connection

Once connected the protocol is the same as our [Cloud Streaming Speech-To-Text API](https://docs.rev.ai/api/streaming/requests/#WebSocket-protocol). The `Gateway` will act as a proxy between the client and the `Streaming` worker to receive binary data and deliver streaming results.
The format of these results is detailed in the [Cloud Streaming Speech-To-Text API documentation on responses](docs.rev.ai/api/streaming/responses/).

## Finishing the Connection

A successful stream is ended via the following steps:
1. Client signals they are finished sending binary data by sending a text WebSocket message containing the following content `EOS`.
2. `Streaming` and `Gateway` instance finish processing any audio still pending
3. `Gateway` sends WebSocket close message to client with code `1000` and reason `Normal Closure.`
4. client responds with successful close message of their own

A failed stream will result in the client receiving a close message with a code and reason other than `1000`/`Normal Closure.`. Possible failures are listed below.

### Close Codes

| Close Code | Close Reason | Explaination |
| ---        | ---          | ---          |
| 1006 | Abnormal Closure | Catch all for most abnormal failures, you will need to look at the logs for further information |
| 4002 | Bad Request... | In the case of a 4002 the Close Reason will contain more information about what specific parameters in your request were invalid |
| 4010 | Shutting Down | Received when the `Gateway` or `Streaming` instance receives a SIGTERM in the middle of a stream |
| 4013 | Try Again Later | There are no `worker`s current available to service the connection. You will need to wait for an ongoing stream to end or scale in more instances of the `Streaming` container |

## Streaming Billing

Upon finishing the connection the `Gateway` will make a request to the same billing endpoint as for asynchronous jobs with the duration of your stream. This duration is
measured as the time between your connection opening and closing. You will not be billed for streams which receive a close code of 4013.

## Streaming Workers

Each `Streaming` application maintains set stateless speech-to-text `worker` threads. The number of `worker` threads can be configured by setting `Workers__NumWorkers`.

The `Streaming` application uses [Ffmpeg](https://ffmpeg.org/ffmpeg.html) to transcode received binary messages into PCM16 audio. The speech-to-text `worker` performs the actual speech-to-text processing. The `worker` ingests a stream of PCM16 audio  and outputs transcribed text back over the same WebSocket connection. If for some reason there is a failure the `worker` communicates this failure back to the `Streaming` application which then communicates it to the client via a WebSocket close code.

```
# Sample log output for a successful job, aggregated by RevRequestId (logs will also include ffmpeg process output but those have been cut from this example for readability)

2022-09-26 18:11:17.508	Refreshing instance with 0 workers
2022-09-26 18:11:17.508	Claiming worker id: "Post-dd095787-8fda-4851-a2cf-65676d57f155" "0"
2022-09-26 18:11:17.508	worker claimed
2022-09-26 18:11:17.509	Acquired Post Processing worker with WorkerId: "Post-dd095787-8fda-4851-a2cf-65676d57f155_0"
2022-09-26 18:11:17.509	worker claimed
2022-09-26 18:11:17.509	Acquired Ctm worker with WorkerId: "Ctm-eadef66a-b50f-4833-97e4-3c54eef8558a_2"
2022-09-26 18:11:17.509	Sending session start message to worker
2022-09-26 18:11:17.509	Signaling connected to midtier
2022-09-26 18:11:17.509	Skipping ffmpeg write-through step
2022-09-26 18:11:17.509	Job request - "{\"id\":\"26\",\"job_dir_name\":\"/tmp/workers/Ctm-eadef66a-b50f-4833-97e4-3c54eef8558a/cfd09c27-939b-4526-b826-758a11d3eacc\",\"full_partials\":false,\"custom_vocab\":\"\",\"sample-rate\":16000,\"enable-speech-activity-detector\":false,\"enable-speaker-switch\":false}"
2022-09-26 18:11:17.509	Claiming worker id: "Ctm-eadef66a-b50f-4833-97e4-3c54eef8558a" "2"
2022-09-26 18:11:17.509	Updating scale in protection
2022-09-26 18:11:17.509	The job config value "enable-speaker-switch" was changed
2022-09-26 18:11:17.509	The job config value "enable-speech-activity-detector" was changed
2022-09-26 18:11:17.510	The job config value "job_dir_name" is ignored
2022-09-26 18:11:17.510	The job config value "sample-rate" was changed
2022-09-26 18:11:17.510	Creating job id:"26" took 0.167366 ms
2022-09-26 18:11:17.510	Starting job processing, job "26" ("0x7f5fa92f98b0")
2022-09-26 18:11:17.510	Sample rate "16000"
2022-09-26 18:11:17.512	Refreshing instance with 0 workers
2022-09-26 18:11:24.355	Final: segment 4500 - 7390, sw count "1"
2022-09-26 18:11:24.375	Postprocessing for 7 words "completed" in 0.0 sec
2022-09-26 18:11:26.256	Final: segment 7260 - 9630, sw count "1"
2022-09-26 18:11:26.270	Postprocessing for 4 words "completed" in 0.0 sec
2022-09-26 18:11:35.687	Final: segment 9630 - 19870, sw count "1"
2022-09-26 18:11:40.885	"Timeout reading WebSocket."
2022-09-26 18:11:40.947	Final: segment 19870 - 19980, sw count "1"
2022-09-26 18:11:45.948	"Timeout reading WebSocket."
2022-09-26 18:11:47.351	Received EOS from midtier
2022-09-26 18:11:47.351	Finished in flight cv step
2022-09-26 18:11:47.351	Signalling EOS to worker
2022-09-26 18:11:47.351	Finished sending EOS to worker
2022-09-26 18:11:47.351	Finished midtier pipeline
2022-09-26 18:11:47.352	Job stats: id - "26" completed. Processed 20 seconds of audio, cv init time 0, decoder time 0, silence time 0, chunk time 22.718523451. Total finals "2". Number speaker switches "0".
2022-09-26 18:11:47.352	SetCompleted didn't change status of the job "26". Current job status is: "3"
2022-09-26 18:11:47.352	Received Eos from streaming worker
2022-09-26 18:11:47.352	Finished worker pipeline
2022-09-26 18:11:47.352	Finishing streaming pipeline
2022-09-26 18:11:47.352	Gracefully finishing client connection
2022-09-26 18:11:47.352	MD5 of the job "26" audio is "CE1D485E8B538EE1F28EC91167EF0CE0"
2022-09-26 18:11:47.352	Freeing worker connections: "Post-dd095787-8fda-4851-a2cf-65676d57f155_0", "Ctm-eadef66a-b50f-4833-97e4-3c54eef8558a_2"
2022-09-26 18:11:47.352	Refreshing instance with 0 workers
2022-09-26 18:11:47.353	Updating scale in protection
2022-09-26 18:11:47.353	Updating scale in protection
2022-09-26 18:11:47.353	Updating scale in protection
2022-09-26 18:11:47.353	Worker unclaimed
2022-09-26 18:11:47.353	Worker unclaimed
2022-09-26 18:11:47.353	Updating scale in protection
2022-09-26 18:11:47.353	Invoked "GET" "https://10.0.49.169/v1/stream?content_type=audio/x-raw;layout=interleaved;rate=16000;format=S16LE;channels=1&remove_disfluencies=True&full_partials=False&enable_speaker_switch=False&skip_postprocessing=False", returned 101
2022-09-26 18:11:47.358	Refreshing instance with 1 workers
2022-09-26 18:11:47.363	Refreshing instance with 1 workers
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

### Streaming

#### Worker Count Metrics

- Number of CTM workers initialized (`worker_count_ready_ctm`)
- Number of Post Processing workers initialized (`worker_count_ready_postprocessing`)
- Total number of worker sets initialized. Every job requires 1 worker set to process (`worker_count_ready_set`)

#### Worker Failure Metrics

- Number of bad request failures (`workers_job_failure_bad_request`)
- Number of transcribing failures due to lack of speech in the audio (`workers_job_failure_no_speech`)
- Number of service not allowed failures (`workers_job_failure_service_not_allowed`)
- Number of internal processing failures (`workers_job_failure_internal_processing`)

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

By default, the `Gateway`, `Workers`, and `Streaming` containers log to standard output (stdout) in [Serilog](https://serilog.net/) format. It is highly recommended to capture the standard output and save them for troubleshooting.

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

The `Streaming` container is a stateless application. All failures will result in a non 1000 close code as detailed above. Some errors such as `4002: Bad Request` are expected. To investigate any other failures we
recommend you find the RevRequestId for your streaming request and look at any associated logs from the `Streaming` and `Gateway` containers. Finding this RevRequestId is easiest done finding the associated API invocation log for your request.
These logs will look like:

```
2022-10-03 13:45:14.520	Invoked "GET" "https://api.rev.ai/speechtotext/v1/stream?access_token=abcd*&content_type=audio/x-raw;layout=interleaved;rate=48000;format=S16LE;channels=1&metadata=test", returned 101
```

The RevRequestId from this log can then be used to find all other logs associated with the stream from both the `Gateway` and `Streaming` containers.

# Cloud Differences

## API

On Prem tries to keep its API interface as similar to Rev AI's Cloud API as possible, however there are a few differences:

- No Access Token required per request. Instead Access Token is provided on container startup.
- `media_url` must be provided for submission. `multiform/form-data` submission does not exist.
- Jobs persist for a maximum of 24 hours instead of 30 days
- Maximum audio length is 15 hours instead of 17
- Allowed languages are restricted to English only
- `delete_after_seconds` is not supported in On Prem
- New job `failure` type: `billing_failure`
- There is no `GET /account` endpoint for On Prem

## Workflow

The workflow of a On Prem has a few key differences compared to Rev AI's Cloud solution:

- On Prem performs preliminary duration detection using ffprobe on the provided `media_url`. If the duration metadata cannot
be accurately detected from the `media_url`, the job will not be chunked correctly which may increase turn around time
- Maximum time the job can spend in the workflow is 2 hours and 15 minutes before it fails with `internal_processing`

## Streaming API

On Prem tries to keep its API interface as similar to Rev AI's Cloud API as possible, however there are a few differences:

- No Access Token required per request. Instead Access Token is provided on container startup
- Jobs are not created for Streaming requests and no information about the stream is persisted to any storage service
- Only allowed language is English
- Custom Vocabularies are not supported

# On Prem API Endpoints

### Paths

#### `/healthcheck`

- **GET**
  - **Summary**: Healthcheck
  - **Description**: 
    Returns a 200 when the `Gateway` application is healthy and ready to receive requests. This endpoint is also available in the `Workers` application.
  - **Responses**:
    - **200**: Healthcheck Success

#### `/speechtotext/v1/jobs/{id}`

- **Parameters**:
    - **Name**: `id`
    - **In**: `path`
    - **Description**: Rev AI API Job Id
    - **Required**: `true`
    - **Schema**:  
        - **Type**: `string`

- **GET**
    - **Summary**: Get Job By Id
    - **Description**: Returns information about a transcription job.
    - **Responses**:
        - **200**: Transcription Job Details
            - **Content**: `application/json`
            - **Schema**:
                | Field | Type | Description | Example |
                |-------|------|-------------|---------|
                | id | string | Id of the job | "Umx5c6F7pH7r" |
                | status | string | Current status of the job. Enum: "in_progress", "transcribed", "failed" | "transcribed" |
                | created_on | string | ISO 8601 timestamp of when the job was created | "2018-05-05T23:23:22.29Z" |
                | completed_on | string | ISO 8601 timestamp of when the job was completed | "2018-05-05T23:45:13.41Z" |
                | metadata | string | Optional metadata that was provided during job submission | "{"order_id": "123456"}" |
                | name | string | Name of the file provided. Present when the file name is available | "sample_audio.mp3" |
                | duration_seconds | number | Duration of the file in seconds. Null if the file could not be retrieved or there was not a valid media file | 324.36 |
                | failure | string | Simple reason of why the transcription job failed. Enum: "internal_processing", "duration_exceeded", "duration_too_short", "invalid_media", "empty_media", "transcription", "billing" | "download_failure" |
                | failure_detail | string | Detailed explanation of the failure | "Failed to download media file. Please check your url and file type" |
                | type | string | Type of speech recognition performed. Always "async" | "async" |
                | callback_url | string | URL to invoke when processing is complete | "https://example.com/callback" |
                | media_url | string | Media url provided by the job submission | "https://www.rev.ai/FTC_Sample_1.mp3" |
                | skip_diarization | boolean | User-supplied preference on whether to skip diarization | true |
                | skip_punctuation | boolean | User-supplied preference on whether to skip punctuation | true |
                | remove_disfluencies | boolean | User-supplied preference on whether to remove disfluencies | true |
                | filter_profanity | boolean | User-supplied preference on whether to remove explicit words | true |
                | speaker_channels_count | integer | User-supplied number of speaker channels in the audio. Min: 1, Max: 8 | 2 |
                | diarization_type | string | Diarization model to use for the speech-to-text job. Enum: "standard", "premium" | "premium" |

      - **Examples**:
        - **New Job**: 
            ```json
            {
                "id": "Umx5c6F7pH7r",
                "status": "in_progress",
                "language": "en",
                "created_on": "2018-05-05T23:23:22.29Z"
            }
            ```
        - **Transcribed Job**: 
            ```json
            {
                "id": "Umx5c6F7pH7r",
                "status": "transcribed",
                "language": "en",
                "created_on": "2018-05-05T23:23:22.29Z",
                "completed_on": "2018-05-05T23:45:13.41Z",
                "callback_url": "https://www.example.com/callback",
                "duration_seconds": 356.24,
                "media_url": "https://www.rev.ai/FTC_Sample_1.mp3"
            }
            ```
        - **Failed Job**: 
            ```json
            {
                "id": "Umx5c6F7pH7r",
                "status": "failed",
                "language": "en",
                "created_on": "2018-05-05T23:23:22.29Z",
                "completed_on": "2018-05-05T23:23:24.11Z",
                "failure": "download_failure",
                "failure_detail": "Failed to download media file. Please check your url and file type"
            }
            ```
    - **404**: 
        ```json
        {
            "type": "https://www.rev.ai/api/v1/errors/job-not-found",
            "title": "could not find job",
            "status": 404
        }
        ```

- **DELETE**
  - **Summary**: Delete Job by Id
  - **Description**: 
    Deletes a transcription job. All data related to the job, such as input media and transcript, will be permanently deleted. A job can only be deleted once it's completed (either with success or failure).
  - **Responses**:
    - **204**: Job was successfully deleted.
    - **404**:
        - **Content**: `application/problem+json`
        - **Examples**:
            ```json
            {
                "type": "https://rev.ai/api/v1/errors/job-not-found",
                "title": "could not find job",
                "status": 404
            }
            ```
    - **409**: Bad Request
      - **Content**: `application/problem+json`
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
  - **Description**: 
    Gets a list of transcription jobs submitted within the last 24 hours in reverse chronological order up to the provided `limit` number of jobs per call. **Note:** Jobs older than 24 hours will not be listed. Pagination is supported via passing the last job `id` from a previous call into `starting_after`.
  - **Parameters**:
    | Parameter | Location | Description | Schema |
    |-----------|----------|-------------|--------|
    | limit | query | The maximum number of jobs to return. | integer |
    | starting_after | query | A cursor for use in pagination. starting_after is an object ID that defines your place in the list. For instance, if you make a list request and receive 100 objects, ending with obj_foo, your subsequent call can include starting_after=obj_foo in order to fetch the next page of the list. | string |
    - **Responses**:
        - **200**: List of Rev AI Transcription Jobs
            - **Content**: `application/json`
            - **Schema**:
                | Field | Type | Description |
                |-------|------|-------------|
                | (root) | array | An array of Async Transcription Job objects |

    - **400**: Bad Request
      - **Content**: `application/json`
      - **Schema**: 
        | Field | Type | Description |
        |-------|------|-------------|
        | type | string | A URI reference that identifies the problem type. |
        | title | string | A short, human-readable summary of the problem type. |
        | status | integer | The HTTP status code (typically 400 for bad request). |
        | detail | string | A human-readable explanation specific to this occurrence of the problem. |
        | instance | string | A URI reference that identifies the specific occurrence of the problem. |
        | errors | object | Additional details about the errors that caused the bad request. |

- **POST**
  - **Summary**: Submit Transcription Job
  - **Description**: 
    Starts an asynchronous job to transcribe speech-to-text for a media file. Media files can be specified by including a public URL to the media in the transcription job `options`.
  - **Request Body**:
    - **Description**: Transcription Job Options
    - **Required**: `true`
    - **Content**: `application/json`
    - **Schema**:
        | Field | Type | Description | Required | Example |
        |-------|------|-------------|----------|---------|
        | media_url | string | Direct download media url. Ignored if submitting job from file. Note: Media files longer than 17 hours are not supported for English transcription, and media files longer than 12 hours are not supported for non-English transcription. For non-English jobs, expected turnaround time can be up to 6 hours. | Yes | "https://www.rev.ai/FTC_Sample_1.mp3" |
        | metadata | string | Optional metadata provided during submission. This field is attached to the external billing request. Do not put sensitive information in this field | No | "{"order_id": "123456"}" |
        | callback_url | string | Optional callback url to invoke when processing is complete. If provided, it will be visible in the response. | No | "https://example.com/callback" |
        | skip_diarization | boolean | Specify if speaker diarization will be skipped by the speech engine | No | true |
        | skip_punctuation | boolean | Specify if "punct" type elements will be skipped by the speech engine. For JSON outputs, this includes removing spaces. For text outputs, words will still be delimited by a space | No | true |
        | remove_disfluencies | boolean | When set to true, disfluencies (currently only 'ums' and 'uhs') will not appear in the transcript. | No | true |
        | filter_profanity | boolean | When enabled, approximately 600 profanities will be filtered. Matching words will have all characters replaced by asterisks except for the first and last. | No | true |
        | speaker_channels_count | integer | Specifies the total number of unique speaker channels in the audio. Each channel will be transcribed separately. | No | 2 |
        | custom_vocabularies | array | Specify a collection of custom vocabulary to be used for this job. Currently only available for English transcription jobs. | No | See example below |
        | language | string | ISO 639-1 language code (or ISO 639-3 for Mandarin) specifying the language to transcribe. | No | "en" |
        | estimated_duration_seconds | number | When provided, skips the FFProbe step and uses this value to chunk the audio instead. Must be positive when provided. | No | 300 |
        | diarization_type | string | Diarization model to use for the speech-to-text job. Options: "standard" or "premium" | No | "premium" |
  - **Responses**:
    - **200**: Transcription Job Details
      - **Content**: `application/json`
      - **Schema**:
        | Field | Type | Description |
        |-------|------|-------------|
        | id | string | Id of the job | "Umx5c6F7pH7r" |
        | status | string | Current status of the job. Enum: "in_progress", "transcribed", "failed" | "transcribed" |
        | created_on | string | ISO 8601 timestamp of when the job was created | "2018-05-05T23:23:22.29Z" |
        | metadata | string | Optional metadata that was provided during job submission | "{"order_id": "123456"}" |
        | name | string | Name of the file provided. Present when the file name is available | "sample_audio.mp3" |
      - **Example**: 
        ```json
        {
          "id": "Umx5c6F7pH7r",
          "status": "in_progress",
          "created_on": "2018-05-05T23:23:22.29Z"
        }
        ```
    - **400**: Bad Request
      - **Content**: `application/problem+json`
      - **Schema**: 
        | Field | Type | Description |
        |-------|------|-------------|
        | type | string | A URI reference that identifies the problem type. |
        | title | string | A short, human-readable summary of the problem type. |
        | status | integer | The HTTP status code (typically 400 for bad request). |
        | detail | string | A human-readable explanation specific to this occurrence of the problem. |
        | instance | string | A URI reference that identifies the specific occurrence of the problem. |
        | errors | object | Additional details about the errors that caused the bad request. |
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
  - **Name**: `id`
  - **In**: `path`
  - **Description**: Rev AI API Job Id
  - **Required**: `true`
  - **Schema**:  
      - **Type**: `string`

- **GET**
  - **Summary**: Get Transcript By Id
  - **Description**: 
    Returns the transcript for a completed transcription job. The transcript can be returned as either JSON or plaintext format. Transcript output format can be specified in the `Accept` header. Returns JSON by default.
  - **Parameters**:
    | Field | Description |
    |-------|-------------|
    | Name | Accept |
    | In | header |
    | Description | MIME type specifying the transcription output format. Allowed values: `text/plain`, `application/vnd.rev.transcript.v1.0+json` |
    | Required | false |
    | Schema | string |
  - **Responses**:
    - **200**: Rev AI API Transcript
      - **Content**:
        - `application/vnd.rev.transcript.v1.0+json`
        - **Schema**:

            Transcript

            | Field | Type | Description |
            |-------|------|-------------|
            | monologues | array | An array of monologue objects, each representing a continuous speech by a single speaker |

            Monologue

            | Field | Type | Description |
            |-------|------|-------------|
            | speaker | integer | A numeric identifier for the speaker |
            | elements | array | An array of element objects, representing the content of the speech |

            Element

            | Field | Type | Description |
            |-------|------|-------------|
            | type | string | The type of the element. Can be "text", "punct", or "unknown" |
            | value | string | The content of the element |
            | ts | number | The timestamp in seconds when this element starts |
            | end_ts | number | The timestamp in seconds when this element ends |
            | confidence | number | A confidence score between 0 and 1 for this element (only for "text" type) |
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

#### `/speechtotext/v1/jobs/{id}/captions`

- **Parameters**:
  - **Name**: `id`
  - **In**: `path`
  - **Description**: Rev AI API Job Id
  - **Required**: `true`
  - **Schema**:  
      - **Type**: `string`

- **GET**
  - **Summary**: Get Captions
  - **Description**: 
    Returns the caption output for a transcription job. We currently support SubRip (SRT) and Web Video Text Tracks (VTT) output.
    Caption output format can be specified in the `Accept` header. Returns SRT by default.
    ***
    Note: For streaming jobs, transient failure of our storage during a live session may prevent the final hypothesis elements from saving properly, resulting in an incomplete caption file. This is rare, but not impossible.
  - **Parameters**:
    | Parameter | Location | Description | Schema |
    |-----------|----------|-------------|--------|
    | id | path | The ID of the job | string |
    | Accept | header | MIME type specifying the caption output format | string |
    | speaker_channel | query | Identifies which channel of the job output to caption. Default is `null` which works only for jobs with no `speaker_channels_count` provided during job submission. | integer |
  - **Responses**:
    - **200**: Rev AI API Captions
      - **Content**: `application/x-subrip`, `text/vtt`
      - **Schema**:
        | Field | Type | Description |
        |-------|------|-------------|
        | (root) | string | Caption content in the specified format |
      - **Note**: Caption output format is required in the Accept header. The supported headers are `application/x-subrip` and `text/vtt`.
    - **404**: Job Not Found
    - **405**: Invalid Job Property Captions
    - **406**: Invalid Caption Format
    - **409**: Bad Request
      - **Content**: `application/problem+json`
      - **Schema**: 
        | Field | Type | Description |
        |-------|------|-------------|
        | allowed_values | array | Allowed values for the job state |
        | current_value | string | Current value of the job state |
        | type | string | A URI reference that identifies the problem type |
        | title | string | A short, human-readable summary of the problem type |
        | detail | string | A human-readable explanation specific to this occurrence of the problem |
        | status | integer | The HTTP status code |