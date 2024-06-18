# Scaling From Zero To Billions Of Requests

## Business Requirements (Real State Company)

### Support rapid user growth
The website should be able to handle an increasing number of visitors searching for properties and agents.

###  Handle high-traffic loads
During peak times, such as weekends or holidays, the system should maintain performance and availability.

###  Fast response times and low latency
Users should experience quick loading of property listings, images, and search results.

### Data consistency and reliability
Property details, user profiles, and transactions should be accurately stored and retrievable.

###  Independent service deployment and scaling
Services like user authentication, property listings, and search should be modular to allow individual scaling and updates.

### Efficient data retrieval and processing
Fast access to property data, including filtering and sorting capabilities.

### Comprehensive monitoring and logging
Track user activities, system performance, and any errors for proactive maintenance and troubleshooting.

### Secure handling of user data
Ensure user information is protected, complying with data privacy laws.


## Initial Single Server Setup
* Browser
* DNS
* Webserver
* Database

## Scaling the server
* Horizonatal or Vertical?
* Stateless vs Statefull Server
* Load balancer

## Database
* Which database to Use?
* Database replication
* Database scaling - Vertical or Horizontal(Sharding)
*
## Cache
* Single Server Cache
* In memory Cache
* Distributed Cache

## CDN
 * Caching static assets
 * Reducing the latency on UI

## Data Centers
* Geo routing
* Data Synchronization
* Failure Safety

## Message Queues
* Async Processing

## Monitoring
* Logging
* Metrics
* Automation

## Conclusion and Next Steps
