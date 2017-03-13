
### This is a stub.


Assuming you have Redis and MySQL already installed for this example.

Next, lets create a table in MySQL:

```sql
	CREATE TABLE IF NOT EXISTS `new_mail` (
	  `mail_id` BIGINT(20) unsigned NOT NULL AUTO_INCREMENT,
	  `date` datetime NOT NULL,
	  `from` varchar(128) character set latin1 NOT NULL,
	  `to` varchar(128) character set latin1 NOT NULL,
	  `subject` varchar(255) NOT NULL,
	  `body` text NOT NULL,
	  `charset` varchar(32) character set latin1 NOT NULL,
	  `mail` longblob NOT NULL,
	  `spam_score` float NOT NULL,
	  `hash` char(32) character set latin1 NOT NULL,
	  `content_type` varchar(64) character set latin1 NOT NULL,
	  `recipient` varchar(128) character set latin1 NOT NULL,
	  `has_attach` int(11) NOT NULL,
	  `ip_addr` varchar(15) NOT NULL,
	  `return_path` VARCHAR(255) NOT NULL,
	  `is_tls` BIT(1) DEFAULT b'0' NOT NULL,
	  PRIMARY KEY  (`mail_id`),
	  KEY `to` (`to`),
	  KEY `hash` (`hash`),
	  KEY `date` (`date`)
	) ENGINE=InnoDB  DEFAULT CHARSET=utf8
```

The above table does not store the body of the email which makes it quick
to query and join, while the body of the email is fetched from Redis
for future processing. The `mail` field can contain data in case Redis is down.
Otherwise, if data is in Redis, the `mail` will be blank, and
the `body` field will contain the word 'redis'.