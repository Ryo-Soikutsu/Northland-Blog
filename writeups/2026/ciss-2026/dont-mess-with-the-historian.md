---
description: 'Category: OT/ICS'
---

# Don't Mess with the Historian

## Description

{% code overflow="wrap" %}
```
Based on the provided historian CSV, the attacker might appear to have manipulated with the data. Which explanation is best supported by the historian evidence?
- The attacker caused a historian outage, as all process values stopped updating at the same timestamp.
- The attacker turned ON MV201, which caused AIT101 to increase.
- The attacker manipulated the field device is unrelated to the process, as it does not show any meaningful change after the event.
- The attacker turned ON MV101, which caused LIT101 to increase. 
```
{% endcode %}

## Walkthrough

We're provided with a CSV, which contains the states and measurements of several sensors, sorted by timestamp.

<figure><img src="../../.gitbook/assets/image (453).png" alt=""><figcaption></figcaption></figure>

After scrolling through the columns, nothing seems to be out of the ordinary. The values all appear to run in the correct order, and the timestamps show no signs of manipulation or timestomping. The only exception is with <mark style="color:$danger;">**MV101.status**</mark> column, which contains 3 values instead of 2

<figure><img src="../../.gitbook/assets/image (455).png" alt=""><figcaption></figcaption></figure>

Column E contains 3 values indicating state, whereas the other columns only contain 2. Based on the logs available, it seems that 1 indicates that the system is deactivated, whereas 2 indicates that the system is running. This is indicated by the value of LIT101 remaining roughly stable during blocks of status 1, and increasing noticeably during blocks of status 2.  During the blocks of status 0, the value of LIT101 still increases notably, indicating potential data manipulation or faulty sensor. Hence, option 4 is the most likely answer.&#x20;

To confirm that 4 is the right answer, we can look at the other options and cancel them out. Option 1 is incorrect, as it has been previously identified that there is no outage detected, since all the timestamps run continuously with no unusual gaps. Option 2 is also incorrect, as there are no tags labelled AIT101 in the dataset. Finally, Option 3 is incorrect, as LIT101 shows a consistent response to the changes in MV101's status.

<figure><img src="../../.gitbook/assets/image (456).png" alt=""><figcaption></figcaption></figure>
