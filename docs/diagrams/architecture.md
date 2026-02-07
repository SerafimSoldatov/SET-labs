## Product choice

Name: Telegram
Link: <https://web.telegram.org/>
Description: A cloud-based instant messaging app that allows users to send messages, photos, videos, and files, create groups and channels, and supports end-to-end encrypted secret chats.

![Telegram Component Diagram](</docs/diagrams/out/telegram/component-diagram/Component Diagram.svg>)

## Main components

Component diagram code: ![Telegram Component Diagram Code](</docs/diagrams/src/telegram/component-diagram.puml>)

Selected components from the diagram:

1. **MTProto Gateway (DC Entry)** - This is the entry point for all client connections to Telegram's data centers. It handles the MTProto protocol, manages connections, and routes requests to appropriate internal services.

2. **Auth & Session Service** - Responsible for user authentication, session management, and maintaining secure connections between clients and Telegram servers.

3. **Message Handling Service** - Processes and routes text messages between users, manages message persistence, and ensures reliable delivery across user devices.

4. **Media & File Service** - Handles the upload, storage, and distribution of media files (photos, videos, documents) with encryption and optimization features.

5. **State Cache (Redis)** - In-memory cache that stores user session states, online statuses, and frequently accessed data to reduce database load and improve response times.

6. **Event Bus (Kafka/Internal)** - Distributed messaging system that handles asynchronous communication between services, enabling event-driven architecture for notifications and updates.

## Data flow

![Telegram Sequence Diagram](</docs/diagrams/out/telegram/sequence-diagram/Sequence Diagram.svg>)

Sequence diagram code: ![Telegram Sequence Diagram Code](/docs/diagrams/src/telegram/sequence-diagram.puml>)

**Group: Media Upload (Encrypted Parts)**

This group shows how Telegram handles media file uploads before sending messages. The process involves:

1. User selects a photo and initiates send
2. Mobile app calls RPC: `saveFilePart(bytes, file_id_A)` to the MTProto Gateway
3. File data is streamed in chunks to the Media Service
4. Each chunk is written to the Distributed File System with checksum verification
5. File metadata (owner information) is saved
6. The process repeats until the entire file is uploaded

**Components interaction:**

- Mobile App ↔ MTProto Gateway (via MTProto protocol)
- MTProto Gateway ↔ Media Service (RPC calls)
- Media Service ↔ Distributed DFS (file storage)
- Media Service saves metadata

**Data exchanged:** File chunks, checksums, file IDs, owner metadata, and acknowledgment messages.

## Deployment

![Telegram Deployment Diagram](</docs/diagrams/out/telegram/deployment-diagram/Deployment%20Diagram.svg>)

Deployment diagram code: [Telegram Deployment Diagram Code](</docs/diagrams/src/telegram/deployment-diagram.puml>)

**Deployment overview:**

Telegram uses a globally distributed infrastructure with:

- **User devices** (smartphones, desktops) running Telegram apps that connect to the nearest data center
- **Edge/Connection Layer** deployed globally to handle incoming connections via MTProto Gateway
- **Primary Data Centers** containing compute clusters (Pods/Containers) running core services (Auth, Message, Media, etc.)
- **In-Memory Cluster** (Redis) for state caching deployed alongside compute clusters
- **Storage Cluster** with distributed databases for persistent data storage
- **External services** integrated via APIs for SMS notifications and push services (FCM/APNs)

Components are deployed across multiple geographic locations to ensure low latency and high availability, with the Primary DC handling core logic and edge nodes managing connections.

## Assumptions

1. **Media deduplication:** I assume Telegram implements file deduplication in its Distributed File System to optimize storage costs when multiple users send the same media file.

2. **Data center synchronization:** I assume Telegram's Primary DC synchronizes with edge data centers to maintain consistency while providing low-latency access to users worldwide.

3. **Encryption at rest:** I assume that while Telegram provides end-to-end encryption for secret chats, regular chats and media are encrypted at rest in their storage systems.

4. **Scalability design:** I assume the Kafka Event Bus is used for asynchronous processing and horizontal scaling, allowing services to handle millions of concurrent users.

## Open questions

1. **Secret chat architecture:** How exactly does the data flow differ for end-to-end encrypted secret chats compared to regular chats in terms of server involvement and key exchange?

2. **Geo-distribution strategy:** What specific strategies does Telegram use for data replication and synchronization between its globally distributed data centers?

3. **Database sharding:** How does Telegram shard its chat and user data across multiple database instances, and what criteria determine the sharding key?

4. **Disaster recovery:** What is Telegram's disaster recovery plan and RTO/RPO targets for different components of their infrastructure?

5. **Monitoring and observability:** What tools and metrics does Telegram use to monitor the health and performance of their distributed system at scale?
