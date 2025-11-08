---
title: Building a Thread-Safe State Tracker Mediator for WSO2: Track State Of Integrations In WSO2 MI
description: Building a Thread-Safe State Tracker Mediator for WSO2: Track State Of Itegrations In WSO2MI
date: '2025-11-08'
categories: [WSO2, WSO2 MI]
keywords: [WSO2, WSO2MI, Java]
tags: [WSO2, WSO2MI, Java]
image:
  path: /assets/img/posts/wso2.png
  width: 800
  height: 400
  alt: ''
---
# Building a Thread-Safe State Tracker Mediator for WSO2: Track State Of Integrations In WSO2 MI

One of the challenges when building integration workflows with WSO2 is handling long-running processes that need to prevent parallel executions. Imagine a scenario where you're processing an order that involves multiple steps: inventory check, payment processing, shipment coordination, and delivery confirmation. You don't want another request to start processing the same order while the first one is still in progress. There are complex designs you can do to solve this problem, but WSO2 doesn not provide a solution OOB for this.

This is where the **WSO2 State Tracker Mediator** comes in handy. It's a thread-safe mediator designed to track process status, manage concurrent access, and monitor progress with built-in expiry support.

## The Problem

In enterprise integration scenarios, you often encounter these challenges:

1. **Duplicate Processing**: Multiple requests trigger the same process simultaneously
2. **State Uncertainty**: No clear way to know if a process is currently running
3. **Resource Wastage**: Repeated execution of expensive operations

Without proper state management, your integration workflows can suffer from race conditions, redundant processing, and operational chaos.

## The Solution: State Tracker Mediator

The WSO2 State Tracker Mediator provides a simple yet powerful way to:

- **Track process execution state** with unique identifiers
- **Block parallel executions** of the same process
- **Handle process expiry** with configurable TTL (Time To Live)
- **Maintain thread safety** for concurrent requests

### Architecture Overview

The mediator operates on three core operations:

1. **START_PROCESS** - Initiates tracking for a process
2. **IS_PROCESS_RUNNING** - Checks if a process is currently active
3. **STOP_PROCESS** - Marks a process as completed

Currently, the mediator uses an in-memory storage mechanism, hence it can only be used in Single instance deployment. However, it's designed with extensibility in mind, you can easily swap it out for database or registry-backed storage for distributed scenarios.

## How It Works

### Basic Usage

#### 1. Starting a Process

```xml
<property name="STATE_TRACKER_OPERATION" value="START_PROCESS"/>
<property name="PROCESS_IDENTIFIER" value="order-12345"/>
<property name="PROCESS_STATE_EXPIRY_TIME" value="3600"/>  <!-- optional, in seconds -->
<class name="com.ycr.wso2.mediator.statetracker.StateTrackerMediator"/>
```

When you start a process, the mediator:
- Records the process with a unique identifier (e.g., "order-12345")
- Captures the current timestamp
- Sets an optional expiry time for automatic cleanup

#### 2. Checking Process Status

```xml
<property name="STATE_TRACKER_OPERATION" value="IS_PROCESS_RUNNING"/>
<property name="PROCESS_IDENTIFIER" value="order-12345"/>
<class name="com.ycr.wso2.mediator.statetracker.StateTrackerMediator"/>
<!-- Result available in: PROCESS_IS_RUNNING (true/false) -->
```

Before starting a process, check if it's already running. The mediator sets the `PROCESS_IS_RUNNING` property to true or false, allowing you to:
- Route based on execution state
- Reject duplicate requests
- Log or audit concurrent attempts

#### 3. Stopping a Process

```xml
<property name="STATE_TRACKER_OPERATION" value="STOP_PROCESS"/>
<property name="PROCESS_IDENTIFIER" value="order-12345"/>
<class name="com.ycr.wso2.mediator.statetracker.StateTrackerMediator"/>
```

Once processing completes, stop tracking the process to allow future executions or cleanup resources.

## Output Properties

The mediator exposes the following properties for use in your workflow:

| Property | Description |
|----------|-------------|
| `STATE_TRACKER_RESULT` | Operation result and status |
| `PROCESS_IS_RUNNING` | Boolean indicating if process is currently running |
| `PROCESS_START_TIMESTAMP` | Timestamp when the process started (milliseconds) |

## Real-World Example

### Example 1: Processing customer orders with duplicate prevention.

```xml
<api name="OrderProcessingAPI" context="/orders">
  <resource methods="POST" uri-template="/process/{orderId}">
    <inSequence>
      <!-- Check if order is already being processed -->
      <property name="STATE_TRACKER_OPERATION" value="IS_PROCESS_RUNNING"/>
      <property name="PROCESS_IDENTIFIER" value="order-{uri.var.orderId}"/>
      <class name="com.ycr.wso2.mediator.statetracker.StateTrackerMediator"/>
      
      <!-- If already running, reject the duplicate request -->
      <filter source="get-property('PROCESS_IS_RUNNING')" regex="true">
        <then>
          <payloadFactory media-type="json">
            <format>{"error": "Order is already being processed"}</format>
          </payloadFactory>
          <respond/>
        </then>
      </filter>
      
      <!-- Mark process as started -->
      <property name="STATE_TRACKER_OPERATION" value="START_PROCESS"/>
      <property name="PROCESS_IDENTIFIER" value="order-{uri.var.orderId}"/>
      <property name="PROCESS_STATE_EXPIRY_TIME" value="7200"/> <!-- 2 hours -->
      <class name="com.ycr.wso2.mediator.statetracker.StateTrackerMediator"/>
      
      <!-- Process the order -->
      
      
      <!-- Mark process as completed -->
      <property name="STATE_TRACKER_OPERATION" value="STOP_PROCESS"/>
      <property name="PROCESS_IDENTIFIER" value="order-{uri.var.orderId}"/>
      <class name="com.ycr.wso2.mediator.statetracker.StateTrackerMediator"/>
      
      <respond/>
    </inSequence>
  </resource>
</api>
```
### Example 2: Bulk Processing and checking status

In this scenario, we will be processing a bulk load in the background and 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<api context="/bulk" name="ProcessRecordAPI" xmlns="http://ws.apache.org/ns/synapse">
    <resource methods="POST" uri-template="/process">
        <inSequence>
            <propertyGroup>
                <property name="STATE_TRACKER_OPERATION" value="IS_PROCESS_RUNNING"/>
                <property name="PROCESS_IDENTIFIER" value="BuldProcessID12345"/>
            </propertyGroup>
            <class name="org.nmdp.integration.mediator.statetracker.StateTrackerMediator"/>
            <filter regex="false" source="boolean($ctx:PROCESS_IS_RUNNING)">
                <then>
                    <propertyGroup>
                        <property name="INTG_NAME" scope="default" type="STRING" value="BulkBusinessPartyTPPAPI"/>
                        <property expression="get-property('MessageID')" name="uuid" scope="default" type="STRING"/>
                        <property value="Bulk records been triggered" name="Message" scope="default" type="STRING"/>
                    </propertyGroup>
                    <!-- Accept the bulk request and run in the background -->
                    <clone continueParent="false" id="CloneTpp">
                        <target>
                            <sequence>
                                <payloadFactory media-type="json">
                                    <format>
                                        {
                                        "Message": {
                                        "ReturnStatus": "Processing",
                                        "ReturnMessage": "Bulk request is accepted for processing, you can track the logs using TraceID.",
                                        "TraceID": "$1"
                                        }
                                        }
                                    </format>
                                    <args>
                                        <arg evaluator="xml" expression="$ctx:uuid"/>
                                    </args>
                                </payloadFactory>
                                <property name="HTTP_SC" value="202" scope="axis2"/>
                                <respond description="respondToClient"/>
                            </sequence>
                        </target>
                        <target>
                            <sequence>
                                <sequence key="ProcessYourRecords"/>
                            </sequence>
                        </target>
                    </clone>
                </then>
                <else>
                    <payloadFactory media-type="json">
                        <format>
                            {
                            "Message": {
                            "ReturnStatus": "Processing",
                            "ReturnMessage": "Bulk request is still in progress. Please try after sometime."
                            }
                            }
                        </format>
                        <args></args>
                    </payloadFactory>
                    <property name="HTTP_SC" value="409" scope="axis2"/>
                    <respond description="respondToClient"/>
                </else>
            </filter>
       </inSequence>
       <outSequence>
       </outSequence>
       <faultSequence>
                <property name="PROCESS_IDENTIFIER" value="BuldProcessID12345"/>
                <property name="STATE_TRACKER_OPERATION" value="STOP_PROCESS"/>
                <class name="org.nmdp.integration.mediator.statetracker.StateTrackerMediator"/>
                <payloadFactory media-type="json">
                    <format>
                        {
                        "ReturnMessage": "Error"
                        }
                    </format>
                    <args/>
                </payloadFactory>
                <property name="HTTP_SC" value="400" scope="axis2"/>
                <respond description="respondToClient"/>
        </faultSequence>
    </resource>
    <!-- Resource to monitor status -->
    <resource methods="GET" uri-template="/process_bulk_status">
        <inSequence>
            <propertyGroup>
                <property name="STATE_TRACKER_OPERATION" value="IS_PROCESS_RUNNING"/>
                <property name="PROCESS_IDENTIFIER" value="BuldProcessID12345"/>
            </propertyGroup>
            <class name="org.nmdp.integration.mediator.statetracker.StateTrackerMediator"/>
            <filter regex="false" source="boolean($ctx:PROCESS_IS_RUNNING)">
                <then>
                    <payloadFactory media-type="json">
                        <format>
                            {
                                "Message": {
                                    "ReturnStatus": "No Process running",
                                    "ReturnMessage": "There are no active jobs running currently."
                                    "IsRunning": "false"
                                }
                            }
                        </format>
                        <args></args>
                    </payloadFactory>
                    <property name="HTTP_SC" value="200" scope="axis2"/>
                </then>
                <else>
                    <payloadFactory media-type="json">
                        <format>
                            {
                                "Message": {
                                    "ReturnStatus": "Processing",
                                    "ReturnMessage": "Bulk request is still in progress. Please try after sometime.",
                                    "IsRunning": "true"
                                }
                            }
                        </format>
                        <args></args>
                    </payloadFactory>
                    <property name="HTTP_SC" value="409" scope="axis2"/>
                </else>
            </filter>
            <respond/>
        </inSequence>
        <outSequence/>
        <faultSequence/>
    </resource>
</api>
```

## Design Considerations

### Thread Safety

The mediator is designed to be thread-safe, making it suitable for high-concurrency environments. It uses appropriate synchronization mechanisms to ensure that state updates are atomic and consistent.

### In-Memory Storage

The current implementation uses in-memory storage, which is:
- **Fast**: No I/O overhead
- **Simple**: Easy to deploy and use
- **Suitable for**: Single server deployments and clustered environments with session replication

### Future Extensibility

The architecture is designed for extensibility. You can implement custom storage backends for:
- **Database Storage**: For persistence and cross-instance sharing
- **Registry Storage**: Leveraging WSO2's registry for distributed state

## Building and Deploying

### Prerequisites
- Java 11 or higher
- Maven 3.6+
- WSO2 MI version compatible with your mediator

### Building from Source

```bash
git clone https://github.com/yasassri/wso2-state-tracker-midiator.git
cd wso2-state-tracker-midiator
mvn clean install
```

### Deploying

1. Copy the compiled JAR to `<WSO2_HOME>/lib/`
2. Restart the WSO2 server
3. Use the mediator in your integration flows as shown above

## Limitations

Currently, the mediator:
- Uses in-memory storage (suitable for single instances) but this can be easily extended to use a datastore. 

## Conclusion

The WSO2 State Tracker Mediator is a lightweight yet effective solution for managing long-running processes and preventing duplicate executions in your integration workflows. By providing a simple API for state management, it helps you build more robust, predictable, and maintainable integration solutions.

Whether you're handling order processing, invoice reconciliation, or any other long-running business process, this mediator gives you the control and visibility you need to ensure reliable execution.

Have you faced challenges with duplicate processing or long-running workflows? Try the State Tracker Mediator and share your experience in the comments below!

**Repository**: [yasassri/wso2-state-tracker-midiator](https://github.com/yasassri/wso2-state-tracker-midiator)  
**License**: Apache License 2.0  

---