---
title: ThinkPHP5.1 实现 ThinkPHP3.1 的 SessionDB 类
date: 2021-11-14 19:14:59
tags: [PHP, ThinkPHP]
categories: PHP
---

`ThinkPHP5.1` 实现 `ThinkPHP3.1` 的 `SessionDB` 类，完成 `Session` 写入到 `MySQL` 的操作

# 第一种实现

## 建表语句

```sql
CREATE TABLE think_session (
    session_id varchar(255) NOT NULL,
    session_expire int(11) UNSIGNED NOT NULL,
    session_data blob,
    UNIQUE KEY `session_id` (`session_id`)
);
```

## 建立数据库驱动扩展类 /extend/driver/session/Mysql.php

```php
<?php
 
namespace driver\session;
 
use SessionHandler;
use think\Db;
use think\Config;
use think\Exception;
 
/**
 * Class Mysql
 * session 数据库驱动
 * @package driver\session
 */
class Mysql extends SessionHandler
{
    protected $handler = null;
    protected $table_name = null;
    protected $config = [
        'session_expire' => 3600,           // Session有效期 单位：秒
        'session_prefix' => 'think_',       // Session前缀
        'table_name'     => 'session',      // 表名（不包含表前缀）
    ];
    protected $database = [
        'type'     => 'mysql',        // 数据库类型
        'hostname' => '127.0.0.1',    // 服务器地址
        'database' => 'my_test',       // 数据库名
        'username' => 'root',         // 用户名
        'password' => '',             // 密码
        'hostport' => '3306',         // 端口
        'prefix'   => '',             // 表前缀
        'charset'  => 'utf8',         // 数据库编码
        'debug'    => true,           // 数据库调试模式
    ];
 
    /**
     * Mysql constructor.
     * @param array $config
     * @throws Exception
     */
    public function __construct($config = [])
    {
        // 获取数据库配置
        if (isset($config['database']) && !empty($config['database'])) {
            if (is_array($config['database'])) {
                $database = $config['database'];
            } elseif (is_string($config['database'])) {
                $database = Config::get($config['database']);
            } else {
                throw new Exception('session error:database');
            }
            unset($config['database']);
        } else {
            // 使用默认的数据库配置
            $database = Config::get('database');
        }
 
        $this->config = array_merge($this->config, $config);
        $this->database = array_merge($this->database, $database);
    }
 
    /**
     * 打开Session
     * @access public
     * @param string $save_path
     * @param mixed $session_name
     * @return bool
     * @throws Exception
     */
    public function open($save_path, $session_name)
    {
        // 判断数据库配置是否可用
        if (empty($this->database)) {
            throw new Exception('session error:database empty');
        }
        $this->handler = Db::connect($this->database);
        $this->table_name = $this->database['prefix'] . $this->config['table_name'];
        return true;
    }
 
    /**
     * 关闭Session
     * @access public
     */
    public function close()
    {
        $this->gc(ini_get('session.gc_maxlifetime'));
        $this->handler = null;
        return true;
    }
 
    /**
     * 读取Session
     * @access public
     * @param string $session_id
     * @return bool|string
     */
    public function read($session_id)
    {
        $where = [
            'session_id'     => $this->config['session_prefix'] . $session_id,
            'session_expire' => time()
        ];
        $sql = 'SELECT session_data FROM ' . $this->table_name . ' WHERE session_id = :session_id AND session_expire > :session_expire';
        $result = $this->handler->query($sql, $where);
        if (!empty($result)) {
            return $result[0]['session_data'];
        }
        return '';
    }
 
    /**
     * 写入Session
     * @access public
     * @param string $session_id
     * @param String $session_data
     * @return bool
     */
    public function write($session_id, $session_data)
    {
        $params = [
            'session_id'     => $this->config['session_prefix'] . $session_id,
            'session_expire' => $this->config['session_expire'] + time(),
            'session_data'   => $session_data
        ];
        $sql = 'REPLACE INTO ' . $this->table_name . ' (session_id, session_expire, session_data) VALUES (:session_id, :session_expire, :session_data)';
        $result = $this->handler->execute($sql, $params);
        return $result ? true : false;
    }
 
    /**
     * 删除Session
     * @access public
     * @param string $session_id
     * @return bool|void
     */
    public function destroy($session_id)
    {
        $where = [
            'session_id' => $this->config['session_prefix'] . $session_id
        ];
        $sql = 'DELETE FROM ' . $this->table_name . ' WHERE session_id = :session_id';
        $result = $this->handler->execute($sql, $where);
        return $result ? true : false;
    }
 
    /**
     * Session 垃圾回收
     * @access public
     * @param string $sessMaxLifeTime
     * @return bool
     */
    public function gc($sessMaxLifeTime)
    {
        $sql = 'DELETE FROM ' . $this->table_name . ' WHERE session_expire < :session_expire';
        return $this->handler->execute($sql, ['session_expire' => time()]);
    }
 
}
```

## Session 的配置

```php
'session'   => [
    'type'           => 'driver\session\Mysql',
    'auto_start'     => true,
    'session_expire' => 3600,
    'session_prefix' => 'think_',
    'table_name'     => 'think_session',
    'database'       => [
        'hostname' => '127.0.0.1',
        'database' => '',
        'username' => 'root',
        'password' => '',
        'hostport' => '3306',
        'prefix'   => '',
        'charset'  => 'utf8',
    ]
],
```

# 第二种实现

## 建表语句

```sql
CREATE TABLE `think_session` (
  `session_id` varchar(255) CHARACTER SET utf8 NOT NULL,
  `session_expire` int(11) NOT NULL DEFAULT '0' COMMENT 'SESSION的过期时间',
  `session_data` blob COMMENT 'SESSION的数据内容',
  `user_name` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
  `ip` varchar(255) CHARACTER SET utf8 COLLATE utf8_general_ci NOT NULL DEFAULT '',
  UNIQUE KEY `session_id` (`session_id`),
  KEY `NewIndex1` (`session_expire`),
  KEY `Uname` (`user_name`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci
```



## `Mysql` 类

```php
namespace think\session\driver;


use SessionHandlerInterface; //PHP实现预留session接口
use think\Db;
use think\Exception;
use think\facade\Config;


class Mysql implements SessionHandlerInterface
{
    protected $handler = null;
    protected $config = [
        'hostname' => '',// 服务器地址
        'database' => '',// 数据库名
        'username' => '',// 用户名
        'password' => '',// 密码
        'hostport' => '',// 端口
        'charset' => '', // 数据库编码默认采用utf8
        'expire' => 3600, // session有效期
        'session_name' => '', // sessionkey前缀
        'session_table' => '', // session存储的数据表名称
    ];
    protected $table_name = null;


    public function __construct($config = [])
    {
        //获取数据库配置,将数据库配置更新
        $this->config = array_merge($this->config, Config::get('database.'), $config);
        $this->table_name = empty($this->config['session_table']) ? $this->config['prefix'] . '_session' : $this->config['session_table'];
    }


    /**
     * 打开Session
     * @access public
     * @param string $savePath
     * @param mixed $sessName
     */
    public function open($savePath, $sessName)
    {
        if (empty($this->config['hostname'])) throw new Exception('database config error');
        $this->handler = Db::connect($this->config);
        return true;
    }


    /**
     * 关闭Session
     * @access public
     */
    public function close()
    {
        $this->gc(ini_get('session.gc_maxlifetime'));
        $this->handler = null;
        return true;
    }


    /**
     * 读取Session
     * @access public
     * @param string $sessID
     */
    public function read($sessID)
    {
        return (string)Db::table($this->table_name)->where([['session_id', '=', $this->config['session_name'] . $sessID], ['session_expire', '>=', time()]])->value('session_data');
    }


    /**
     * 写入Session
     * @access public
     * @param string $sessID
     * @param string $sessData
     * @return bool
     */
    public function write($sessID, $sessData)
    {
        //获取通过session传入的隐藏参数,用于其他操作
        $unserialize_session_data = unserialize(explode('|', $sessData)[1]);


        //构建存入数据库的数据
        $params = [
            'session_id' => $this->config['session_name'] . $sessID,
            'session_expire' => $this->config['expire'] + time(),
            'session_data' => $sessData,  
            'user_name' => $user_name,
            'ip' => get_ip(),
       
        ];


        $sql = "REPLACE INTO {$this->table_name} (session_id,session_expire,session_data,user_name,ip) VALUES (:session_id, :session_expire, :session_data,:user_name,:ip)";
        $result = $this->handler->execute($sql, $params);
        return $result ? true : false;
    }


    /**
     * 删除Session
     * @access public
     * @param string $sessID
     * @return bool
     */
    public function destroy($sessID)
    {
        $result = $this->handler->execute("DELETE FROM {$this->table_name} WHERE session_id = :session_id", ['session_id' => $this->config['session_name'] . $sessID]);
        return $result ? true : false;
    }


    /**
     * Session 垃圾回收
     * @access public
     * @param string $sessMaxLifeTime
     * @return true
     */
    public function gc($sessMaxLifeTime)
    {
        $result = $this->handler->execute("DELETE FROM {$this->table_name} WHERE session_expire < :session_expire", ['session_expire' => time()]);
        return $result ? true : false;
    }
}
```

