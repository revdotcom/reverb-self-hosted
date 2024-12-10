# Docker Compose for Self-Hosted Asynchronous Transcription

This Docker Compose configuration provides a local testing environment for the self-hosted asynchronous transcription solution. It sets up and orchestrates all necessary services, including the transcription service, Redis for managing asynchronous tasks and queuing, and Prometheus for monitoring system performance. With this configuration, users can quickly deploy and test the full solution on their local machine, ensuring smooth operation and integration before moving to production. The setup is designed to be lightweight and easy to customize, making it ideal for testing and development purposes.

### How to Run

Replace `[REVAI_API_KEY]` with your actual Rev AI access token in the `docker-compose.yaml` file.
To start the services, simply run the following command in the directory containing the `docker-compose.yml` file:

```bash
docker-compose up
```
 
### Testing the Solution

To understand how to interact with the self-hosted transcription service, use the provided 
markdown [Postman example collection](/onprem-api/local-environment/postman). This collection includes pre-configured requests for submitting audio files, checking transcription status, and retrieving the results. It is a great way to get familiar with the service and test its capabilities in your local environment.

### Accessing the Services

- **Gateway**: http://localhost:57350
- **English Worker**: http://localhost:57360
- **Spanglish Worker**: http://localhost:57365
- **Streaming Worker**: http://localhost:57370
- **Redis**: http://localhost:6379
- **Prometheus**: http://localhost:9090
