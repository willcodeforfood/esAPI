esAPI
=====

Makes creating new APIs fast, simple, and secure, by dynamically preparing SQL statements, based on GET/POST parameters. 

Here's how to get started:

1. Clone the esAPI repo, and put it on your PHP/MySQL webserver.
```bash
git clone git@github.com:willcodeforfood/esAPI.git
```

2. Edit the 'esDB.conf.php' file, to put in your database connection info.
```bash
vim esAPI/esDB.conf.php
```

3. Edit routes.json to create the 'routes' that make up your application!..
```bash
vim esAPI/routes.json
```

**Tail your error_log, to see debugging information**
```bash
touch esAPI/error_log
chmod 0777 esAPI/error_log
tail -f esAPI/error_log
```

Examples
===

### *SELECT* routes fetch specific fields/columns from your database.

##### HTTP Request..
> /esAPI/?action=Message_GetNew&messageId=150&convoId=3

##### Response..
```json
{"ok":true,"result":[{"Message.message_id":159,"Message.content":"Anyone in here?","Message.user_id":40,"User.name":"Gohan","Message.time":1396279063}]}
```

##### How to define this route in routes.json..
```json
"Message_GetNew" : {
	"select" : "Message",
	"fields" : [
		"Message.message_id",
		"Message.content",
		"Message.user_id",
		"User.name",
		"unix_timestamp(Message.timestamp) as Message.time"
	],
	"join": {
		"Message.user_id" : "User.user_id"
	},
	"where" : {
		"Message.convo_id": "_convoId",
		"Message.message_id": ">messageId"
	}
}
```

##### MySQL Query that esAPI prepares dynamically..
```sql
SELECT
	Message.message_id, Message.content, Message.user_id,
	User.name, unix_timestamp(Message.timestamp)
FROM Message
	LEFT JOIN (User) ON (Message.user_id=User.user_id)
WHERE Message.convo_id = ? AND Message.message_id > ?
```

##### Placeholder Values..
```php
[placeholderValues] => Array (
	[0] => 3
	[1] => 150
)
```


### *INSERT* routes create new rows in the database
	
##### HTTP Request..
	esAPI/?action=Message_Send&userId=44&convoId=3&content=Over%20Here!

##### MySQL Query that esAPI prepares dynamically..
	INSERT INTO Message ( user_id, content, convo_id ) VALUES ( ?, ?, ? )

##### Placeholder Values..
*The values submitted for 'convoId' and 'content', and the value of $_SESSION['userId'], are passed to MySQL AFTER preparing the query, to prevent SQL injection attacks.*
```php
[placeholderValues] => Array (
	[0] => 44
	[1] => Over Here!
	[2] => 3
)
```

##### How to define this route in routes.json..
```json
"Message_Send": {
	"insert": {
		"Message.user_id"  : "+userId",
		"Message.content"  : "_content",
		"Message.convo_id" : "_convoId"
	}
}
```

#### UPDATE routes modify row(s) in the database
> This section coming soon!

#### *DELETE* routes delete row(s) from the database.

##### HTTP Request..
> esAPI/?action=Message_Delete&messageId=123

##### MySQL Query that esAPI prepares dynamically..
```sql
DELETE FROM Message WHERE message_id = ?
```

##### Placeholder values..
```php	
[placeholderValues] => Array (
	[0] => 123
)
```

How to define this route in routes.json..
```json
"Message_Delete": {
	"delete": "Message",
	"where" : {
		"Message.message_id": "_messageId"
	}
}
```
