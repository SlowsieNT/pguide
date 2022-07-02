# pguide
Welcome.


## Realtime messaging with MySQL, and bit of php
This will have least requirements.

### Database structure

Table `channel` has following fields:
- `id` SERIAL, not primary
- `name` VARCHAR(255)



Optional fields:
- `logo_blob` BLOB
- `logo_url` VARCHAR(2048)

Table `messages` has following fields:
- `id` SERIAL, not primary
- `uid` BIGINT, attribute: `UNSIGNED`
- `message` TEXT, or VARCHAR
- `channel_id` BIGINT UNIQUE, attribute: `UNSIGNED`

Table `last_message`<br>
This table does not spam with `INSERT`, it's done only once,<br>
Meaning: every time new message is sent, `UPDATE` must be called immediately.<br>
<br>
Has following fields:
- `last_id` BIGINT, attribute: `UNSIGNED`
- `channel_id` BIGINT UNIQUE, attribute: `UNSIGNED`
- `uid` BIGINT, attribute: `UNSIGNED`
The `channel_id` will crash `INSERT` if duplicate id is found.
Thus `UPDATE` must be called instead.

<br><br>
Assuming that php will be used, you would want to use `usleep(256e3)` for the MINIMUM delay between each `fetchall`.<br>
However if you are sure your server is capable of higher stress, use `usleep(128e3)` instead.<br>
For free hosting, about `usleep(500e3)` is recommended.<br>
<br><br>

This is how you do realtime fetch:
```php
// dummy functions, let's assume you wrote actual code
function fetchAll($last_msg){
  $last_id = $last_msg->last_id;
  $q = "SELECT * from messages where id>$last_id";
  //...
}
function get_last_message($channel_id){
  $q = "SELECT * from last_message where channel_id='$channel_id'";
  //...
}

$channel_id = 1;
$maxticks = 32;
$last_msg = get_last_message($channel_id);
// last_msg contains last_id which is later used by fetchAll to fecth messages AFTER that id
while ($maxticks--) {
  $curr_msg = get_last_message($channel_id);
  if ($last_msg != $curr_msg) {
    $last_msg = $curr_msg;
    break;
  }
  usleep(500e3);
}
echo fetchAll($last_msg);

```
