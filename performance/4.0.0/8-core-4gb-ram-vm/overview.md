#Sherlock VM Based Testing

##Hardware Setup

**Server Specs:**

* Centos 6 VM
* 8 Core ()
* 8GB RAM
* 2 Servers

**Client Specs:**

* Centos 6 VM
* 4 Core ()
* 4GB RAM
* 1 Server

##Testing

###Baseline Tests

Baseline tests were ran for 15 minutes each and items remaining in the replication queue (items remaining) as well as network utilization (bytes sent per second) were measured. The intent was to find the point at which items remaining to be replicated became large or varied highly. We consider the expected amount of items to be remaining for replication at any given time to be 15% of the incoming sets/sec to be an expected scenario. Each test uses value sizes of 1KB. Note that stats are collected across all nodes. For example this means and 1000 items remaining on average means means 500 items reminaing on each node given a two node cluster.

	Replication Items Remaining (Avg/Max)

	|--------|--------------|-------------|-------------|--------------------|
	|Ops/Sec | Replication  | Replication | Replication | Replication, Views |
	|        |              | and Views   | and XDCR    | and XDCR           |
    |--------|--------------|-------------|-------------|--------------------|
	| 6,000  |              |             |  179/1026   | 
	|--------|--------------|-------------|-------------|--------------------|
	| 9,000  | 184/15772    |             |  487/23333  |  
	|--------|--------------|-------------|-------------|--------------------|
	| 10,000 | 738/26943    | 6910/31252  |  810/30244  |
	|--------|--------------|-------------|-------------|--------------------|
	| 11,000 | 1262/45704   | 19508/67336 | 1050/35956  |
	|--------|--------------|-------------|-------------|--------------------|
	| 12,000 | 1246/31390   |
	|--------|--------------|-------------|-------------|--------------------|
	| 13,000 | 3643/56245   |
	|--------|--------------|-------------|-------------|--------------------|
	| 14,000 | 13231/98963  |
	|--------|--------------|-------------|-------------|--------------------|
	| 15,000 | 13140/95863  |
	|--------|--------------|-------------|-------------|--------------------|

	Replication Bytes/Sec in MB (Avg/Max/Min)

	|--------|-------------|-------------|-------------|--------------------|
	|Ops/Sec | Replication | Replication | Replication | Replication, Views |
	|        |             | and Views   | and XDCR    | and XDCR           |
    |--------|-------------|-------------|-------------|--------------------|
	| 6,000  |             |             | 7.4/12.3/1.7|               
	|--------|-------------|-------------|-------------|--------------------|
	| 9,000  | 11.2/14.3/1 |             | 11/16.7/2.5 |
	|--------|-------------|-------------|-------------|--------------------|
	| 10,000 |  12.3/31/1  | 11.7/22.6/2 | 12.2/15/2.1 |
	|--------|-------------|-------------|-------------|--------------------|
	| 11,000 | 13.3/28.9/2 |             | 13.5/24/2   |
	|--------|-------------|-------------|-------------|--------------------|
	| 12,000 | 14.6/29/2   |
	|--------|-------------|-------------|-------------|--------------------|
	| 13,000 | 14.2/25/2   |
	|--------|-------------|-------------|-------------|--------------------|
	| 14,000 | 14/29.8/0   |
	|--------|-------------|-------------|-------------|--------------------|
	| 15,000 | 13.9/29.5/0 |
	|--------|-------------|-------------|-------------|--------------------|


	Replication Latency in Milliseconds (80th/95th/99th)

	|--------|----------------|--------------|----------------|--------------------|
	|Ops/Sec | Replication    | Replication  | Replication    | Replication, Views |
	|        |                | and Views    | and XDCR       | and XDCR           |
    |--------|----------------|--------------|----------------|--------------------|
	| 6,000  |                |              | 54/432/950     |
	|--------|----------------|--------------|----------------|--------------------|
	| 9,000  | 30/735/986     |              | 43/615/1010    |
	|--------|----------------|--------------|----------------|--------------------|
	| 10,000 | 33/790/1036    |958/2186/4709 | 68/683/1034    |
	|--------|----------------|--------------|----------------|--------------------|
	| 11,000 | 42/784/1085    |              | 74/755/1061    |
	|--------|----------------|--------------|----------------|--------------------|
	| 12,000 | 57/751/1054    |
	|--------|----------------|--------------|----------------|--------------------|
	| 13,000 | 90/873/5396    |
	|--------|----------------|--------------|----------------|--------------------|
	| 14,000 | 342/3984/7172  |
	|--------|----------------|--------------|----------------|--------------------|
	| 15,000 | 480/3314/7015  |
	|--------|----------------|--------------|----------------|--------------------|

Detailed Test Results:

* [Replication Only](rep-only.md)
* [Replication with Views](rep-views.md)
* [Replication with XDCR](rep-xdcr.md)
* [Replication with Views and XDCR](rep-views-xdcr.md)

### Tests with external clients

Tests with external clients were ran for 15 minutes each and items remaining in the replication queue (items remaining) as well as network utilization (bytes sent per second) were measured. The intent was to find the point at which items remaining to be replicated became large or varied highly in the presence of different number of external clients.

	Replication Items Remaining (Avg/Max)

	|--------|--------------|-------------|-------------|--------------|--------------|--------------|
	|Ops/Sec | 1 Client     | 5 Clients   | 10 Clients  | 25 Clients   | 50 Clients   | 100 Clients  |
	|--------|--------------|-------------|-------------|--------------|--------------|--------------|
	| 11,000 | 
	|--------|--------------|-------------|-------------|--------------|--------------|--------------|
	| 12,000 | 
	|--------|--------------|-------------|-------------|--------------|--------------|--------------|
	| 13,000 | 
	|--------|--------------|-------------|-------------|--------------|--------------|--------------|
	| 14,000 | 
	|--------|--------------|-------------|-------------|--------------|--------------|--------------|

	Replication Bytes/Sec in MB (Avg/Max/Min)

	|--------|-------------|-------------|-------------|-------------|-------------|-------------|
	|Ops/Sec | 1 Client    | 5 Clients   | 10 Clients  | 25 Clients  | 50 Clients  | 100 Clients |
	|--------|-------------|-------------|-------------|-------------|-------------|-------------|
	| 11,000 | 
	|--------|-------------|-------------|-------------|-------------|-------------|-------------|
	| 12,000 | 
	|--------|-------------|-------------|-------------|-------------|-------------|-------------|
	| 13,000 | 
	|--------|-------------|-------------|-------------|-------------|-------------|-------------|
	| 14,000 | 
	|--------|-------------|-------------|-------------|-------------|-------------|-------------|


	Replication Latency in Milliseconds (80th/95th/99th)

	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|
    |Ops/Sec | 1 Client       | 5 Clients    | 10 Clients   | 25 Clients      |  50 Clients     | 100 Clients     |
	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|
	| 11,000 | 
	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|
	| 12,000 | 
	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|
	| 13,000 | 
	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|
	| 14,000 | 
	|--------|----------------|--------------|--------------|-----------------|-----------------|-----------------|

* [Replication with 1 client](rep-1_client.md)
* [Replication with 5 clients](rep-5_clients.md)
* [Replication with 10 clients](rep-10_clients.md)
* [Replication with 25 clients](rep-25_clients.md)
* [Replication with 50 clients](rep-50_clients.md)
* [Replication with 100 clients](rep-100_clients.md)