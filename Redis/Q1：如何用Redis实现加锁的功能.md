# 如何用 Redis 实现分布式锁

## 背景

比方说一个后端服务部署在几十台机器上，这时候如何用 Redis 实现一个分布式锁保证这些机器安全地竞争资源？

## 实现

```go
/*
 基于 Redis 实现的锁操作
*/

const (
 RedisLockKeyPrefix = "DUSEPK_REDIS_LOCK_"
 RedisOperatorSuccess = 1
)

type RedisLock struct {
 CodisAccess *db.CodisAccess
}

var RedisLockPtr *RedisLock

func NewRedisLock(ca *db.CodisAccess) *RedisLock {
 var rl RedisLock
 rl.CodisAccess = ca
 return &rl
}

/*
功能: 加锁
@param1: key 锁的唯一标识
@param2: token 用于解锁的 token
@param3: expireTimeSec 过期时间(秒)
@return1: isLockSuccess 是否加锁成功
@return2: err 不为空时 isLockSucc 不可信
*/
func (rl *RedisLock) Lock(sid string, key string, token string, expireTimeSec int) (isLockSuccess bool, err error) {
 isLockSuccess = false

 if rl.CodisAccess == nil {
  err = derrors.New(global.ERRNO_REDIS_HAVE_NO_CODISACCESS, "RedisLock has no CodisAccess available")
  return
 }

 // 仅当 Key 不存在时才能设置成功, 成功返回"OK"
 // SET KEY VALUE EX expireTimeSec NX
 redisCmd := "SET"
 redisKey := RedisLockKeyPrefix + key
 redisInputParams := []interface{}{token, "EX", expireTimeSec, "NX"}

 var redisResult string
 redisResult, err = redis.String(rl.CodisAccess.SetData(sid, global.CODIS_GROUP_PK, redisCmd, redisKey, redisInputParams))
 if err == nil && redisResult == "OK" {
  isLockSuccess = true
 }

 return
}

/*
功能: 解锁
@param1: key 锁的唯一标识
@param2: token 用于解锁的 token
@return1: isUnLockSuccess 是否解锁失败
@return2: err 不为 nil 时 isUnLockSuccess 不可信
*/
func (rl *RedisLock) UnLock(sid string, key string, token string) (isUnLockSuccess bool, err error) {
 isUnLockSuccess = false

 if rl.CodisAccess == nil {
  err = derrors.New(global.ERRNO_REDIS_HAVE_NO_CODISACCESS, "RedisLock has no CodisAccess available")
  return
 }

 redisCmd := "GET"
 redisKey := RedisLockKeyPrefix + key
 var redisResult string
 redisResult, err = redis.String(rl.CodisAccess.GetData(sid, global.CODIS_GROUP_PK, redisCmd, redisKey))

 // key 不存在或者 key 已过期: 直接返回解锁成功
 if err == redis.ErrNil {
  isUnLockSuccess = true
  err = nil
  return
 }

 if err == nil && redisResult == token {
  redisCmd = "DEL"
  var nRet int
  nRet, err = redis.Int(rl.CodisAccess.SetData(sid, global.CODIS_GROUP_PK, redisCmd, redisKey, nil))
  if nRet == RedisOperatorSuccess {
   isUnLockSuccess = true
   return
  }
 }

 return
}
```
