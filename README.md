# ssh-copy-id
无密码登录备份，详细文档参考wiki


#sync map
type Map struct {
	mu Mutex //互斥锁，主要用来锁dirty这个map

	read atomic.Value // readOnly
	
	dirty map[interface{}]*entry

	misses int //记录了穿透read 直接读 dirty map的次数，当这个次数超过了len(dirty)时会有升级
}

实现的基本思想：
1、读写分离
2、读请求尽可能的落到原子结构(atomic.Value) 的read上去,体现在两点
    a.读请求先到read在到dirty(这里有个加双锁的操作需要注意)
    b.只要读请求没有在read中找到且 dirty存在read中没有的数据(amended为true)，则misses++
      amended bool                   // true if the dirty map contains some key not in m.
