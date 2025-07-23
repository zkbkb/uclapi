# UCL API - Complete API Endpoints Reference

## Overview
This document provides a comprehensive reference for all available API endpoints in UCL API v1.4.11. Each endpoint includes request parameters, response formats, and example usage.

## Authentication Methods

### 1. API Token Authentication
Used for non-personal data endpoints.
```
GET https://uclapi.com/endpoint?token=YOUR_API_TOKEN
```

### 2. OAuth Token Authentication
Used for personal data endpoints.
```
GET https://uclapi.com/endpoint?token=YOUR_OAUTH_TOKEN&client_secret=YOUR_CLIENT_SECRET
```

## Room Bookings API

### GET /roombookings/rooms
Get all bookable rooms.

**Parameters:**
- `token` (required): Your API token
- `roomid` (optional): Filter by specific room ID
- `roomname` (optional): Filter by room name
- `siteid` (optional): Filter by site ID
- `sitename` (optional): Filter by site name
- `classification` (optional): Filter by room classification
- `capacity` (optional): Minimum room capacity

**Response:**
```json
{
  "ok": true,
  "rooms": [
    {
      "roomid": "G08",
      "roomname": "Chadwick Building G08",
      "siteid": "086",
      "sitename": "Chadwick Building",
      "classification": "CR",
      "classification_name": "Classroom",
      "capacity": 50,
      "automated": "Y",
      "location": {
        "coordinates": {
          "lat": "51.524699",
          "lng": "-0.13366"
        },
        "address": ["Gower Street", "London", "WC1E 6BT"]
      }
    }
  ]
}
```

### GET /roombookings/bookings
Get room bookings.

**Parameters:**
- `token` (required): Your API token
- `roomid` (optional): Filter by room ID
- `siteid` (optional): Filter by site ID
- `date` (optional): Date in YYYYMMDD format
- `start_datetime` (optional): ISO 8601 datetime
- `end_datetime` (optional): ISO 8601 datetime
- `results_per_page` (optional): Number of results (default: 1000)
- `page_token` (optional): Pagination token

**Response:**
```json
{
  "ok": true,
  "bookings": [
    {
      "slotid": 1234567,
      "end_time": "2024-01-15T11:00:00+00:00",
      "description": "COMP0001 Lecture",
      "roomid": "G08",
      "siteid": "086",
      "roomname": "Chadwick Building G08",
      "sitename": "Chadwick Building",
      "start_time": "2024-01-15T10:00:00+00:00",
      "contact": "Dr. Smith",
      "weeknumber": 20
    }
  ],
  "next_page_exists": true,
  "page_token": "next_page_token",
  "count": 50
}
```

### GET /roombookings/equipment
Get equipment available in a room.

**Parameters:**
- `token` (required): Your API token
- `roomid` (required): Room ID
- `siteid` (required): Site ID

**Response:**
```json
{
  "ok": true,
  "equipment": [
    {
      "type": "Projector",
      "description": "Ceiling mounted projector",
      "units": 1
    },
    {
      "type": "Whiteboard",
      "description": "Wall mounted whiteboard",
      "units": 2
    }
  ]
}
```

### GET /roombookings/freerooms
Find available rooms.

**Parameters:**
- `token` (required): Your API token
- `start_datetime` (required): ISO 8601 datetime
- `end_datetime` (required): ISO 8601 datetime
- `capacity` (optional): Minimum capacity

**Response:**
```json
{
  "ok": true,
  "free_rooms": [
    {
      "roomid": "G08",
      "roomname": "Chadwick Building G08",
      "siteid": "086",
      "sitename": "Chadwick Building",
      "classification": "CR",
      "capacity": 50
    }
  ]
}
```

## Timetable API

### GET /timetable/personal
Get personal timetable (OAuth required).

**Parameters:**
- `token` (required): OAuth token
- `client_secret` (required): Client secret
- `date_filter` (optional): Date in YYYY-MM-DD format

**Response:**
```json
{
  "ok": true,
  "timetable": {
    "2024-01-15": [
      {
        "module": {
          "module_code": "COMP0001",
          "module_name": "Introduction to Programming",
          "department": "Computer Science",
          "department_id": "COMPS_ENG"
        },
        "location": {
          "name": "Chadwick Building G08",
          "address": ["Gower Street", "London", "WC1E 6BT"],
          "site_name": "Chadwick Building"
        },
        "start_time": "10:00",
        "end_time": "11:00",
        "duration": 60,
        "session_type": "Lecture",
        "session_title": "Variables and Data Types",
        "contact": "Dr. Smith",
        "instance": 1
      }
    ]
  }
}
```

### GET /timetable/bymodule
Get timetable by module code.

**Parameters:**
- `token` (required): Your API token
- `module` (required): Module code(s), comma-separated
- `date_filter` (optional): Date in YYYY-MM-DD format

**Response:**
```json
{
  "ok": true,
  "timetable": {
    "2024-01-15": [
      {
        "location": {
          "name": "Roberts Building 106",
          "address": ["Torrington Place", "London", "WC1E 7JE"]
        },
        "module": {
          "module_code": "COMP0001",
          "module_name": "Introduction to Programming",
          "course_owner": "COMPS_ENG"
        },
        "start_time": "14:00",
        "end_time": "16:00",
        "duration": 120,
        "session_type": "Practical",
        "session_title": "Lab Session 1"
      }
    ]
  }
}
```

### GET /timetable/data/courses
Get list of all courses.

**Parameters:**
- `token` (required): Your API token

**Response:**
```json
{
  "ok": true,
  "courses": [
    {
      "course_code": "BENG-COMP",
      "course_name": "Computer Science BEng",
      "department": "Computer Science",
      "faculty": "Engineering Sciences"
    }
  ]
}
```

### GET /timetable/data/modules
Get list of all modules.

**Parameters:**
- `token` (required): Your API token
- `course` (optional): Filter by course code

**Response:**
```json
{
  "ok": true,
  "modules": [
    {
      "module_code": "COMP0001",
      "module_name": "Introduction to Programming",
      "course_owner": "COMPS_ENG",
      "credits": 15
    }
  ]
}
```

## OAuth API

### GET /oauth/authorise
Initiate OAuth authorization flow.

**Parameters:**
- `client_id` (required): Your application's client ID
- `state` (required): Random state for security

**Response:**
Redirects to UCL SSO login page.

### POST /oauth/token
Exchange authorization code for access token.

**Parameters:**
- `client_id` (required): Your application's client ID
- `client_secret` (required): Your application's client secret
- `code` (required): Authorization code from callback

**Response:**
```json
{
  "ok": true,
  "token": "oauth-token-string",
  "scope": "student_number",
  "state": "your-state-value",
  "client_id": "your-client-id"
}
```

### GET /oauth/user/data
Get authenticated user's data.

**Parameters:**
- `token` (required): OAuth token
- `client_secret` (required): Client secret

**Response:**
```json
{
  "ok": true,
  "upi": "zcabcd1",
  "full_name": "John Smith",
  "given_name": "John",
  "cn": "John Smith",
  "department": "Computer Science",
  "email": "john.smith.19@ucl.ac.uk",
  "is_student": true
}
```

### GET /oauth/user/studentnumber
Get student number (requires special scope).

**Parameters:**
- `token` (required): OAuth token
- `client_secret` (required): Client secret

**Response:**
```json
{
  "ok": true,
  "student_number": "19012345"
}
```

## Search API

### GET /search/people
Search for UCL staff members.

**Parameters:**
- `token` (required): Your API token
- `query` (required): Search query (name or department)

**Response:**
```json
{
  "ok": true,
  "people": [
    {
      "name": "Dr. Jane Smith",
      "department": "Computer Science",
      "email": "j.smith@ucl.ac.uk",
      "status": "Academic Staff"
    }
  ]
}
```

## Resources API

### GET /resources/desktops
Get desktop PC availability.

**Parameters:**
- `token` (required): Your API token
- `roomid` (required): Room ID
- `siteid` (required): Site ID

**Response:**
```json
{
  "ok": true,
  "desktops": [
    {
      "room_name": "ISD PC Lab 1",
      "total": 50,
      "free": 23,
      "occupied": 27,
      "out_of_service": 0,
      "last_updated": "2024-01-15T10:30:00Z"
    }
  ]
}
```

## Workspaces API

### GET /workspaces/sensors
Get real-time sensor data for study spaces.

**Parameters:**
- `token` (required): Your API token
- `survey_ids` (optional): Comma-separated survey IDs
- `survey_filter` (optional): "all" or specific survey ID

**Response:**
```json
{
  "ok": true,
  "surveys": [
    {
      "id": 46,
      "name": "UCL Science Library",
      "start_time": "2024-01-15T08:00:00+00:00",
      "end_time": "2024-01-15T22:00:00+00:00",
      "active": true,
      "maps": [
        {
          "id": 123,
          "name": "Level 2",
          "image_id": 456,
          "sensors": {
            "occupied": 45,
            "total": 100,
            "percentage": 45.0
          }
        }
      ]
    }
  ],
  "last_updated": "2024-01-15T10:28:00Z"
}
```

### GET /workspaces/maps
Get workspace maps with sensor locations.

**Parameters:**
- `token` (required): Your API token
- `survey_id` (required): Survey ID
- `map_id` (required): Map ID
- `image_scale` (optional): Image scale factor (default: 0.02)

**Response:**
```json
{
  "ok": true,
  "map": {
    "id": 123,
    "name": "Level 2",
    "image_url": "https://uclapi.com/workspace/images/123.png",
    "sensors": [
      {
        "sensor_id": "sensor-001",
        "x": 100,
        "y": 200,
        "occupied": true
      }
    ]
  }
}
```

### GET /workspaces/surveys
Get historical workspace survey data.

**Parameters:**
- `token` (required): Your API token
- `survey_ids` (optional): Comma-separated survey IDs
- `days` (optional): Number of days of history (max: 30)

**Response:**
```json
{
  "ok": true,
  "surveys": [
    {
      "survey_id": 46,
      "name": "UCL Science Library",
      "averages": {
        "monday": [
          {"time": "08:00", "occupancy": 15.2},
          {"time": "09:00", "occupancy": 25.5}
        ]
      }
    }
  ]
}
```

## LibCal API

### GET /libcal/locations
Get bookable library locations.

**Parameters:**
- `token` (required): OAuth token
- `client_secret` (required): Client secret

**Response:**
```json
{
  "ok": true,
  "locations": [
    {
      "id": 123,
      "name": "Main Library",
      "categories": [
        {
          "id": 456,
          "name": "Study Room",
          "spaces": [
            {
              "id": 789,
              "name": "Study Room 1",
              "capacity": 6
            }
          ]
        }
      ]
    }
  ]
}
```

### GET /libcal/available
Check space availability.

**Parameters:**
- `token` (required): OAuth token
- `client_secret` (required): Client secret
- `space_id` (required): Space ID
- `date` (required): Date in YYYY-MM-DD format

**Response:**
```json
{
  "ok": true,
  "available_slots": [
    {
      "start": "2024-01-15T10:00:00Z",
      "end": "2024-01-15T11:00:00Z"
    }
  ]
}
```

### POST /libcal/reserve
Make a reservation.

**Parameters:**
- `token` (required): OAuth token
- `client_secret` (required): Client secret
- `space_id` (required): Space ID
- `start` (required): Start time (ISO 8601)
- `end` (required): End time (ISO 8601)

**Response:**
```json
{
  "ok": true,
  "booking_id": "ABC123",
  "confirmation": {
    "space": "Study Room 1",
    "start": "2024-01-15T10:00:00Z",
    "end": "2024-01-15T11:00:00Z"
  }
}
```

### GET /libcal/mybookings
Get user's bookings.

**Parameters:**
- `token` (required): OAuth token
- `client_secret` (required): Client secret

**Response:**
```json
{
  "ok": true,
  "bookings": [
    {
      "booking_id": "ABC123",
      "space": "Study Room 1",
      "location": "Main Library",
      "start": "2024-01-15T10:00:00Z",
      "end": "2024-01-15T11:00:00Z",
      "status": "confirmed"
    }
  ]
}
```

## Common Response Elements

### Success Response
```json
{
  "ok": true,
  "data": {...},
  "_metadata": {
    "last_modified": "2024-01-15T10:30:00Z"
  }
}
```

### Error Response
```json
{
  "ok": false,
  "error": "Error message",
  "error_code": "ERROR_CODE"
}
```

### Rate Limit Response (HTTP 429)
```json
{
  "ok": false,
  "error": "Rate limit exceeded",
  "retry_after": 3600
}
```

## Error Codes

- `TOKEN_INVALID`: Invalid or missing API token
- `TOKEN_EXPIRED`: OAuth token has expired
- `INSUFFICIENT_SCOPE`: OAuth token lacks required scope
- `INVALID_PARAMETERS`: Missing or invalid request parameters
- `RESOURCE_NOT_FOUND`: Requested resource not found
- `RATE_LIMIT_EXCEEDED`: Daily rate limit exceeded
- `INTERNAL_ERROR`: Server error

## Best Practices

1. **Caching**: Respect the `Last-Modified` header and implement appropriate caching
2. **Rate Limiting**: Monitor your usage and implement backoff strategies
3. **Error Handling**: Always check the `ok` field before processing data
4. **Pagination**: Use page tokens for large result sets
5. **Date Formats**: Use ISO 8601 for datetime parameters
6. **Security**: Never expose your `client_secret` in client-side code

---

*For the latest updates and interactive documentation, visit https://uclapi.com/docs*