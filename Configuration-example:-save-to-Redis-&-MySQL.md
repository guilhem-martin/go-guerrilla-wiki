
### This is a stub.


Assuming you have Redis and MySQL already installed for this example.

Next, let's create a table in MySQL:

```sql
CREATE TABLE IF NOT EXISTS `new` (
  `mail_id` BIGINT(20) unsigned NOT NULL AUTO_INCREMENT,
  `message_id` varchar(256) character set latin1 NOT NULL COMMENT 'value of [Message-ID] from headers',
  `date` datetime NOT NULL,
  `from` varchar(256) character set latin1 NOT NULL COMMENT 'value of [From] from headers or return_path (MAIL FROM) if no header present',
  `to` varchar(256) character set latin1 NOT NULL COMMENT 'value of [To] from headers or recipient (RCPT TO) if no header present',
  `reply_to` varchar(256) NULL COMMENT 'value of [Reply-To] from headers if present',
  `sender` varchar(256) NULL COMMENT 'value of [Sender] from headers of present',
  `subject` varchar(255) NOT NULL,
  `body` varchar(16) NOT NULL,
  `mail` longblob NOT NULL,
  `spam_score` float NOT NULL,
  `hash` char(32) character set latin1 NOT NULL,
  `content_type` varchar(64) character set latin1 NOT NULL,
  `recipient` varchar(255) character set latin1 NOT NULL COMMENT 'set by the RCPT TO command.',
  `has_attach` int(11) NOT NULL,
  `ip_addr` varbinary(16) NOT NULL,
  `return_path` VARCHAR(255) NOT NULL COMMENT 'set by the MAIL FROM command. Can be empty to indicate a bounce, i.e <>',
  `is_tls` BIT(1) DEFAULT b'0' NOT NULL,
  PRIMARY KEY  (`mail_id`),
  KEY `to` (`to`),
  KEY `hash` (`hash`),
  KEY `date` (`date`)
) ENGINE=InnoDB  DEFAULT CHARSET=utf8;
```

#### mail_id and message_id

`mail_id` is the primary auto-increment key, `message_id` is a value that was parsed from the email headers.
If not present in the headers, it will use `<hash>.<local-part-to>@<primary_mail_host>` - where <primary_mail_host>
comes from the config, <hash> is the hash of the body, <local-part-to> is the local part of the recipient address.


#### mail and body columns

The above table does not store the body of the email which makes it quick
to query and join, while the body of the email is fetched from Redis
for future processing. The `mail` field can contain data in case Redis is down.
Otherwise, if data is in Redis, the `mail` will be blank, and
the `body` field will contain the word `redis`.

#### hash column

To get the email body from Redis, use the value from the `hash` column for the redis key.

#### ip_address column

The ip_address is packed into a 16 byte binary. 
For details see this [stack overflow](http://stackoverflow.com/questions/5133580/which-mysql-datatype-to-use-for-an-ip-address) answer.
Eg, In MySQL, one would use `select INET6_NTOA(ip_addr) from new;` and in PHP one would use `inet_ntop` function to get the human readable format.

#### from, to, recipient, return_path and sender columns

These columns are always the extracted email address, and do not include sender name.
See individual comments embedded in the table definition SQL for details.

#### Indexes

By there's an index on `to` `hash` and `date`. Adding indexes may reduce insert performance, it's up to whenever you want to keep these or not, or add more. 