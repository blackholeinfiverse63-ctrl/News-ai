# Scheduler Run Policy Document

## Overview

This document specifies the operational policies and run guidelines for the News AI Backend scheduler. It defines how scheduled jobs are executed, prioritized, and managed within the system.

## Run Policy Principles

### 1. Asynchronous Execution
- All scheduled jobs are submitted to the background queue for asynchronous processing
- Direct pipeline execution is avoided to prevent blocking the scheduler
- Job status tracking is maintained through queue job IDs

### 2. Priority-Based Processing
Jobs are prioritized based on news category importance:
- **Live News**: Priority 10 (highest) - Breaking news requiring immediate attention
- **Finance News**: Priority 7 - Market-moving information
- **World News**: Priority 5 - International developments
- **Regional News**: Priority 3 - Local/regional updates
- **Kids News**: Priority 1 (lowest) - Educational content

### 3. Resource Management
- **Max Concurrent Jobs**: 3 simultaneous executions
- **Job Staggering**: 2-minute intervals between sources within a category
- **Misfire Handling**: 30-second grace period for missed executions
- **Coalesce**: Missed runs are combined to prevent backlog

### 4. Error Handling
- Failed jobs are logged but do not halt the scheduler
- Automatic retry logic is not implemented (jobs are fire-and-forget)
- Monitoring systems track failure rates and patterns

## Category-Specific Run Policies

### Live News (`live`)
- **Trigger**: Every 15 minutes (`*/15 * * * *`)
- **Execution**: Immediate queue submission with high priority
- **Options**: BHIV push enabled, audio generation enabled, urgent voice
- **Failure Impact**: High - monitor closely for breaking news delays

### Finance News (`finance`)
- **Trigger**: Hourly (`0 * * * *`)
- **Execution**: Staggered submission (2-minute intervals between sources)
- **Options**: BHIV push enabled, professional voice
- **Failure Impact**: Medium - market timing sensitive

### World News (`world`)
- **Trigger**: Every 6 hours (`0 */6 * * *`)
- **Execution**: Sequential processing with medium priority
- **Options**: BHIV push enabled, neutral voice
- **Failure Impact**: Medium - international news cycle

### Regional News (`regional`)
- **Trigger**: Every 6 hours (`0 */6 * * *`)
- **Execution**: Batch processing with low priority
- **Options**: BHIV push enabled, conversational voice
- **Failure Impact**: Low - local news has longer shelf life

### Kids News (`kids`)
- **Trigger**: Every 6 hours (`0 */6 * * *`)
- **Execution**: Off-peak processing with lowest priority
- **Options**: BHIV push enabled, friendly voice, audio disabled
- **Failure Impact**: Low - educational content timing flexible

## Manual Trigger Policies

### Single Source Trigger
- **Endpoint**: `/api/scheduler/trigger?category={category}&source_url={url}`
- **Policy**: Immediate execution bypassing normal schedule
- **Use Case**: Testing specific sources or urgent manual processing

### Category Trigger
- **Endpoint**: `/api/scheduler/trigger?category={category}`
- **Policy**: Process all sources in category immediately
- **Use Case**: Force refresh of entire category

### Full System Trigger
- **Endpoint**: `/api/scheduler/trigger`
- **Policy**: Process one source from each category
- **Use Case**: System health check or manual full cycle

## Queue Integration Policies

### Job Submission
- All scheduled jobs use unified pipeline endpoint (`/v1/run_pipeline`)
- Category-specific options are applied automatically
- Job metadata includes category, source, and priority information

### Monitoring
- Job status tracked via `/api/queue/job/{job_id}`
- Queue statistics available at `/api/queue/stats`
- Failed jobs logged with full context for debugging

### Resource Limits
- Queue size limited to prevent memory exhaustion
- Old completed jobs automatically cleaned up
- Queue health monitored for backpressure indicators

## Operational Guidelines

### Startup Procedure
1. Scheduler initializes with UTC timezone
2. Jobs are added to memory store (lost on restart)
3. Queue connection verified before starting jobs
4. Initial stagger applied to prevent startup spike

### Shutdown Procedure
1. Graceful shutdown signal stops new job scheduling
2. Running jobs allowed to complete
3. Queue notified of scheduler unavailability
4. Clean shutdown logged

### Maintenance Windows
- No scheduled maintenance windows defined
- Rolling updates supported through queue buffering
- Failed jobs can be manually retriggered during maintenance

## Monitoring and Alerting

### Key Metrics
- Job execution success rate per category
- Queue depth and processing times
- Scheduler uptime and restart frequency
- Misfire events and coalesce occurrences

### Alert Thresholds
- >5% job failure rate triggers warning
- Queue depth >50 triggers alert
- Scheduler downtime >5 minutes triggers critical alert

### Logging
- All job submissions logged with full context
- Failures logged with stack traces
- Performance metrics logged every 5 minutes

## Version Control

This run policy is versioned with the API contract. Changes require:
1. Documentation update
2. Code review
3. Testing in staging environment
4. Gradual rollout with monitoring