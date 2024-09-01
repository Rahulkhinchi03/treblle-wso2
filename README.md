# Treblle Data Publisher

## Overview of Extension

The extension retrieves data from Global Synapse Handler (https://ei.docs.wso2.com/en/latest/micro-integrator/develop/customizations/creating-synapse-handlers/) in the API Gateway  and creates a payload to send to Treblle. The data is added onto a queue once received and is processed by a worker thread. The worker thread sends the data asynchronously to Treblle, if the data is successfully sent, the data is removed from the queue. If the data is not successfully sent, the worker thread will attempt to send the data again, after 1 retry attempt, the event will be dropped.

## Build the source code

Execute the following command from the root directory of the project to build

```sh
mvn clean install
```

## Usage

### Configuration

- Copy the built JAR artifact and place it inside the `<gateway>/repository/components/lib` directory and start the server to load the required classes.
- Add the following configuration at the **beginning** of the `<gateway>/repository/conf/deployment.toml` to engage the API Log Handler

  ```toml
  enabled_global_handlers = ["api_log_handler"]

  [synapse_handlers]
  api_log_handler.name = "TreblleHandler"
  api_log_handler.class = "com.treblle.wso2publisher.handlers.APILogHandler"
  ```


- Add the following in `<gateway>/repository/conf/log4j2.properties` for logging purposes
  
  > Following to enable the logs to populate under default `wso2carbon.log`. You can create a custom appender to log the entries to a separate log file.

  ```properties
  loggers = api-log-handler, AUDIT_LOG, ...

  logger.api-log-handler.name = com.treblle.wso2publisher
  logger.api-log-handler.level = INFO
  logger.api-log-handler.appenderRef.CARBON_LOGFILE.ref = CARBON_LOGFILE
  ```

- Before starting the WSO2 Server add the following environment variables.

  MacOs
  ```
  export TREBLLE_API_KEY=API-KEY
  export TREBLLE_PROJECT_ID=Project-id
  export TREBLLE_GATEWAY_URL="https://test.com" 
  export TREBLLE_QUEUE_SIZE=20000
  export TREBLLE_WORKER_THREADS=1
  export ADDITIONAL_MASK_KEYWORDS=testkey,Authorization,token
  ```
  Windows
  ```
  set TREBLLE_API_KEY=API-KEY
  set TREBLLE_PROJECT_ID=Project-id
  set TREBLLE_GATEWAY_URL="https://test.com" 
  set ADDITIONAL_MASK_KEYWORDS=testkey,Authorization,token
  set TREBLLE_QUEUE_SIZE=20000
  set TREBLLE_WORKER_THREADS=1
  ```

  - Definitions

  ```
  TREBLLE_API_KEY=<API Key of the Treblle Project>
  TREBLLE_PROJECT_ID=<Project Id of the Treblle Project>
  TREBLLE_GATEWAY_URL=<WSO2 API Manager Gateway URL> 
  TREBLLE_QUEUE_SIZE=<Messages queue size>
  TREBLLE_WORKER_THREADS=<Number of worker threads for publishing data>
  ADDITIONAL_MASK_KEYWORDS=<Masking keywords such as header names and body parameters>
  ```

