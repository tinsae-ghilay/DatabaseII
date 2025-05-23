# Redis

### for Strings
Setting a Key value pairs for Strings
```bash
# SET key value [EX seconds] [PX milliseconds] [NX | XX]
# Set a key to a string value. EX/PX for expiration. NX for "only if not exists", XX for "only if exists".
SET mykey "Hello Redis"
SET counter 10
SET another_key "another value" EX 10 # Key expires in 10 seconds
# substrings
GETRANGE mykey 4 9 # returns o Redi so starting from 1, after 4 and after 9
SETRANGE mykey 6 'World' # replaces Redis with World if we do GET mykey-> returns Hello world
```
Getting value of keys with Strings
```bash
# GET key
# Get the string value of a key.
GET mykey # Returns "Hello Redis"
GET non_existent_key # Returns (nil)
GETSET counter 20 # gets 10 and sets value to 20
GETDEL counter    # returns 20 (old value) and counter is deleted
```
and on multiple keys,  
>[!NOTE]  a trick to save objects in redis is to add structure yourselsf, as redis doe not have types or schema eg. the key would look like `user:ID:property`
```bash
MSET user:1:name "Alice" user:1:email "alice@example.com"
MGET user:1:name user:1:email user:2:name # Returns "Alice", "alice@example.com", (nil)

MSETNX config:option1 "val1" config:option2 "val2" # Returns 1 (all set if none of the keys existed, otherwise 0)
```
Commands on a key
```bash
# DEL key [key ...]
# Delete one or more keys.
DEL mykey counter

# EXISTS key [key ...]
# Check if one or more keys exist. Returns number of keys that exist.
EXISTS mykey # Returns 0 or 1
# TTL key
# Get the time to live (TTL) of a key in seconds.
# Returns -1 if key exists but has no expire, -2 if key does not exist.
TTL another_key # Returns remaining seconds from the TTL that was set to 10

# EXPIRE key seconds
# Set a timeout on key. After the timeout, the key will be automatically deleted.
EXPIRE mykey 60 # Set TTL to 60 seconds

TYPE mykey # returns  string
TYPE counter # returns string

# KEYS pattern
# Find all keys matching the given pattern. Use with caution in production on large datasets as it can block the server.
KEYS * # All keys
KEYS my* # Keys starting with 'my'
```
Although Redis doesnt have types, it can still do computation on numbers. eg.
```bash
INCR counter # returns  11
INCRBY counter 9 # returns 20 (11+9)

# other ways 
INCRBYFLOAT key # to increase float values
# we also have
DECR counter # returns 19
DECRBY counter 9 # returns 10
```
Redis commands on Bits
```bash
SETBIT users_online 10 1   # Set bit at offset 10 to 1 (user with ID 10 is online)
SETBIT users_online 20 1   # Set bit at offset 20 to 1 returns 0 becase its new(old value returned), as does the above
GETBIT users_online 10     # Returns 1
GETBIT users_online 15     # Returns 0 (by default if not set)
BITCOUNT users_online      # this now return 2, because we set 10 and 20 as active
SETBIT users_online 10 0   # returns 1(old value), but now set to 0
BITCOUNT users_online      # this now returns 1, because only bit 20 is set
```
Other bit operations
```bash
# Imagine users_A_active, users_B_active are bitmaps
BITOP AND common_active_users users_A_active users_B_active
BITCOUNT common_active_users # Count users active in both A and B
```

### for sets
```bash
# SADD key member [member ...]
# Add one or more members to a set.
SADD myset "apple" "banana" "orange" "apple" # Returns 3 (number of elements added)

# SMEMBERS key
# Get all the members in a set. Order is not guaranteed.
SMEMBERS myset # Example: ["apple", "banana", "orange"]

# SISMEMBER key member
# Check if a member is a part of a set.
SISMEMBER myset "banana" # Returns 1 (true)
SISMEMBER myset "grape" # Returns 0 (false)

# SREM key member [member ...]
# Remove one or more members from a set.
SREM myset "orange" # Returns 1 (number of elements removed)

# SCARD key
# Get the number of members in a set.
SCARD myset # Returns 2 (for "apple", "banana")
```
other set operations
```bash
# SPOP key [count]
# Remove and return one or multiple random members from a set.
SADD lottery_winners "Alice" "Bob" "Charlie" "David"
SPOP lottery_winners # Returns a random member (e.g., "Charlie"), removes it from the set.
SPOP lottery_winners 2 # Returns 2 random members (e.g., "David", "Alice"), removes them.

# SRANDMEMBER key [count]
# Return one or multiple random members from a set without removing them.
SADD attendees "User1" "User2" "User3" "User4"
SRANDMEMBER attendees # Returns a random member (e.g., "User2")
SRANDMEMBER attendees 2 # Returns 2 random members (e.g., "User4", "User1")
SRANDMEMBER attendees -2 # Returns 2 random members that may contain duplicates (useful if set has fewer than 'count' members)

# SMOVE source destination member
# Move a member from one set to another.
SADD registered_users "Alice" "Bob"
SADD active_users "Bob" "Charlie"
SMOVE registered_users active_users "Alice" # Moves "Alice" from registered_users to active_users. Returns 1.

# SINTER key [key ...]
# Returns the members of the set resulting from the intersection of all the given sets.
SADD programming_skills "Python" "Java" "C++"
SADD web_skills "HTML" "CSS" "JavaScript" "Python"
SINTER programming_skills web_skills # Returns "Python"

# SUNION key [key ...]
# Returns the members of the set resulting from the union of all the given sets.
SUNION programming_skills web_skills # Returns "Python", "Java", "C++", "HTML", "CSS", "JavaScript" (order not guaranteed)

# SDIFF key [key ...]
# Returns the members of the set resulting from the difference between the first set and all successive sets.
SDIFF programming_skills web_skills # Returns "Java", "C++" (skills in programming_skills but not in web_skills)

# SINTERSTORE destination key [key ...]
# This is like SINTER, but it stores the result in a new set at 'destination' key.
SINTERSTORE common_skills programming_skills web_skills # Stores {"Python"} in 'common_skills'

# SUNIONSTORE destination key [key ...]
# Like SUNION, but stores the result.
SUNIONSTORE all_skills programming_skills web_skills

# SDIFFSTORE destination key [key ...]
# Like SDIFF, but stores the result.
SDIFFSTORE only_programming_skills programming_skills web_skills
```

### On Hash
```bash
# HSET key field value [field value ...]
# Set the string value of a hash field.
HSET user:100 id 100 name "Alice" email "alice@example.com" age 30

# HGET key field
# Get the value of a hash field.
HGET user:100 name # Returns "Alice"

# HGETALL key
# Get all the fields and values in a hash.
HGETALL user:100 # Returns "id", "100", "name", "Alice", "email", "alice@example.com", "age", "30"

# HMGET key field [field ...]
# Get the values of all the given hash fields.
HMGET user:100 name email # Returns "Alice", "alice@example.com"

# HDEL key field [field ...]
# Delete one or more hash fields.
HDEL user:100 email # Returns 1 (number of fields deleted)

# HLEN key
# Get the number of fields in a hash.
HLEN user:100 # Returns 3 (id, name, age)
```
more advanced hash commands
```bash
# HSETNX key field value
# Set the string value of a hash field, only if the field does not exist.
HSETNX user:100 name "Bob" # Returns 0 (field 'name' already exists)
HSETNX user:100 city "New York" # Returns 1 (field 'city' did not exist)

# HINCRBY key field increment
# Increment the integer value of a hash field by the given amount.
HINCRBY user:100 age 1 # Returns 31
HINCRBY user:100 views 10 # Returns 10 (creates 'views' field if it doesn't exist)

# HINCRBYFLOAT key field increment
# Increment the float value of a hash field by the given amount.
HSET product:1 price 99.99
HINCRBYFLOAT product:1 price 5.01 # Returns "105.00"

# HKEYS key
# Get all the fields in a hash.
HKEYS user:100 # Returns "id", "name", "age", "city", "views"

# HVALS key
# Get all the values in a hash.
HVALS user:100 # Returns "100", "Alice", "31", "New York", "10"

# HSTRLEN key field
# Get the length of the value of a hash field. (Introduced in Redis 3.2)
HSTRLEN user:100 name # Returns 5 (length of "Alice")
```

### For list

```bash
 # LPUSH key value [value ...]
# Add elements to the **head** (left side) of a list.
LPUSH mylist "itemC" "itemB" "itemA"

# RPUSH key value [value ...]
# Add elements to the **tail** (right side) of a list.
RPUSH mylist "itemD" "itemE"

# LRANGE key start stop
# Get a range of elements from a list by index. `0` is the first element, `-1` is the last.
LRANGE mylist 0 -1

# LPOP key
# Remove and return the element from the **head** of the list.
LPOP mylist

# RPOP key
# Remove and return the element from the **tail** of the list.
RPOP mylist

# LLEN key
# Get the length of a list.
LLEN mylist
LINDEX mylist 1 # Returns "itemA"
LREM mylist 2 # removes the second item from list
LINSERT mylist BEFORE "itemD" "newItem" # List: ["itemB", "newItem", "itemD", "itemX", "itemA"]
```
### rename 
on any type remname key like so
```bash
RENAME oldname newname  # key oldname changes to newname
```
### publish - subscribe

Redis can also be used for message passing
```bash
# In one redis-cli:
SUBSCRIBE news_feed

# In another redis-cli:
PUBLISH news_feed "Breaking news: Redis is awesome!"
# The first client will receive the message.
```