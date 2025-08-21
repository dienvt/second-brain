## Best practict

[https://instagram-engineering.com/storing-hundreds-of-millions-of-simple-key-value-pairs-in-redis-1091ae80f74c](https://instagram-engineering.com/storing-hundreds-of-millions-of-simple-key-value-pairs-in-redis-1091ae80f74c)

## Cheat sheet

### HGETALL

- By default, Redis stores the Hash object as a zipped list when the hash has less than 512 entries and when each element’s size is smaller than 64 bytes.
- If either limit is exceeded, Redis converts the list to a hashtable, and this is irreversible. That is, Redis won’t convert the hashtable back to a list again, even if the entries/size falls below the limit.

### RLIST

RList<> không được quá 1,000,000 nếu không sẽ bị lỗi redis timeout.

### RBUCKET

RBucket<> sử dụng codec new KryoCodecWithDefaultSerializer(); để compatable với data cũ

### EVAL: thực thi lua scrip.

- param EVAL `<`script`>` `<`num key`>` KEY[] ARGV[]
- Ex: eval "return {KEYS[1],KEYS[2],ARGV[1],ARGV[2]}" 2 key1 key2 first second  
    result: - key1  
    - key2  
    - first  
    - second  
    

### Remove key pattern

Use SCAN, Never Use KEYS

```Plain
redis-cli --scan --pattern "pattern:*" |xargs -L 1 redis-cli del
```

# **Serialize**

java doc of CompatibleFieldSerializer

com.esotericsoftware.kryo.serializers public class CompatibleFieldSerializer extends FieldSerializer

Serializes objects using direct field assignment, providing both forward and backward compatibility. This means fields can be added or removed without invalidating previously serialized bytes. Changing the type of a field is not supported. Like FieldSerializer, it can serialize most classes without needing annotations. The forward and backward compatibility comes at a cost: the first time the class is encountered in the serialized bytes, a simple schema is written containing the field name strings. Also, during serialization and deserialization buffers are allocated to perform chunked encoding. This is what enables CompatibleFieldSerializer to skip bytes for fields it does not know about.

Removing fields when references are enabled can cause compatibility issues. See here .

Note that the field data is identified by name. The situation where a super class has a field with the same name as a subclass must be avoided.

1. sử dụng: initRedisClient(config, new KryoCodec());

- thêm mới field trong cache object, read lại -> bị EXCEPTION (io.netty.handler.codec.DecoderException)
- thêm mới field trong cache object, delete cache -> OK
- thêm mới object (composite object) trong cache object, read lại or delete cache -> bị EXCEPTION (same as above)
- thêm mới field trong composite object, read lại or delete cache -> bị EXCEPTION (same as above)
- delete field cũ trong cache object, read lại -> deserialize ra dữ liệu KHÔNG ĐÚNG
    
    (thường không delete field cũ, vẫn giữ field cũ để compatibility với version cũ)
    

1. sử dụng: initRedisClient(config, new KryoCodecWithDefaultSerializer());

- thêm mới field trong cache object, read lại -> No exception, data is correct.
- thêm mới field trong cache object, delete cache -> OK
- thêm mới object (composite object) trong cache object, read lại or delete cache -> No exception, data is correct.
- thêm mới field trong composite object, read lại or delete cache -> No exception, data is correct.
- thêm mới ENUM field trong cache object, read lại, đảo thứ tục enum rồi read lại -> No exception, data is correct.
- delete field cũ trong cache object, read lại -> No exception, the other data is correct
    
    (thường không delete field cũ, vẫn giữ field cũ để compatibility với version cũ)
    

TÓM LẠI:

Nên chuyển qua sử dụng KryoCodecWithDefaultSerializer(), vì:

- khi thêm field mới (việc này thường xảy ra theo yêu cầu business), không cần phải xoá cache cũ
- đọc lại cache cũ không bị exception.