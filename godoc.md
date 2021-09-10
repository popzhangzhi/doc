### unsafe包使用
#### unsafe包的Pointer

```go
  type ArbitraryType int
  type Pointer *ArbitraryType 
```
unsafe包函数
```go
  func Sizeof(x ArbitraryType) uintptr
  func Offsetof(x ArbitraryType) uintptr
  func Alignof(x ArbitraryType) uintptr
```
 
