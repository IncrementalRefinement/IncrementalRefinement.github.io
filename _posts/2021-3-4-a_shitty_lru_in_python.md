被问了LRU怎么写，先想用clock实现，但被怼实在是太蠢，面现场想出来用链表和hash实现但是接口不熟悉没写出来，事后补一下，这个写法我觉得也很蠢，找点文章看看还有哪些lru写法

in my shitty python code:

```python
class lru:
    def __intit__(self, capacity):
        self.capacity = capacity
        self.slot_list = []
        self.key_index_map = {}
        
    def get(self, key):
        index = self.key_index_map.get(key)
        if index == None:
            return None
        else:
            return self.slot_list[index] 
    
    def insert(self, key, value):
        if self.get(key) != None:
            index = self.key_index_map(key)
            temp = self.slot_list.pop(index)
            temp[2] = value
            self.slot_list.append(temp)
        else:
            if len(self.slot_lsit) < self.capacity:
                self.slot_list.append((key, value))
                self.key_index_map[key] = len(self.slot_list)
            else:
                index = 0
                pop_key = self.slot_list[index][0]
                del self.key_index_map[pop_key]
                self.slot_list.pop(0)
                self.slot_list.append((key, value))
                self.key_index_map[key] = len(self.slot_list)
                
```
