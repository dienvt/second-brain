# Domain Models Dependency Diagram

  

This diagram represents the domain models and their relationships in the Digima Backend Email API, following Clean Architecture principles.

  

## Core Domain Models Structure

  

```mermaid

graph TB

%% Connection Domain Models

subgraph "Connection Domain"

Connection[Connection]

Provider[Provider]

Authorization[Authorization]

ProviderDetail[ProviderDetail]

ServerSetting[ServerSetting]

  

%% Connection relationships

Connection --> ProviderDetail

Connection --> ServerSetting

Connection --> Authorization

Provider --> ServerSetting

Authorization --> ProviderDetail

end

  

%% Email Domain Models

subgraph "Email Domain"

Email[Email]

Envelope[Envelope]

Thread[Thread]

Attachment[Attachment]

AttachmentResource[AttachmentResource]

Link[Link]

Click[Click]

Open[Open]

EnvelopeAddress[EnvelopeAddress]

EnvelopeUser[EnvelopeUser]

  

%% Email relationships

Email --> AttachmentResource

Envelope --> AttachmentResource

Envelope --> Link

Envelope --> Click

Envelope --> Open

Envelope --> EnvelopeAddress

Envelope --> EnvelopeUser

Thread --> Envelope

Link --> Click

AttachmentResource --> Attachment

end

  

%% Usage Domain Models

subgraph "Usage Domain"

FeatureUsage[FeatureUsage]

Usage[Usage]

  

%% Usage relationships

FeatureUsage --> Usage

end

  

%% Cross-domain relationships

Connection -.->|"User owns"| Email

Connection -.->|"User owns"| Envelope

Authorization -.->|"User has"| Connection

  

%% Styling

classDef connectionClass fill:#e1f5fe,stroke:#01579b,stroke-width:2px

classDef emailClass fill:#f3e5f5,stroke:#4a148c,stroke-width:2px

classDef usageClass fill:#e8f5e8,stroke:#1b5e20,stroke-width:2px

  

class Connection,Provider,Authorization,ProviderDetail,ServerSetting connectionClass

class Email,Envelope,Thread,Attachment,AttachmentResource,Link,Click,Open,EnvelopeAddress,EnvelopeUser emailClass

class FeatureUsage,Usage usageClass

```

  

## Detailed Model Relationships

  

```mermaid

erDiagram

%% Connection Domain

Connection {

uint64 id PK

time created_at

time updated_at

bool auto_configure

int status

uint64 user_id FK

string email_address

string credentials_vault_key

string sent_folder_name

string received_folder_name

bool copy_to_sent_folder

string authorization_type

}

  

Provider {

uint64 id PK

time created_at

time updated_at

uint64 user_id FK

string email_address_domain_name

}

  

Authorization {

uint64 id PK

time created_at

time updated_at

string username

string state

string url

uint64 user_id FK

int provider

string credentials_vault_key

string sent_folder_name

string received_folder_name

json folders

json meta

}

  

ProviderDetail {

json protocols

json folders

}

  

ServerSetting {

string server_name

int security_type

int port

}

  

%% Email Domain

Email {

uint64 id PK

time created_at

time updated_at

string status

time scheduled_at

time processed_at

uint64 author_id FK

json sender

json recipients

string subject

string content

json options

json notification_preferences

}

  

Envelope {

uint64 id PK

time created_at

time updated_at

string message_id

string type

string code

time occurred_at

string subject

string content

string content_type

json options

json notification_preferences

string reply_to_message_id

string thread_origin_message_id

bool is_imported

}

  

Thread {

uint64 id PK

time occurred_at

uint64 contact_id FK

string from_addresses

string to_addresses

string subject

string content

string content_type

string thread_origin_message_id

string cc_addresses

string bcc_addresses

uint64 envelopes_count

bool has_reply

}

  

Attachment {

uint64 id PK

time created_at

time updated_at

string mime_type

string file_name

string file_path

uint64 file_size

string fingerprint

}

  

AttachmentResource {

uint64 id PK

uint64 attachment_id FK

string resource_type

uint64 resource_id FK

}

  

Link {

uint64 id PK

time created_at

time updated_at

string url

string code

string title

uint64 envelope_id FK

}

  

Click {

uint64 id PK

time created_at

time updated_at

uint64 envelope_id FK

uint64 link_id FK

time clicked_at

string ip_address

string user_agent

}

  

Open {

uint64 id PK

time created_at

time updated_at

uint64 envelope_id FK

time opened_at

string ip_address

string user_agent

bool is_reopen

}

  

EnvelopeAddress {

uint64 id PK

time created_at

time updated_at

uint64 envelope_id FK

string email_address

string name

string type

}

  

EnvelopeUser {

uint64 id PK

time created_at

time updated_at

uint64 envelope_id FK

uint64 user_id FK

string role

}

  

%% Usage Domain

FeatureUsage {

json attachments

json envelopes

time from

time to

time calculated_at

}

  

Usage {

uint64 storage_bytes

uint64 resource_count

}

  

%% Relationships

Connection ||--o{ Email : "user_owns"

Connection ||--o{ Envelope : "user_owns"

Authorization ||--|| Connection : "authorizes"

Provider ||--|| ServerSetting : "has_smtp_config"

Provider ||--|| ServerSetting : "has_imap_config"

Connection ||--|| ProviderDetail : "has_details"

Connection ||--|| ServerSetting : "has_smtp_config"

Connection ||--|| ServerSetting : "has_imap_config"

  

Email ||--o{ AttachmentResource : "has_attachments"

Envelope ||--o{ AttachmentResource : "has_attachments"

AttachmentResource ||--|| Attachment : "references"

Envelope ||--o{ Link : "contains_links"

Envelope ||--o{ Click : "tracks_clicks"

Envelope ||--o{ Open : "tracks_opens"

Envelope ||--o{ EnvelopeAddress : "has_addresses"

Envelope ||--o{ EnvelopeUser : "has_users"

Thread ||--o{ Envelope : "contains_envelopes"

Link ||--o{ Click : "tracks_clicks"

  

FeatureUsage ||--|| Usage : "has_attachment_usage"

FeatureUsage ||--|| Usage : "has_envelope_usage"

```

  

## Package Structure Overview

  

```mermaid

graph LR

subgraph "Domain Layer"

subgraph "Models"

CM[Connection Models]

EM[Email Models]

UM[Usage Models]

end

  

subgraph "Services"

CS[Connection Services]

ES[Email Services]

US[Usage Services]

end

  

subgraph "Repositories"

CR[Connection Repositories]

ER[Email Repositories]

UR[Usage Repositories]

end

end

  

subgraph "Infrastructure Layer"

DB[(Database)]

Cache[(Cache)]

Storage[(Storage)]

end

  

subgraph "Interface Layer"

API[gRPC API]

Commands[Commands]

Listeners[Event Listeners]

end

  

%% Dependencies

API --> CS

API --> ES

API --> US

Commands --> CS

Commands --> ES

Listeners --> CS

Listeners --> ES

  

CS --> CR

ES --> ER

US --> UR

  

CR --> DB

ER --> DB

UR --> DB

  

CS --> Cache

ES --> Storage

  

%% Model dependencies

CS --> CM

ES --> EM

US --> UM

  

classDef domainClass fill:#e3f2fd,stroke:#1976d2,stroke-width:2px

classDef infraClass fill:#f3e5f5,stroke:#7b1fa2,stroke-width:2px

classDef interfaceClass fill:#e8f5e8,stroke:#388e3c,stroke-width:2px

  

class CM,EM,UM,CS,ES,US,CR,ER,UR domainClass

class DB,Cache,Storage infraClass

class API,Commands,Listeners interfaceClass

```

  

## Key Architectural Patterns

  

### 1. **Clean Architecture Separation**

  

- **Domain Layer**: Contains business logic and models

- **Infrastructure Layer**: Handles external dependencies (database, cache, storage)

- **Interface Layer**: Manages external communication (API, commands, events)

  

### 2. **Model Relationships**

  

- **Connection Domain**: Manages email provider connections and authentication

- **Email Domain**: Handles email processing, tracking, and threading

- **Usage Domain**: Tracks feature usage and resource consumption

  

### 3. **Polymorphic Relationships**

  

- `AttachmentResource` uses polymorphic associations with both `Email` and `Envelope`

- `ProviderDetail` stores flexible JSON configurations for different email providers

  

### 4. **Event-Driven Architecture**

  

- Models trigger domain events for cross-cutting concerns

- Listeners handle side effects and external integrations

  

### 5. **Repository Pattern**

  

- Each domain has its own repository interfaces

- Infrastructure implements concrete repository classes

- Services orchestrate business logic using repositories

  

This architecture ensures:

  

- **Separation of Concerns**: Each layer has a specific responsibility

- **Testability**: Business logic can be tested independently

- **Flexibility**: External dependencies can be easily swapped

- **Maintainability**: Clear boundaries between different parts of the system