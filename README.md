# mysql-proxy
mysql代理服务设计指南

## 关键难点
1. 代理不知道客户端的密码, 需要使用AuthSwitchRequest来切换到真实mysql服务的加密算法和盐码
2. 执行带?的sql, 会生成2个command命令,第一步COM_STMT_PREPARE发送sql,第二步COM_STMT_EXECUTE发送参数, 这样代理要识别COM_STMT_PREPARE中的sql是否需要路由,等到COM_STMT_EXECUTE后, 解析sql后将逻辑表改写成物理表sql后, 将多个查询结果聚合后重新数据包返回给client

## 核心思路
1. mysql客户端和代理服务端建立Tcp连接
2. 代理服务端发送给mysql客户端, 初始化握手数据包: 下发加密算法、盐码 
3. mysql客户端读取初始化握手, 得到加密算法、盐码:  用盐码对密码加密: 发送给代理服务端用户名、database、加密密码
4. 代理服务端得到: 用户名、database、加密密码;  然后连接mysql服务, 得到新加密算法、盐码
5. 代理服务端发送给mysql客户端切换加密算法和新盐, (mysql给的新加密算法、新的盐码)
6. mysql客户端根据新的加密算法和盐码, 生成加密密钥发给代理服务端
7. 代理服务端得到新的加密密钥, (代理服务端给mysql服务发送认证数据: 用户名、database、加密密码), 得到mysql的响应后决定后续给mysql客户端成功与失败
8. mysql客户端发送command给代理服务端,  代理服务端将command命令发送给mysql服务, 读取到响应数据后,回写数据给mysql客户端
9. 逻辑表查询需要对相应结果解码后, 聚合数据重新编码







