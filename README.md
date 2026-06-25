**AI/ML Cybersecurity Lab 02: Data Exfiltration Anomaly Detection**

**Introduction**

In this lab, I applied Artificial Intelligence (AI) and Machine Learning (ML) concepts to detect potential data exfiltration activity using network traffic data. Data exfiltration is the unauthorized transfer of sensitive information from an organization to an external destination and is a common objective of cyber attackers after gaining access to a system.

The primary goal of this lab was to identify abnormal network behavior that may indicate the theft of organizational data. Using Splunk Enterprise, I analyzed network traffic records containing information such as users, hosts, source and destination IP addresses, bytes transferred, geographic destinations, and network activity patterns. By examining these records, I investigated whether any hosts or users exhibited behavior that significantly deviated from normal activity.

Unlike the previous Authentication Classification lab, which focused on supervised learning and labeled data, this lab introduces anomaly detection and unsupervised learning concepts. Instead of relying on predefined attack labels, anomaly detection focuses on identifying unusual patterns that differ from established baselines. This approach is widely used in Security Operations Centers (SOCs) to detect insider threats, compromised systems, unauthorized data transfers, and advanced persistent threats (APTs).

Using Splunk Enterprise, I ingested the dataset, analyzed network activity, identified outliers, and investigated hosts that transferred unusually large volumes of data. Throughout the lab, I explored how machine learning systems detect anomalies by comparing normal behavior against unusual behavior patterns and how security analysts validate those findings through investigation.

### **Data Exfiltration Anomaly Detection**

**Dataset:**

network_exfil_dataset.csv

\
\
\
\
\
\
\
\
\
**Uploading file into Splunk enterprise**\
\
\
\
**Verification\
\**
\
\
\
\
\
\
Splunk successfully indexed 100 Data Exfiltration Anomaly Detection events from network_exfil_dataset.csv

the dataset through using the query “ source="network_exfil_dataset.csv" host="MacBookPro" index="network" sourcetype="network_exfil" The screenshot displays the first five events as a sample to verify successful ingestion of 100 events of Data Exfiltration Anomaly Detection file.\
\
\
\
**1.** What network traffic events are present in the data exfiltration dataset?\
\
using this query:\
index=network sourcetype=network_exfil\
\| table user host src_ip dest_ip country bytes_sent protocol label\
\
\
\
\
\

I displayed the network traffic dataset fields to verify successful ingestion and review the available information for each event. The output includes the user, host, source IP address, destination IP address, destination country, number of bytes transferred, network protocol, and classification label.

The dataset contains network traffic events with the following fields:

user\
host\
src_ip\
dest_ip\
country\
bytes_sent\
protocol\
label

These fields provide the information necessary to investigate potential data exfiltration activity and identify anomalous network behavior.

**2.** Which host transferred the most data?\
\
Using this query:\
index=network sourcetype=network_exfil\
\| stats sum(bytes_sent) as total_bytes by host\
\| sort - total_bytes\
\
\
I calculated the total volume of data transferred by each host and sorted the results in descending order. This analysis helps identify systems responsible for large amounts of network traffic, which may indicate data exfiltration, backups, file transfers, or other significant network activity.

The host that transferred the most data was **MacBookPro**, with a total of 162,971,464 bytes transferred across all network events in the dataset. I identified MacBookPro as the host responsible for the highest volume of network traffic. However, this result alone does not confirm data exfiltration because MacBookPro is currently the only host represented in the dataset output. Additional analysis is required to determine whether the transferred data volume is normal or anomalous compared to other hosts, users, destinations, or countries.

**.** Is the amount of data unusual?\
Possibly unusual, but not enough evidence yet to classify it as an anomaly. Further investigation is required.\
\
MacBookPro transferred 162,971,464 bytes, which is the highest total data volume observed. However, this query alone does not provide enough information to determine whether the traffic volume is unusual or indicative of data exfiltration.

Additional analysis is required before classifying the activity as anomalous. A high volume of data transfer may be legitimate (backups, file synchronization, software updates, or normal business operations) or malicious (data exfiltration). The traffic should be compared against other users, hosts, and historical baselines to determine whether it represents abnormal behavior.\
\
**3.** Which users transferred the most data?

Using this quesry:

index=network sourcetype=network_exfil\
\| stats sum(bytes_sent) as total_bytes by user\
\| sort - total_bytes

\
I calculated the total amount of data transferred by each user and sorted the results from highest to lowest. This analysis helps identify users whose network activity significantly exceeds that of other users and may indicate abnormal behavior or potential data exfiltration.

The users who transferred the most data were:

**finance_admin** — 69,138,027 bytes\
**backup_admin** — 67,002,604 bytes\
**admin** — 5,367,378 bytes\
\
observed that **finance_admin** and **backup_admin** transferred substantially more data than all other users in the dataset. Both users transferred over **67 million bytes**, while the next highest user (**admin**) transferred only about **5.3 million bytes**.

This large difference suggests that **finance_admin** and **backup_admin** are potential outliers and should be prioritized for investigation. Such unusually large transfers may indicate:

Data exfiltration\
Large backup operations\
Administrative file transfers\
Unauthorized movement of sensitive information

Additional investigation is required to determine whether this activity is legitimate or malicious.\
\
AI/ML Insight
----------------------------------------------------------------------------------------------------

This is the first place where we can reasonably start talking about anomalies.

Compare:

Typical users:\
1–5 million bytes

versus:

finance_admin: 69 million bytes\
backup_admin: 67 million bytes

These two users are roughly **13–50 times larger** than most users in the dataset.

That doesn't prove malicious activity, but from an anomaly-detection perspective:

**finance_admin and backup_admin are strong candidate outliers.**

**\**
\
4. **Which countries received the most data?\
Using this query:\
index=network sourcetype=network_exfil\
\| stats sum(bytes_sent) as total_bytes by country\
\| sort - total_bytes\
\**
\**
calculated the total amount of data transferred to each destination country and sorted the results in descending order. This analysis helps identify geographic destinations receiving unusually large amounts of data and can reveal potential exfiltration targets.

The countries that received the most data were:

**Unknown** — 66,237,606 bytes\
**Russia** — 41,142,909 bytes\
**China** — 34,127,494 bytes

These destinations received significantly more data than the USA, UK, and Canada.

I observed that the majority of transferred data was sent to **Unknown**, **Russia**, and **China**. Together, these destinations account for most of the data volume in the dataset.

From a security perspective, this activity is significantly more suspicious than the transfers to the USA, UK, and Canada because:

The destination is unknown or poorly identified.\
The data volume is substantially higher than normal destinations.\
These destinations are associated with the anomaly records in the dataset.

These countries should be prioritized for further investigation when looking for potential data exfiltration activity.

## AI/ML Insight

This is our first strong anomaly.

Notice the pattern:

USA 9.7 million\
UK 6.8 million\
Canada 4.8 million

versus

Unknown 66.2 million\
Russia 41.1 million\
China 34.1 million

The difference is very large. An anomaly-detection model would likely assign a high anomaly score to these destinations because they deviate substantially from the normal geographic pattern.\
\
5. Which individual network events transferred the largest amount of data?\
\
Using this query:\
\
index=network sourcetype=network_exfil\
\| sort - bytes_sent\
\| table user host country bytes_sent protocol label

\
\
\
\
I sorted all network events by the bytes_sent field in descending order to identify the largest individual data transfers. Large outbound transfers are important because they may indicate data exfiltration, unauthorized backups, bulk file movement, or other suspicious activity.

The largest individual data transfers were:

| **User**      | **Country** | **Bytes Sent** | **Label** |
|---------------|-------------|----------------|-----------|
| backup_admin  | Unknown     | 23,144,378     | Anomaly   |
| finance_admin | Russia      | 22,503,995     | Anomaly   |
| backup_admin  | China       | 19,160,521     | Anomaly   |
| backup_admin  | Russia      | 18,638,914     | Anomaly   |
| finance_admin | Unknown     | 15,588,126     | Anomaly   |

I identified multiple extremely large data transfers associated with the accounts **backup_admin** and **finance_admin**.

Several indicators increase the risk level of these events:

All top events are labeled **Anomaly**.\
The transferred data volumes are dramatically larger than normal events.\
The destinations are **Unknown**, **Russia**, and **China**.\
The transfers use HTTPS and SFTP, both common protocols for moving large amounts of data.\
\
The largest normal event transferred approximately:

494,263 bytes

while the largest anomaly transferred:

23,144,378 bytes

which is approximately **47 times larger**.

This difference strongly suggests abnormal behavior and makes these events prime candidates for a data exfiltration investigation.

## AI/ML Insight

This is where anomaly detection becomes obvious.

Normal events:

~400,000 to 500,000 bytes

Anomalous events:

9 million to 23 million bytes

An ML model doesn't know what "Russia" or "China" means politically.

Instead it sees:

Very large bytes_sent\
Rare destinations\
Rare user behavior

and concludes:

These events are statistically different from the normal population.

That is exactly how anomaly detection systems in UEBA, Splunk MLTK, Darktrace, and many modern SOC platforms work.

**6.** Which users generated the most anomaly events?\
\
Using this query:

index=network sourcetype=network_exfil label=Anomaly\
\| stats count by user\
\| sort – count\
\
\
\
\
\
\
\
\
\
\
\
\
\
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

I filtered the dataset to include only events labeled **Anomaly** and counted the number of anomaly events associated with each user. This helps identify users responsible for the majority of suspicious network activity.The users responsible for the anomaly events were:

**finance_admin** — 5 anomaly events\
**backup_admin** — 4 anomaly events\
**admin** — 1 anomaly event

I found that **finance_admin** and **backup_admin** accounted for **9 of the 10 anomaly events** in the dataset. This is significant because:

These same users previously transferred the largest amounts of data.\
These same users were associated with transfers to **Unknown**, **Russia**, and **China**.\
These same users generated the largest individual network transfers.

The repeated appearance of these users across multiple investigations increases confidence that they represent the primary anomaly sources in the dataset.

## AI/ML Insight

This is where multiple indicators begin to correlate. Earlier we found:

### **High Data Volume**

finance_admin\
backup_admin

### **Suspicious Destinations**

Unknown\
Russia\
China

### **Largest Individual Transfers**

finance_admin\
backup_admin

### Most Anomaly Events

finance_admin\
backup_admin

When multiple independent indicators point to the same entities, an AI/ML system increases its confidence score. In a real UEBA or anomaly-detection platform, **finance_admin** would likely receive the highest risk score in the environment.

**7.** Which network protocols were used by the anomaly events?

Using this query:\
\
index=network sourcetype=network_exfil label=Anomaly\
\| stats count by protocol\
\| sort - count

This helps determine how the data may have been transferred (HTTPS, SFTP, etc.) and is a common step in exfiltration investigations.

\
I filtered the dataset to include only events labeled as Anomaly and counted the number of events associated with each network protocol. Understanding which protocols are used helps determine how data may have been transferred outside the organization.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

The anomaly events used:

**HTTPS** — 5 events\
**SFTP** — 5 events

Both protocols were used equally among the anomaly records.

I observed that all anomaly events used either **HTTPS** or **SFTP**. This is significant because:

**HTTPS** encrypts network traffic and is commonly used for secure web communications, making malicious transfers harder to inspect.

**SFTP** is specifically designed for secure file transfers and is frequently used to move large files between systems.

The exclusive use of HTTPS and SFTP by the anomaly events suggests that the suspicious data transfers were conducted through encrypted channels, which is a common characteristic of real-world data exfiltration activity.

**AI/ML Insight**

Is HTTPS malicious?\
\
HTTPS itself is normal. But the anomaly comes from combining multiple features:

| **Feature** | **Observation**             |
|-------------|-----------------------------|
| User        | finance_admin, backup_admin |
| Country     | Unknown, Russia, China      |
| Bytes Sent  | Extremely large             |
| Protocol    | HTTPS / SFTP                |
| Label       | Anomaly                     |

So, For example: HTTPS alone = Normal but\
\
HTTPS\
+ 23 million bytes\
+ Unknown destination\
+ finance_admin\
becomes highly suspicious.

**\
\
\
\
8. How many events were classified as Normal and how many were classified as Anomaly?**

Using this query:**\
\**
index=network sourcetype=network_exfil\
\| stats count by label

\
\
I counted the number of events associated with each classification label in the dataset. This provides a baseline understanding of the distribution of normal and anomalous network activity before performing further investigations.The dataset contains:
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

**90 Normal events**\
**10 Anomaly events**\
for a total of **100 network traffic events**.

I observed that the dataset is heavily weighted toward normal activity, with only 10% of events classified as anomalies.

This distribution is realistic because:

Most network activity in an organization is legitimate.\
Suspicious or malicious activity typically represents only a small portion of overall traffic.\
Security analysts must identify a small number of anomalies hidden within a much larger volume of normal events.

The anomaly records represent the highest-risk activity in the dataset and should be prioritized for investigation.

## AI/ML Insight

This is exactly how anomaly detection datasets often look:

Normal = 90%\
Anomaly = 10%

A machine-learning system learns:

What is normal?

and then identifies:

What deviates from normal?

### \
In this lab:\
\
**Normal Population**

USA\
UK\
Canada\
Lower data volumes\
Regular users\
Typical network activity

### **Anomalous Population**

finance_admin\
backup_admin\
admin\
Unknown destinations\
Russia\
China\
Extremely large transfers\
HTTPS / SFTP transfers

This contrast between the 90 normal events and 10 anomaly events is what makes anomaly detection possible.

**9.** Which user-country combinations transferred the most anomalous data?\
\
Using this query:

index=network sourcetype=network_exfil label=Anomaly\
\| stats sum(bytes_sent) as total_bytes by user country\
\| sort - total_bytes

\
\
I analyzed only the anomaly events and calculated the total amount of data transferred for each user-country combination. This helps identify which users were responsible for the largest suspicious transfers and where the data was sent. The largest anomalous transfers were:

**finance_admin → Unknown** — 37,725,850 bytes\
**backup_admin → China** — 25,219,312 bytes\
**backup_admin → Unknown** — 23,144,378 bytes\
**finance_admin → Russia** — 22,503,995 bytes

These combinations account for the majority of anomalous data transfers in the dataset.

I identified **finance_admin** and **backup_admin** as the primary sources of anomalous network activity. The investigation revealed:

Extremely large outbound transfers.\
Repeated transfers to **Unknown**, **Russia**, and **China**.\
Multiple anomaly events associated with the same users.\
Use of encrypted protocols (HTTPS and SFTP).\
Data volumes significantly larger than normal network activity.

Based on the available evidence, **finance_admin** represents the highest-risk entity in the dataset, followed closely by **backup_admin**.

In this lab, I investigated 100 network traffic events using Splunk Enterprise to identify potential data exfiltration activity through anomaly detection techniques. I analyzed user behavior, destination countries, transfer volumes, protocols, and anomaly classifications to determine which entities posed the greatest risk.

### **Key Findings**

- 100 total network events analyzed.

- 90 Normal events.

- 10 Anomaly events.

- finance_admin generated 5 anomaly events.

- backup_admin generated 4 anomaly events.

- Largest transfer: 23,144,378 bytes.

- Primary destinations: Unknown, Russia, and China.

- Primary protocols: HTTPS and SFTP.

- Highest-risk user: finance_admin.

### **AI/ML Concepts Anomaly Detection** 

Unsupervised Learning\
Outlier Analysis\
Behavioral Analytics\
Entity-Based Investigation\
Feature Correlation\
Risk Prioritization
