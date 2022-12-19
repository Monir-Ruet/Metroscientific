# RocksDB Basic Usages Implementation in C++ and Documentation Important Keyword.

The main purpose of this document is to introduce with the functions and basic operations of Rocksdb.

### Block Based Table

This is the default table type that we inherited from [LevelDB](http://leveldb.googlecode.com/svn/trunk/doc/index.html), which was designed for storing data in hard disk or flash device. Data is chunked into fix sized blocks .Compression and encoding data can be eficiently within block.

###### Default Block-based table

```c
#include "rocksdb/db.h"
rocksdb::DB* db;
// Get a db with block-based table without any change.
rocksdb::DB::Open(rocksdb::Options(), "/tmp/testdb", &db);
```

###### Customized block-based table

```c
#include "rocksdb/db.h"
#include "rocksdb/table.h"

rocksdb::DB* db;
rocksdb::Options options;
// RESET DEFAULT BLOCK TABLE
options.table_factory.reset(NewBlockBasedTableFactory());
options.block_size = 4096; /* block size for the block-based table */
rocksdb::DB::Open(options, "/tmp/testdb", &db);
```

### Plain Based Table

Plain table, as its name suggests, stores data in a sequence of key/value pairs.For applications that requires low-latency in-memory database, a better alternative emerges: Plain table.

```c
#include "rocksdb/db.h"
// rocksdb/table.h includes all supported tables.
#include "rocksdb/table.h"

rocksdb::DB* db;
rocksdb::Options options;
// To enjoy the benefits provided by plain table, you have to enable
// allow_mmap_reads for plain table.
options.allow_mmap_reads = true;
// plain table will extract the prefix from a key. The prefix will be
// used for the calculating hash code, which will be used in hash-based
// index.
// Unlike Prefix_extractor is a raw pointer, please remember to delete it
// after use.
SliceTransform* prefix_extractor = new NewFixedPrefixTransform(8);
options.prefix_extractor = prefix_extractor;
options.table_factory.reset(NewPlainTableFactory(
    // plain table has optimization for fix-sized keys, which can be
    // specified via user_key_len.  Alternatively, you can pass
    // `kPlainTableVariableLength` if your keys have variable lengths.
    8,
    // For advanced users only. 
    // Bits per key for plain table's bloom filter, which helps rule out non-existent
    // keys faster. If you want to disable it, simply pass `0`.
    // Default: 10.
    10,
    // For advanced users only.
    // Hash table ratio. the desired utilization of the hash table used for prefix
    // hashing. hash_table_ratio = number of prefixes / #buckets in the hash table.
    0.75
));
rocksdb::DB::Open(options, "/tmp/testdb", &db);
...
delete prefix_extractor;
```

###### Rocksdb Options

```c
rocksdb::Options options;
options.create_if_missing = true; // CREATES DB IF THE DB MISSING
options.error_if_exists = true; // RAISES AN ERROR IF DB IS PRESENT WHEN TO OPEN A DB

options.compression = rocksdb::kNoCompression; // Default is fast enough but can be set to another.

// Write Options
rocksdb::WriteOptions write_options;
write_options.sync = true; //Defaut option is async. This is to set to synchronous.
```

###### Opening A Database

A `rocksdb` database has a name which corresponds to a file system directory. All of the contents of database are stored in this directory.

```c
  #include <cassert>
  #include "rocksdb/db.h"

  rocksdb::DB* db;
  rocksdb::Options options;
  options.create_if_missing = true;
  rocksdb::Status status = rocksdb::DB::Open(options, "/tmp/testdb", &db);
  assert(status.ok());
  ...
```

###### Rocksdb Status

This holds the status of an operation of rocksdb.Every operation returns status.

```c
   rocksdb::Status s = 0;
   // Example status
   if (!s.ok()) cerr << s.ToString() << endl;
```

###### Delete DB

```c
Status s = db->Close();
// s holds status.
 delete db;
```

#### Basic Operation

###### Read

```c
std::string value;
rocksdb::Status s = db->Get(rocksdb::ReadOptions(), key1, &value);
```

###### Write/ Update

```c
std::string value='Something';
rocksdb::Status s = db->Put(rocksdb::WriteOptions(), key2, value);
```

###### Delete

```c
rocksdb::Status s = db->Delete(rocksdb::WriteOptions(), key1);
```

###### Multiget

Multiget is the better option than looping through get operation of a key.

```c
std::vector<Slice> keys;
std::vector<PinnableSlice> values;
std::vector<Status> statuses;
std::vector<string>key={"a","b","c"};
for(auto i:key) {
   keys.emplace_back(i);
}
values.resize(keys.size());
statuses.resize(keys.size());
// cf means coloums family
// keys.data() holds the keys
// values.data() hold all the returned values
// status.data() holds all the status.
db->MultiGet(ReadOptions(), cf, keys.size(), keys.data(), values.data(), statuses.data());
```

###### Iteration

Iteration methods

```c
rocksdb::Iterator* it = db->NewIterator(rocksdb::ReadOptions()); // Iterator
it->SeekToFirst() // To the first key.
it->Valid()  // Returns true if iterator valid;
it->Next()   // Navigate to next;
it->key().ToString()  // Return iterator key
it->value().ToString()  // Return iterator value.
it->status()    // Returns status all the operation
it->status().ToString()  // Returns status message.
it->SeekToLast();  // Navigate iterator to last
it->Prev()    // Goto prev key.
it->Seek()    // Return iterator to a specific key
it->SeekForPrev('key') // If the key present then that key otherwise previous lower key
```

The following example demonstrates how to print all (key, value) pairs in a database.

```c
  rocksdb::Iterator* it = db->NewIterator(rocksdb::ReadOptions());
  for (it->SeekToFirst(); it->Valid(); it->Next()) {
    cout << it->key().ToString() << ": " << it->value().ToString() << endl;
  }
  assert(it->status().ok()); // Check for any errors found during the scan
  delete it;
```

The following variation shows how to process just the keys in the range `[start, limit)`

```c
  for (it->Seek(start);it->Valid() && it->key().ToString() < limit;
       it->Next()) {
    // Do the stuffs.
  }
  assert(it->status().ok()); // Check for any errors found during the scan
```

###### rocksdb::slice vs std::string

In previous multiget code we have used vector of slice.Rocksdb uses slice alternative to string. A string is terminated when null terminated character `\0` is found.But key value can contains null terminated character. `Slice` is a simple structure that contains a length and a pointer to an external byte array.

A Slice can be easily converted back to a C++ string:

```c
rocksdb::Slice s1 = "hello";

std::string str("world");
rocksdb::Slice s2 = str;
std::string str = s1.ToString();
assert(str == std::string("hello"));
```

###### FILE Upload in rocksdb in C/C++

```c
#include<bits/stdc++.h>
#include<rocksdb/db.h>

int main() {
  FILE* file_in=fopen("Filepath","rb");
  fseek(file_in, 0, SEEK_END);
  long int file_size = ftell(file_in);
  rewind(file_in);
  char* buffer = (char*)malloc(file_size);
  fread(buffer, file_size, 1, file_in);
  rocksdb::Slice str(buffer,file_size);
  std::string s=str.ToString();
  // cout<<s<<'\n';
  rocksdb::DB* db;
  rocksdb::Options options;
  options.create_if_missing = true;
  rocksdb::Status status = rocksdb::DB::Open(options, "../rocksdb/db", &db);
  assert(status.ok());
  rocksdb::Slice key('key');
  status=db->Put(rocksdb::WriteOptions(),key,str);
  assert(status.ok());
  std::string p;
  status=db->Get(rocksdb::ReadOptions(),key,&p);
  assert(status.ok());
  std::cout<<p.size()<<'\n';
}
```

###### Marge

Marge needs lot of implementation. Like mongodb it should marge json, object.

###### DeleteRange

```c
Slice start, end;
// set start and end
auto it = db->NewIterator(ReadOptions());

for (it->Seek(start); cmp->Compare(it->key(), end) < 0; it->Next()) {
  db->Delete(WriteOptions(), it->key());
}
```

Instead you can use 

```c
Slice start, end;
// set start and end
db->DeleteRange(WriteOptions(), start, end);
```


