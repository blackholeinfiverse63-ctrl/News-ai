# News AI Scheduler Behavior Documentation

## Overview

The News AI Scheduler is responsible for automated news processing across multiple categories and sources. It uses APScheduler with cron-based triggers to ensure regular content updates while maintaining system stability through background queue processing.

## Architecture

### Components
- **APScheduler**: Core scheduling engine using AsyncIOScheduler
- **Background Queue**: Job processing system with priority queues
- **Unified Pipeline**: News processing pipeline with RL corrections
- **Category-based Configuration**: Different settings per news category

### Key Features
- Cron-based scheduling with UTC timezone
- Priority-based job queuing
- Category-specific pipeline options
- Staggered execution to prevent system overload
- Comprehensive statistics tracking

## Scheduling Configuration

### News Categories and Intervals

| Category | Interval | Sources | Priority | Channels | Avatars | Voice |
|----------|----------|---------|----------|----------|---------|-------|
| **live** | Every 15 minutes (`*/15 * * * *`) | 5 major news sites | 10 (Highest) | `news_channel_live` | `avatar_breaking` | `urgent` |
| **finance** | Every hour (`0 * * * *`) | 5 financial sites | 7 | `news_channel_finance` | `avatar_business` | `professional` |
| **world** | Every 6 hours (`0 */6 * * *`) | 4 world news sites | 5 | `news_channel_world` | `avatar_global` | `neutral` |
| **regional** | Every 6 hours (`0 */6 * * *`) | 4 regional sites | 3 | `news_channel_regional` | `avatar_local` | `conversational` |
| **kids** | Every 6 hours (`0 */6 * * *`) | 4 kids news sites | 1 (Lowest) | `news_channel_kids` | `avatar_fun` | `friendly` |

### Source Lists

#### Live News (15-minute intervals)
- BBC News (`https://www.bbc.com/news`)
- Reuters (`https://www.reuters.com/`)
- New York Times (`https://www.nytimes.com/`)
- Al Jazeera (`https://www.aljazeera.com/`)
- The Guardian (`https://www.theguardian.com/international`)

#### Finance (Hourly)
- Bloomberg (`https://www.bloomberg.com/`)
- Wall Street Journal (`https://www.wsj.com/`)
- Financial Times (`https://www.ft.com/`)
- CNBC (`https://www.cnbc.com/`)
- MarketWatch (`https://www.marketwatch.com/`)

#### World News (6-hour intervals)
- BBC World (`https://www.bbc.com/news/world`)
- Reuters World (`https://www.reuters.com/world/`)
- Al Jazeera News (`https://www.aljazeera.com/news/`)
- DW News (`https://www.dw.com/en/top-stories/s-9097`)

#### Regional News (6-hour intervals)
- The Hindu (`https://www.thehindu.com/`)
- Indian Express (`https://indianexpress.com/`)
- NDTV (`https://www.ndtv.com/`)
- Times of India (`https://timesofindia.indiatimes.com/`)

#### Kids News (6-hour intervals)
- Scholastic (`https://www.scholastic.com/`)
- Time for Kids (`https://www.timeforkids.com/`)
- National Geographic Kids (`https://www.nationalgeographic.com/for-kids/`)
- BBC Newsround (`https://www.bbc.co.uk/newsround`)

## Execution Behavior

### Job Scheduling
1. **Initialization**: Scheduler creates cron jobs for each source in each category
2. **Staggering**: Jobs within the same category are staggered by 2-minute intervals to prevent simultaneous execution
3. **Priority Assignment**: Jobs are assigned priorities based on category importance
4. **Queue Submission**: Scheduled jobs submit work to background queue instead of executing directly

### Pipeline Options by Category

#### Live News Options
```json
{
  "enable_bhiv_push": true,
  "enable_audio": true,
  "channels": ["news_channel_live"],
  "avatars": ["avatar_breaking"],
  "voice": "urgent",
  "force_correction": false
}
```

#### Finance News Options
```json
{
  "enable_bhiv_push": true,
  "enable_audio": true,
  "channels": ["news_channel_finance"],
  "avatars": ["avatar_business"],
  "voice": "professional",
  "force_correction": false
}
```

#### World/Regional News Options
```json
{
  "enable_bhiv_push": true,
  "enable_audio": true,
  "channels": ["news_channel_world"],
  "avatars": ["avatar_global"],
  "voice": "neutral",
  "force_correction": false
}
```

#### Kids News Options
```json
{
  "enable_bhiv_push": true,
  "enable_audio": false,
  "channels": ["news_channel_kids"],
  "avatars": ["avatar_fun"],
  "voice": "friendly",
  "force_correction": false
}
```

## Queue Processing

### Priority System
- **Priority 10**: Live news (highest priority)
- **Priority 7**: Finance news
- **Priority 5**: World news
- **Priority 3**: Regional news
- **Priority 1**: Kids news (lowest priority)

### Background Queue Configuration
- **Max Workers**: 5 concurrent jobs
- **Max Queue Size**: 1000 jobs
- **Job Defaults**:
  - `coalesce`: true (merge missed jobs)
  - `max_instances`: 3 (limit concurrent instances)
  - `misfire_grace_time`: 30 seconds

## Monitoring and Statistics

### Scheduler Statistics
- `jobs_scheduled`: Total jobs created
- `jobs_completed`: Successfully processed jobs
- `jobs_failed`: Failed job executions
- `last_run`: Timestamp of last job execution
- `next_runs`: Upcoming job execution times

### API Endpoints for Monitoring
- `GET /api/scheduler/stats` - Current scheduler status and job list
- `POST /api/scheduler/trigger` - Manual job triggering
- `GET /api/queue/stats` - Background queue status

## Error Handling and Resilience

### Job Failure Handling
- Failed jobs are logged with error details
- Statistics track failure rates
- Jobs continue running even if individual sources fail

### System Stability
- Background queue prevents scheduler blocking
- Priority system ensures critical news processing
- Staggered execution prevents resource spikes

## Manual Operations

### Triggering Manual Runs
```bash
# Trigger all live news sources
POST /api/scheduler/trigger?category=live

# Trigger specific source
POST /api/scheduler/trigger?category=finance&source_url=https://www.bloomberg.com/

# Trigger one source from each category
POST /api/scheduler/trigger
```

### Scheduler Control
```bash
# Start scheduler
POST /api/scheduler/start

# Stop scheduler
POST /api/scheduler/stop

# Get scheduler statistics
GET /api/scheduler/stats
```

## Performance Characteristics

### Expected Load
- **Live News**: ~96 jobs/day (every 15 minutes × 5 sources)
- **Finance**: ~120 jobs/day (every hour × 5 sources)
- **World/Regional/Kids**: ~80 jobs/day each (every 6 hours × 4 sources)
- **Total Daily Jobs**: ~456 jobs/day

### Processing Times
- **Average Job Duration**: 2-5 minutes
- **Peak Concurrent Jobs**: 5 (queue worker limit)
- **Queue Throughput**: 12 jobs/hour maximum

## Configuration Management

### Environment Variables
- Scheduler configuration is hardcoded for production stability
- No environment-based configuration changes allowed
- All timing and source configurations are fixed

### Category Management
- Categories are predefined and cannot be modified at runtime
- Source URLs are static and require code changes for updates
- Pipeline options are category-specific and fixed

## Production Deployment Notes

### Railway Deployment
- Scheduler starts automatically with application
- Uses UTC timezone for consistent scheduling
- Memory-based job store (jobs lost on restart)
- Background queue persists across restarts

### Monitoring Recommendations
- Monitor queue size and worker utilization
- Track job completion rates by category
- Alert on scheduler failures or high failure rates
- Regular review of source URL validity

### Scaling Considerations
- Current configuration supports moderate load
- Queue size limits prevent memory issues
- Worker count can be increased for higher throughput
- Consider database-persisted job store for production reliability

## Troubleshooting

### Common Issues
1. **Jobs not executing**: Check scheduler status and queue worker health
2. **High failure rates**: Verify source URLs and network connectivity
3. **Queue backlog**: Monitor worker utilization and increase if needed
4. **Memory issues**: Check queue size limits and job cleanup

### Diagnostic Commands
```bash
# Check scheduler health
GET /health

# View active jobs
GET /api/scheduler/stats

# Check queue status
GET /api/queue/stats

# View job details
GET /api/queue/job/{job_id}
```

This documentation ensures the scheduler behavior is transparent, predictable, and maintainable for production operations.</content>
</xai:function_call">...