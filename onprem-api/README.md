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
| `Billing__Endpoint` | string | `https://api.rev.ai/external/v2/billing` | The provided endpoint is responsible for forwarding the billing request to `https://api.rev.ai/external/v2/billing`. |
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
| `OnPremBillingEndPoint` | string | `https://api.rev.ai/external/v2/billing` | The provided endpoint is responsible for forwarding the billing request to `https://api.rev.ai/external/v2/billing`. |
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
    "revaiapi_endpoint": "https://api.rev.ai/external/v2/billing",
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

## Container User
For the sake of security, the containers by default run under non-root users predefined by the image. For certain operating systems, non-root users by default are not able to bind port 80 for `localhost`, which the application requires by default.

If network or OS configurations prevent this, you can change the default application port by providing the following environment variable: `--emv ASPNETCORE_URLS=http://+:<custom_port>` on `docker run`. Optionally you can also run the docker as root user: `-u root` 

## Providing InstanceIP

In order for streams to function `Gateway` container(s) must be able to form WebSocket connections with `Streaming` containers. Internally we do this by keeping track of IP addresses
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
