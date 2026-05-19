# Catch The Criminal

## Description

{% code overflow="wrap" %}
```
Defender was recovering after a crucial mission. Then came new orders.

[Defender, through your analysis, you identified the confidential file that Alex, a former employee of Company P, leaked before leaving the company.
Intelligence indicates that he plans to deliver the leaked file to the client, and you must identify the rendezvous.
As time passes, the communication signal is fading away... Complete the mission ASAP.
Received from He_dqua_ters]

1 = Where is the rendezvous? (All lowercase, replace spaces with _ (underscore))
2 = When is the rendezvous? (YYMMDD_HHMM)

Flag format : CDDC2026{1_2}
```
{% endcode %}

Category: Forensics (100pts)

## Walkthrough

We're provided with a zip file which contains an AD1 disk image file and the auto-generated disk summary file.

<figure><img src="../../.gitbook/assets/image (412).png" alt=""><figcaption></figcaption></figure>

We can open the AD1 file in FTK Imager to analyze the disk image a lot easier

<figure><img src="../../.gitbook/assets/image (413).png" alt=""><figcaption></figcaption></figure>

<figure><img src="../../.gitbook/assets/image (414).png" alt=""><figcaption></figcaption></figure>

There isn't much to work with here. We have a couple of folders, and one user. We can investigate the user's AppData folders first, since they tend to contain valuable information.&#x20;

<figure><img src="../../.gitbook/assets/image (415).png" alt=""><figcaption></figcaption></figure>

There are a couple of interesting folders in the Roaming directory, most notably the Signal and Wire folders. Signal databases are encrypted, and since the Microsoft DPAPI-related folders were not included in the dump file, we wont be able to decrypt them to see the contents. We can instead shift our focus to Wire, and see what information we can get out of it.

<figure><img src="../../.gitbook/assets/image (416).png" alt=""><figcaption></figcaption></figure>

Wire stores information about conversations and messages in the IndexedDB folder, in IndexedDB format. [IndexedDB](https://www.cclsolutionsgroup.com/post/indexeddb-on-chromium) is a special file format that functions similarly to a database, and is used by web applications to store information offline. Since it is in a binary format, we will either need to use an existing tool or write our own script to parse it. Fortunately, there are libraries available, such as [CCL Chromium Reader](https://github.com/cclgroupltd/ccl_chromium_reader). We can use this script to extract stored data related to conversations, and write them to a file.

<details>

<summary><em>extract.py</em></summary>

{% code overflow="wrap" %}
```python
from ccl_chromium_reader import ccl_chromium_indexeddb
import json

wrapper = ccl_chromium_indexeddb.WrappedIndexDB('C:\\Users\\Admin\\Desktop\\Artifacts\\https_app.wire.com_0.indexeddb.leveldb')

# To identify which database to target, you can comment this code out
try:
    for i in range(1, 20):
        print(wrapper[i])
except Exception:
    print("Moving on")

db = wrapper[4]

# To identify which table to target, you can comment this code out
try:
    for i in range(1, 20):
        print(db[i])
except Exception as e:
    print("Moving on")

conversations = db[8]
conversation_events = db[3]
events = db[9]

for record in events.iterate_records():
    data_json = record.value
    if "conversation.asset-add" in str(data_json):
        with open("conversation_assets.txt", "a") as f:
            f.write(json.dumps(data_json, indent=2, default=str))
    elif "conversation.message" in str(data_json):
        with open("conversation_messages.txt", "a") as f:
            f.write(json.dumps(data_json, indent=2, default=str))
          
```
{% endcode %}

</details>

<figure><img src="../../.gitbook/assets/image (417).png" alt=""><figcaption></figcaption></figure>

Below are the contents of the 2 generated text files.

<details>

<summary><em>conversation_assets.txt</em></summary>

{% code overflow="wrap" %}
```json
{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "268bfe29-95f4-4a2a-8dc7-d088d86c4f6e",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 1,
  "time": "2026-04-16T11:59:20.009Z",
  "data": {
    "content_length": 6827615,
    "content_type": "image/png",
    "info": {
      "name": null,
      "height": 1536,
      "width": 2816,
      "tag": "medium"
    },
    "domain": "wire.com",
    "key": "3-5-c96e2d5b-9ac9-4306-899a-cd0d62f6522e",
    "otr_key": [
      26,
      8,
      151,
      170,
      142,
      89,
      156,
      114,
      144,
      59,
      123,
      78,
      48,
      4,
      134,
      179,
      5,
      225,
      60,
      239,
      214,
      70,
      29,
      159,
      95,
      80,
      195,
      174,
      96,
      203,
      243,
      142
    ],
    "sha256": [
      255,
      167,
      18,
      169,
      127,
      64,
      164,
      163,
      116,
      182,
      236,
      137,
      90,
      101,
      52,
      226,
      112,
      74,
      158,
      45,
      60,
      250,
      167,
      239,
      233,
      126,
      23,
      96,
      155,
      143,
      2,
      192
    ],
    "status": "uploaded",
    "token": "",
    "expects_read_confirmation": false,
    "legal_hold_status": 0
  },
  "type": "conversation.asset-add",
  "category": 128
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "268bfe29-95f4-4a2a-8dc7-d088d86c4f6e",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 2,
  "time": "2026-04-16T11:59:22.266Z",
  "data": {
    "content_length": 6827615,
    "content_type": "image/png",
    "info": {
      "name": null,
      "height": 1536,
      "width": 2816,
      "tag": "medium"
    },
    "domain": "wire.com",
    "key": "3-5-c96e2d5b-9ac9-4306-899a-cd0d62f6522e",
    "otr_key": [
      26,
      8,
      151,
      170,
      142,
      89,
      156,
      114,
      144,
      59,
      123,
      78,
      48,
      4,
      134,
      179,
      5,
      225,
      60,
      239,
      214,
      70,
      29,
      159,
      95,
      80,
      195,
      174,
      96,
      203,
      243,
      142
    ],
    "sha256": [
      255,
      167,
      18,
      169,
      127,
      64,
      164,
      163,
      116,
      182,
      236,
      137,
      90,
      101,
      52,
      226,
      112,
      74,
      158,
      45,
      60,
      250,
      167,
      239,
      233,
      126,
      23,
      96,
      155,
      143,
      2,
      192
    ],
    "status": "uploaded",
    "token": "",
    "expects_read_confirmation": false,
    "legal_hold_status": 0
  },
  "type": "conversation.asset-add",
  "category": 128,
  "primary_key": 7
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "268bfe29-95f4-4a2a-8dc7-d088d86c4f6e",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:59:22.266Z",
  "data": {
    "content_length": 6827615,
    "content_type": "image/png",
    "info": {
      "name": null,
      "height": 1536,
      "width": 2816,
      "tag": "medium"
    },
    "domain": "wire.com",
    "key": "3-5-c96e2d5b-9ac9-4306-899a-cd0d62f6522e",
    "otr_key": [
      26,
      8,
      151,
      170,
      142,
      89,
      156,
      114,
      144,
      59,
      123,
      78,
      48,
      4,
      134,
      179,
      5,
      225,
      60,
      239,
      214,
      70,
      29,
      159,
      95,
      80,
      195,
      174,
      96,
      203,
      243,
      142
    ],
    "sha256": [
      255,
      167,
      18,
      169,
      127,
      64,
      164,
      163,
      116,
      182,
      236,
      137,
      90,
      101,
      52,
      226,
      112,
      74,
      158,
      45,
      60,
      250,
      167,
      239,
      233,
      126,
      23,
      96,
      155,
      143,
      2,
      192
    ],
    "status": "uploaded",
    "token": "",
    "expects_read_confirmation": false,
    "legal_hold_status": 0
  },
  "type": "conversation.asset-add",
  "category": 128,
  "primary_key": 7
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "268bfe29-95f4-4a2a-8dc7-d088d86c4f6e",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:59:22.266Z",
  "data": {
    "content_length": 6827615,
    "content_type": "image/png",
    "info": {
      "name": null,
      "height": 1536,
      "width": 2816,
      "tag": "medium"
    },
    "domain": "wire.com",
    "key": "3-5-c96e2d5b-9ac9-4306-899a-cd0d62f6522e",
    "otr_key": [
      26,
      8,
      151,
      170,
      142,
      89,
      156,
      114,
      144,
      59,
      123,
      78,
      48,
      4,
      134,
      179,
      5,
      225,
      60,
      239,
      214,
      70,
      29,
      159,
      95,
      80,
      195,
      174,
      96,
      203,
      243,
      142
    ],
    "sha256": [
      255,
      167,
      18,
      169,
      127,
      64,
      164,
      163,
      116,
      182,
      236,
      137,
      90,
      101,
      52,
      226,
      112,
      74,
      158,
      45,
      60,
      250,
      167,
      239,
      233,
      126,
      23,
      96,
      155,
      143,
      2,
      192
    ],
    "status": "uploaded",
    "token": "",
    "expects_read_confirmation": false,
    "legal_hold_status": 0
  },
  "type": "conversation.asset-add",
  "category": 128,
  "primary_key": 7
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "268bfe29-95f4-4a2a-8dc7-d088d86c4f6e",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:59:22.266Z",
  "data": {
    "content_length": 6827615,
    "content_type": "image/png",
    "info": {
      "name": null,
      "height": 1536,
      "width": 2816,
      "tag": "medium"
    },
    "domain": "wire.com",
    "key": "3-5-c96e2d5b-9ac9-4306-899a-cd0d62f6522e",
    "otr_key": [
      26,
      8,
      151,
      170,
      142,
      89,
      156,
      114,
      144,
      59,
      123,
      78,
      48,
      4,
      134,
      179,
      5,
      225,
      60,
      239,
      214,
      70,
      29,
      159,
      95,
      80,
      195,
      174,
      96,
      203,
      243,
      142
    ],
    "sha256": [
      255,
      167,
      18,
      169,
      127,
      64,
      164,
      163,
      116,
      182,
      236,
      137,
      90,
      101,
      52,
      226,
      112,
      74,
      158,
      45,
      60,
      250,
      167,
      239,
      233,
      126,
      23,
      96,
      155,
      143,
      2,
      192
    ],
    "status": "uploaded",
    "token": "",
    "expects_read_confirmation": false,
    "legal_hold_status": 0
  },
  "type": "conversation.asset-add",
  "category": 128,
  "primary_key": 7
}
```
{% endcode %}

</details>

<details>

<summary><em>conversation_messages.txt</em></summary>

{% code overflow="wrap" %}
```json
{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "4e7526c9-aee5-4247-be0b-50cc2ae3d821",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 1,
  "time": "2026-04-16T11:52:19.217Z",
  "data": {
    "content": "I\u2019ve leaked the confidential files as discussed.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "4e7526c9-aee5-4247-be0b-50cc2ae3d821",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 2,
  "time": "2026-04-16T11:52:21.261Z",
  "data": {
    "content": "I\u2019ve leaked the confidential files as discussed.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 2
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "4e7526c9-aee5-4247-be0b-50cc2ae3d821",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:52:21.261Z",
  "data": {
    "content": "I\u2019ve leaked the confidential files as discussed.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 2
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "4e7526c9-aee5-4247-be0b-50cc2ae3d821",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:52:21.261Z",
  "data": {
    "content": "I\u2019ve leaked the confidential files as discussed.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 2
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "4e7526c9-aee5-4247-be0b-50cc2ae3d821",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:52:21.261Z",
  "data": {
    "content": "I\u2019ve leaked the confidential files as discussed.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 2
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "fda3ed51-9e55-4673-bd33-bf339f601fe0",
  "from_client_id": "<Undefined>",
  "id": "53023c94-ed88-4a49-b66a-81b6a20db683",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": {
    "domain": "wire.com",
    "id": "fda3ed51-9e55-4673-bd33-bf339f601fe0"
  },
  "status": "<Undefined>",
  "time": "2026-04-16T11:53:44.138Z",
  "data": {
    "content": "Well done. I knew you were the right choice. Now for the final step. To keep things secure, it\u2019s best we meet in person to hand over the storage device. Have you picked a spot?",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "26d37cc4-2b9b-49ea-ac96-57ca80feb701",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 1,
  "time": "2026-04-16T11:55:38.118Z",
  "data": {
    "content": "I wanted to avoid any radar from the security team, so I consulted an AI to find a high-traffic location where I could easily blend in. It gave me a few recommendations, and I\u2019ve made my choice.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "26d37cc4-2b9b-49ea-ac96-57ca80feb701",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 2,
  "time": "2026-04-16T11:55:40.116Z",
  "data": {
    "content": "I wanted to avoid any radar from the security team, so I consulted an AI to find a high-traffic location where I could easily blend in. It gave me a few recommendations, and I\u2019ve made my choice.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 4
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "26d37cc4-2b9b-49ea-ac96-57ca80feb701",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:55:40.116Z",
  "data": {
    "content": "I wanted to avoid any radar from the security team, so I consulted an AI to find a high-traffic location where I could easily blend in. It gave me a few recommendations, and I\u2019ve made my choice.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 4
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "26d37cc4-2b9b-49ea-ac96-57ca80feb701",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:55:40.116Z",
  "data": {
    "content": "I wanted to avoid any radar from the security team, so I consulted an AI to find a high-traffic location where I could easily blend in. It gave me a few recommendations, and I\u2019ve made my choice.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 4
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "26d37cc4-2b9b-49ea-ac96-57ca80feb701",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:55:40.116Z",
  "data": {
    "content": "I wanted to avoid any radar from the security team, so I consulted an AI to find a high-traffic location where I could easily blend in. It gave me a few recommendations, and I\u2019ve made my choice.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 4
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "fda3ed51-9e55-4673-bd33-bf339f601fe0",
  "from_client_id": "<Undefined>",
  "id": "2dab4683-4c73-4400-abce-eb927768b273",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": {
    "domain": "wire.com",
    "id": "fda3ed51-9e55-4673-bd33-bf339f601fe0"
  },
  "status": "<Undefined>",
  "time": "2026-04-16T11:57:04.256Z",
  "data": {
    "content": "Using AI for that? You\u2019re even sharper than I thought. Send me the details later. What time are we talking? If it\u2019s too late, it might raise some red flags.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "db2596fd-54c9-4ae5-8257-8498e3b5277d",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 1,
  "time": "2026-04-16T11:58:16.388Z",
  "data": {
    "content": "This Friday is my last day. I think it\u2019s best to meet right after I finish packing my things and clear out of the office for good.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "db2596fd-54c9-4ae5-8257-8498e3b5277d",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 2,
  "time": "2026-04-16T11:58:19.646Z",
  "data": {
    "content": "This Friday is my last day. I think it\u2019s best to meet right after I finish packing my things and clear out of the office for good.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 6
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "db2596fd-54c9-4ae5-8257-8498e3b5277d",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:58:19.646Z",
  "data": {
    "content": "This Friday is my last day. I think it\u2019s best to meet right after I finish packing my things and clear out of the office for good.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 6
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "db2596fd-54c9-4ae5-8257-8498e3b5277d",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:58:19.646Z",
  "data": {
    "content": "This Friday is my last day. I think it\u2019s best to meet right after I finish packing my things and clear out of the office for good.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 6
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "db2596fd-54c9-4ae5-8257-8498e3b5277d",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T11:58:19.646Z",
  "data": {
    "content": "This Friday is my last day. I think it\u2019s best to meet right after I finish packing my things and clear out of the office for good.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 6
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "fda3ed51-9e55-4673-bd33-bf339f601fe0",
  "from_client_id": "<Undefined>",
  "id": "174ddd43-b5b1-4e46-8515-18adcd6691d1",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": {
    "domain": "wire.com",
    "id": "fda3ed51-9e55-4673-bd33-bf339f601fe0"
  },
  "status": "<Undefined>",
  "time": "2026-04-16T12:00:02.849Z",
  "data": {
    "content": "See you then. The remaining 9 BTC will be transferred to your wallet the moment I\u2019ve verified the files.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "5ccd90b4-5cfd-480d-91a3-61361231dba6",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 1,
  "time": "2026-04-16T12:00:11.720Z",
  "data": {
    "content": "Understood. I\u2019ll see you then.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "5ccd90b4-5cfd-480d-91a3-61361231dba6",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 2,
  "time": "2026-04-16T12:00:13.735Z",
  "data": {
    "content": "Understood. I\u2019ll see you then.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 9
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "5ccd90b4-5cfd-480d-91a3-61361231dba6",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T12:00:13.735Z",
  "data": {
    "content": "Understood. I\u2019ll see you then.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 9
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "5ccd90b4-5cfd-480d-91a3-61361231dba6",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T12:00:13.735Z",
  "data": {
    "content": "Understood. I\u2019ll see you then.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 9
}{
  "conversation": "33689219-91c4-5e49-b2fa-aa7f70ef2442",
  "from": "68470b59-fe93-40a7-9557-da09a3e24686",
  "from_client_id": "<Undefined>",
  "id": "5ccd90b4-5cfd-480d-91a3-61361231dba6",
  "qualified_conversation": {
    "domain": "wire.com",
    "id": "33689219-91c4-5e49-b2fa-aa7f70ef2442"
  },
  "qualified_from": "<Undefined>",
  "status": 3,
  "time": "2026-04-16T12:00:13.735Z",
  "data": {
    "content": "Understood. I\u2019ll see you then.",
    "mentions": [],
    "previews": [],
    "expects_read_confirmation": false,
    "legal_hold_status": 1
  },
  "type": "conversation.message-add",
  "category": 16,
  "primary_key": 9
}
```
{% endcode %}

</details>

<figure><img src="../../.gitbook/assets/image (418).png" alt=""><figcaption></figcaption></figure>

Useful information for sure, but doesn't answer the two questions posed to us. Notice that there are some image data that was grepped as well, which could indicate that an attachment was sent in the conversation. This attachment may include the location and meetup time.&#x20;

In Wire, attachments that were sent are encrypted and saved to the Cache\_Data folder, as such

<figure><img src="../../.gitbook/assets/image (419).png" alt=""><figcaption></figcaption></figure>

By correlating the content length information in the previous JSON output, we can speculate that files f\_00000f thru f\_000013 are the attachment that was shared. Save one of these files for decryption purposes. Earlier, we also noticed a field in the attachments JSON, labelled "otr\_key" and "sha256". The SHA256 hash is the hash value of the encrypted file, used to verify integrity. The binary file itself also contains the IV used to encrypt the original attachment, as well as the checksum SHA256 hash. With this information, we can write a decryption script to recover the attachment

Asking an LLM to generate a decryption script gave me the following code

<details>

<summary><em>image_decrypt.py</em></summary>

{% code overflow="wrap" %}
```python
from Crypto.Cipher import AES
from Crypto.Util.Padding import unpad
import base64
import hashlib
import json
import sys

def decrypt_wire_asset(encrypted_bytes, otr_key_b64, sha256_b64, output_path):
    """
    Decrypt a Wire encrypted asset file.
    
    encrypted_bytes : raw bytes of the encrypted file from cache/disk
    otr_key_b64     : base64-encoded AES key from IndexedDB event record
    sha256_b64      : base64-encoded expected SHA256 hash for verification
    output_path     : where to save the decrypted image
    """
    
    print(otr_key_b64)
    print(sha256_b64)

    # Decode key and hash
    otr_key = base64.b64decode(otr_key_b64)   # 32 bytes (AES-256)
    expected_sha256 = base64.b64decode(sha256_b64)

    # Wire asset format:
    # [ IV (16 bytes) ][ Ciphertext ][ SHA256 of ciphertext (32 bytes) ]
    iv          = encrypted_bytes[:16]
    ciphertext  = encrypted_bytes[16:-32]
    file_hash   = encrypted_bytes[-32:]

    # Step 1: verify integrity
    actual_sha256 = hashlib.sha256(encrypted_bytes).digest()
    if actual_sha256 != expected_sha256:
        print("[!] SHA256 mismatch — file may be corrupt or tampered")
    else:
        print("[+] SHA256 verified OK")

    # Step 2: decrypt
    cipher = AES.new(otr_key, AES.MODE_CBC, iv)
    decrypted = unpad(cipher.decrypt(encrypted_bytes), AES.block_size)

    # Step 3: save
    
    png_header = b'\x89PNG\r\n\x1a\n'
    offset = decrypted.find(png_header)
    if offset == -1:
        print("[!] Could not find PNG header, decryption may have failed or original file is not PNG format")
        return 0
    else:
        print(f"[+] PNG header found at offset {offset}. Removing leading bytes...")
        png_data = decrypted[offset:]

        with open(output_path, "wb") as f:
            f.write(png_data)
        print(f"[+] Saved to {output_path}")
        return decrypted


# Example usage:
with open("encrypted.dat", "rb") as f:
    encrypted = f.read()

with open("assets_add_record.txt", "r") as f:
    assets = json.load(f)

decrypt_wire_asset(
    encrypted_bytes = encrypted,
    otr_key_b64     = base64.b64encode(bytes(assets["data"]["otr_key"])).decode(),
    sha256_b64      = base64.b64encode(bytes(assets["data"]["sha256"])).decode(),
    output_path     = "recovered_image.png"
)

```
{% endcode %}

</details>

<figure><img src="../../.gitbook/assets/image (420).png" alt=""><figcaption></figcaption></figure>

This is one half of the flag. Investigating the image itself further, we can see that it is AI-generated, based on the watermark on the bottom right side, as well as the C2PA metadata. However, there isn't any information hinting at the location.&#x20;

Returning to investigate the disk image, we can see a couple of packages in the Local AppData folder. One of the interesting folders is a ChatGPT Desktop folder, which contains a nested LocalCache folder. Inside this folder is another IndexedDB cache file, which contains the results from a user query in plaintext.

<figure><img src="../../.gitbook/assets/image (421).png" alt=""><figcaption></figcaption></figure>

There are a couple of options in the list, and we can try them all. The answer for this half of the flag is Times Square

### Flag

Unfortunately I couldn't take a screenshot of the flag submission, but this is the correct flag for this challenge.

`CDDC2026{times_square_260420_1930}`
