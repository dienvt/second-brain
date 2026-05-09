- [ ] # Digima Backend SMS API - Domain Models Dependencies Diagram

  

## Overview

  

This diagram shows the relationships and dependencies between all models in the `domain/models` directory of the Digima Backend SMS API.

## Entity Relationship Diagram

  

```mermaid

erDiagram

%% Core Message Entities

V2Message {

uint id

uint setting_id

uint user_id

string status

string contact_phone_number_priority

string recipients

string options

string body

string sent_at

string scheduled_at

}

  

V2ContactMessage {

uint id

uint message_id

uint device_id

string device_phone_number

string body

string code

uint contact_id

string contact_type

string phone_number

string phone_number_type

string phone_number_carrier

string mediasms_id

}

  

%% Device Management

Device {

uint id

uint account_id

string phone_number

string daily_dispatch_limit_reached_at

string push_notification_last_checked_at

string token

string name

}

  

%% Settings and Configuration

Setting {

uint id

string account_code

string sms_phone_number

string send_time_restriction

string web_hook_url

}

  

%% Template System

V2Template {

uint id

uint folder_id

string name

string body

string body_searchable

}

  

TemplateFolder {

uint id

string name

}

  

TemplateLabel {

uint id

string name

}

  

%% Link Management

V2Link {

uint id

uint message_id

string code

string url

string fingerprint

}

  

ContactMessageLink {

string code

uint contact_message_id

uint link_id

}

  

%% Click Tracking

V2Click {

uint id

uint link_id

uint contact_message_id

uint recipient_id

}

  

%% Event Tracking

V2Event {

uint id

uint contact_message_id

uint recipient_id

string name

string reason

uint8 provider_sms_parts

int64 occurred_at_timestamp

string provider_event_name

bool is_visible

}

  

%% Recipient Management

V2Recipient {

uint id

string batch_uid

int64 batch_timestamp

uint contact_id

uint message_id

}

  

%% Statistics

V2SmsStatistic {

string id

string created_at

string carrier

string event_type

uint message_id

uint8 provider_sms_parts

uint contact_message_id

}

  

%% Utility Models

ServiceHours {

string from

string to

bool outside_service_hours

}

  

LastReadMessage {

uint user_id

uint contact_id

uint contact_message_id

}

  

%% Relationships - Core Message Flow

V2Message ||--o{ V2ContactMessage : "has many"

V2Message ||--o{ V2Link : "has many"

V2Message ||--o{ V2Recipient : "has many"

Setting ||--o{ V2Message : "has many"

  

%% Device Relationships

Device ||--o{ V2ContactMessage : "sends/receives"

  

%% Template Relationships

TemplateFolder ||--o{ V2Template : "contains"

V2Template }o--o{ TemplateLabel : "many-to-many"

  

%% Link and Click Relationships

V2Link ||--o{ V2Click : "tracked by"

V2Link ||--o{ ContactMessageLink : "referenced by"

V2ContactMessage ||--o{ ContactMessageLink : "contains"

V2ContactMessage ||--o{ V2Click : "tracked by"

  

%% Event Relationships

V2ContactMessage ||--o{ V2Event : "has events"

V2Recipient ||--o{ V2Event : "has events"

  

%% Statistics Relationships

V2ContactMessage ||--o{ V2SmsStatistic : "tracked in"

V2Message ||--o{ V2SmsStatistic : "tracked in"

  

%% Read Status

V2ContactMessage ||--o{ LastReadMessage : "read status"

```

  

## Dependency Analysis

  

### Core Message Flow

  

1. **V2Message** is the central entity representing SMS messages

2. **V2ContactMessage** represents individual message instances sent to contacts

3. **V2Recipient** tracks unique contacts per message batch

4. **Setting** provides configuration for message sending

  

### Device Management

  

- **Device** represents SMS-capable devices with phone numbers

- Connected to **V2ContactMessage** for tracking which device sent/received messages

  

### Template System

  

- **V2Template** provides reusable message templates

- **TemplateFolder** organizes templates hierarchically

- **TemplateLabel** allows tagging templates for categorization

  

### Link Tracking

  

- **V2Link** represents URLs within messages

- **ContactMessageLink** connects links to specific contact messages

- **V2Click** tracks when recipients click on links

  

### Event System

  

- **V2Event** tracks the lifecycle of SMS messages (queued, sent, delivered, etc.)

- Connected to both **V2ContactMessage** and **V2Recipient**

  

### Statistics and Analytics

  

- **V2SmsStatistic** stores aggregated statistics in MongoDB

- Tracks various event types and carrier information

  

### Utility Models

  

- **ServiceHours** defines operational time windows

- **LastReadMessage** tracks user read status

  

## Key Dependencies

  

### Strong Dependencies (Foreign Keys)

  

- V2ContactMessage → V2Message (message_id)

- V2ContactMessage → Device (device_id)

- V2Message → Setting (setting_id)

- V2Link → V2Message (message_id)

- V2Click → V2Link (link_id)

- V2Click → V2ContactMessage (contact_message_id)

- V2Event → V2ContactMessage (contact_message_id)

- V2Recipient → V2Message (message_id)

- V2Template → TemplateFolder (folder_id)

  

### Many-to-Many Relationships

  

- V2Template ↔ TemplateLabel (via pivot table)

  

### Weak Dependencies (References)

  

- V2SmsStatistic references V2Message and V2ContactMessage

- LastReadMessage references V2ContactMessage

  

## Database Considerations

  

### Primary Database (MySQL/PostgreSQL)

  

- Most entities use GORM with soft deletes

- Standard timestamps (created_at, updated_at, deleted_at)

- Foreign key constraints for data integrity

  

### MongoDB Integration

  

- V2SmsStatistic uses MongoDB for analytics

- Implements custom collection and document management

  

### Indexing Strategy

  

- Primary keys on all entities

- Foreign key indexes for performance

- Composite indexes for query optimization

  

## Clean Architecture Compliance

  

This model structure follows Clean Architecture principles:

  

1. **Domain Layer**: All models are in the domain layer

2. **Dependency Direction**: Dependencies point inward toward domain

3. **Entity Independence**: Models are independent of external concerns

4. **Business Logic**: Core business rules are encapsulated in model methods

  

## Usage Patterns

  

### Message Creation Flow

  

1. Create V2Message with Setting

2. Generate V2Recipient records

3. Create V2ContactMessage instances

4. Generate V2Link records if URLs present

5. Track events via V2Event

6. Record statistics in V2SmsStatistic

  

### Template Usage

  

1. Organize templates in TemplateFolder

2. Tag templates with TemplateLabel

3. Use templates to create V2Message instances

  

### Analytics and Reporting

  

1. V2Event provides real-time status tracking

2. V2SmsStatistic provides aggregated analytics

3. V2Click provides engagement metrics