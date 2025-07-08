**Time Series Data**

This NIP defines how time series data for statistical purposes should be stored using event kind xxxxx.

Note: Nostr events cannot replace centralized database systems such as SQL in terms of performance and availability. Therefore, storing statistical data on Nostr relays is meant to offer users ownership of their data and encourage interoperability throughout different apps that build on Nostr.

Time Series Content

/\* With daily datapoints (culminated data or measured data points) \*/

{

	startDate: 1746427299000,

	endDate: 1746427319000,

	interval: 'day',

	dataType: 'steps walked',

	datasetName: 'My steps health data',

	source: 'Satoshis iPhone 21 Plus',

	applicationSource: 'Sound HSA',

	data: \[1233,4535,984,234,678,...\],

	info: 'measured with pedometer on mobile phone'

}

/\* with Epoch UNIX timestamps in milliseconds \*/ 

{

	startDate: 1746427299000,

	endDate: 1746427319000,

	interval: 'with-timestamps',

	dataType: 'water temperature in C',

	datasetName: 'My pool temperatures',

  source: 'Sensor01',

  metric: 'Celsius',

	data: \[\[1746427299000, 12\],\[1746427299000, 13\],\[1746427299000, 13.3\],\[1746427299000, 12.8\],\[1746427299000, 11.9\],...\],

}

...

Interval options should be selected from the following list

\['second', 'minute', 'hour', 'day', 'week', 'month', 'year', 'with-timestamps'\]

The dataType value can be freely chosen. We encourage apps to tackle interoperability by merging similar dataType\[\] via white listings or using AI to match dataType\[\] without exposing the user data. Same applies for metrics.

Client applications should choose wisely how and if they culminate data. For instance, steps measured with a pedometer could be culminated into steps per day and stored into a single event for each day. Synching to or from Nostr should be implemented carefully. Data compression from multiple events into single events by setting the interval could be an option to improve loading times. Data queries should be performed on local data.

**Event Structure**

**Required Fields**

- kind: 123456
- content: Encrypted JSON payload (NIP-44)
- tags:
  - \["dataset_id", "<data set unique identifier>" | "<sha256 hash from datasetName>"\] (e.g. "steps walked in sound hsa app on myiPhone" or a hash)
  - optional:\["start", "<unix timestamp in seconds>"\] (for querying, but would give away some privacy, therefore it is optional)
  - optional: \["end", "<unix timestamp in seconds>"\] (for querying, but would give away some privacy, therefore it is optional)

**Encryption**

Since this NIP addresses user data, encryption should be the default for time series events. For public available data, apps may choose to publish events without encryption but should consider performance issues of time series data populated through events. That said, public (unencrypted) data on Nostr offers transparency in regards to the originality of the data itself.

val owner = nostr.generateKeyPair()

/\* 

 \* use owner as viewer if data should only be accessible for the owner 

\*/

val viewer = nostr.generateKeyPair() 

{

  "id": "<32-bytes lowercase hex-encoded SHA256 of the serialized event data>",

  "kind": xxxxx,

  "pubkey": "<32-bytes lowercase hex-encoded public key of the event creator>",

  "created_at": 1725893321,

  "pubkey": sign.publicKey

  "tags": \[

	  \["date_range", "1725893321000", "1725899321000"\],

	  \["d", "<sha256-hashed datasetName>",

	  \["p", viewer.pubKey\]

  \],

  "content": nip44Encrypt("<kindxxx-event>", owner.privateKey, viewer.publicKey),

  "sig": signWith(owner.privateKey)

}

**Privacy considerations**

Some data might be too sensitive to even share its type or metric in unencrypted tags. We propose to run a hash function like SHA256 to create a unique id for the whole dataset (which may hold many events).

let hashed_id = sha256(content.datasetName)

[event.tags.data](http://event.tags.data)\_set_id = hashed_id

**Sharing data**

The purpose of this NIP is to offer users ownership for their data and to serve interoperability for different applications that address common data sets. When sharing with others, data should be culminated (compressed) into single events. Access rights are provided by using a viewer pubKey different from the owner.

**Event size limits**

Apps and relays should allow standard event sizes up to 128K. Therefore, apps must calculate and check the actual size of the encrypted and signed event before publishing it on a relay. Chunking data into multiple events is the job of applications providing NIP-XXX functionalities.