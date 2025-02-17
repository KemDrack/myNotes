```go
type Cache interface {  
    Set(k, v string)  
    Get(k string) (v string, ok bool)  
}  
  
type Partition struct {  
    data map[string]string  
    m    sync.RWMutex  
}  
  
type CacheImpl struct {  
    partitions []*Partition  
}  
  
func (c *CacheImpl) Set(k, v string) {  
    partitionIndex := c.getPartitionIndex(k)  
    partition := c.partitions[partitionIndex]  
  
    partition.m.Lock() // Эксклюзивная блокировка записи  
    defer partition.m.Unlock()  
  
    partition.data[k] = v // Запись в map  
}  
  
func (c *CacheImpl) Get(k string) (string, bool) {  
    partitionIndex := c.getPartitionIndex(k)  
    partition := c.partitions[partitionIndex]  
  
    partition.m.RLock() // Разрешает несколько одновременных чтений  
    defer partition.m.RUnlock()  
  
    v, ok := partition.data[k]  
    return v, ok  
}  
  
func (c *CacheImpl) getPartitionIndex(k string) int {  
    return hash(k) % len(c.partitions) // Примерный вариант  
}
```

- Данные хранятся в нескольких **разделах (partitions)**.
- Каждая **партиция – это `map[string]string` + `sync.RWMutex`**.
- **Чтение (`Get`) выполняется с `RLock` (разрешает конкурентные чтения)**.
- **Запись (`Set`) выполняется с `Lock` (блокирует запись, пока не освободится)**.
- Выбор партиции происходит с помощью `getPartitionIndex(k string) int`.