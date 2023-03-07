## 配置和数据库



### 配置

- https://github.com/spf13/viper
- https://pkg.go.dev/github.com/spf13/viper
- https://darjun.github.io/2020/01/18/godailylib/viper/

利用`viper`来加载配置文件，`viper`功能强大，可以解析`toml`格式的字段称为一个结构体，同时，支持多种格式例如`json`配置文件的读取，支持多路径查找和多种格式的配置文件查找和读取。

```shell
go get -u github.com/spf13/viper
```

首先写一下`toml`文件：

`app/config.toml`

```toml
appName = "chat_room"
staticPath = "static/file/"

[dbConfig]
host = "127.0.0.1"
port = 3306
userName = "root"
password = "227008"
dbName = "chat"
tablePrefix = ""

[log]
level = "debug"
logPath = "logs/chat.log"
errorLogPath = "logs/chat_error.log"
```

针对各个部分编写结构体：

`app/config/config.go`

```go
type Config struct {
	AppName    string
	DBConfig   DBConfig
	Log        LogConfig
	StaticPath string
}

type DBConfig struct {
	Host        string
	Port        string
	UserName    string
	Password    string
	DBName      string
	TablePrefix string
}

type LogConfig struct {
	LogPath      string
	ErrorLogPath string
	Level        string
}
```

一定要初始化，我选择手动初始化方法`MustInitConfig`，也可以用Golang自带的初始化方法`init`：

```go
var c Config

func MustInitConfig(path string) {
	// set config file name
	viper.SetConfigName("config")
	// set config file type
	viper.SetConfigType("toml")
	// set config path, multi-path traverse in order
	viper.AddConfigPath(path)
	viper.AutomaticEnv()
	if err := viper.ReadInConfig(); err != nil {
		panic(fmt.Errorf("fatal error config file: %s", err))
	}

	viper.Unmarshal(&c)
}

func GetConfig() *Config {
	return &c
}
```

这里我们只设置一个路径和一种格式的配置文件，并解析到`Config`结构体内。测试一下：

```go
MustInitConfig("../")
fmt.Printf("%#+v\n", c)
```

测试结果：

```shell
config.Config{AppName:"chat_room", DBConfig:config.DBConfig{Host:"127.0.0.1", Port:"3306", UserName:"root", Password:"227008", DBName:"chat", TablePrefix:""}, Log:config.LogConfig{LogPath:"logs/chat.log", ErrorLogPath:"logs/chat_error.log", Level:"debug"}, StaticPath:"static/file/"}
```



## 数据库

在`app/config.toml`中我们提到了数据库部分：

```toml
[dbConfig]
host = "127.0.0.1"
port = 3306
userName = "root"
password = "227008"
dbName = "chat"
tablePrefix = ""
```

我们将使用`MySQL`作为数据库，首先去执行数据库创建的`database/chat.sql`：

```sql
CREATE DATABASE chat;

USE `chat`;

DROP TABLE IF EXISTS `users`;
CREATE TABLE IF NOT EXISTS `users`
(
    `id`        int          NOT NULL AUTO_INCREMENT COMMENT 'id',
    `uuid`      varchar(150) NOT NULL COMMENT 'uuid',
    `username`  varchar(191) NOT NULL COMMENT '''用户名''',
    `nickname`  varchar(255) DEFAULT NULL COMMENT '昵称',
    `email`     varchar(80)  DEFAULT NULL COMMENT '邮箱',
    `password`  varchar(150) NOT NULL COMMENT '密码',
    `avatar`    varchar(250) NOT NULL COMMENT '头像',
    `create_at` datetime(3)  DEFAULT NULL,
    `update_at` datetime(3)  DEFAULT NULL,
    `delete_at` bigint       DEFAULT NULL,
    PRIMARY KEY (`id`),
    UNIQUE KEY `username` (`username`),
    UNIQUE KEY `idx_uuid` (`uuid`),
    UNIQUE KEY `username_2` (`username`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_0900_ai_ci COMMENT '用户表';


DROP TABLE IF EXISTS `user_friends`;
CREATE TABLE IF NOT EXISTS `user_friends`
(
    `id`         int NOT NULL AUTO_INCREMENT COMMENT 'id',
    `created_at` datetime(3)     DEFAULT NULL COMMENT '创建时间',
    `updated_at` datetime(3)     DEFAULT NULL COMMENT '更新时间',
    `deleted_at` bigint unsigned DEFAULT NULL COMMENT '删除时间戳',
    `user_id`    int             DEFAULT NULL COMMENT '用户ID',
    `friend_id`  int             DEFAULT NULL COMMENT '好友ID',
    PRIMARY KEY (`id`),
    KEY `idx_user_friends_user_id` (`user_id`),
    KEY `idx_user_friends_friend_id` (`friend_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_0900_ai_ci COMMENT '好友信息表';


DROP TABLE IF EXISTS `messages`;
CREATE TABLE IF NOT EXISTS `messages`
(
    `id`           bigint unsigned NOT NULL AUTO_INCREMENT COMMENT 'id',
    `created_at`   datetime(3)     DEFAULT NULL COMMENT '创建时间',
    `updated_at`   datetime(3)     DEFAULT NULL COMMENT '更新时间',
    `deleted_at`   bigint unsigned DEFAULT NULL COMMENT '删除时间戳',
    `from_user_id` int             DEFAULT NULL COMMENT '发送人ID',
    `to_user_id`   int             DEFAULT NULL COMMENT '发送对象ID',
    `content`      varchar(2500)   DEFAULT NULL COMMENT '消息内容',
    `url`          varchar(350)    DEFAULT NULL COMMENT '''文件或者图片地址''',
    `pic`          text COMMENT '缩略图',
    `message_type` smallint        DEFAULT NULL COMMENT '''消息类型：1单聊，2群聊''',
    `content_type` smallint        DEFAULT NULL COMMENT '''消息内容类型：1文字，2语音，3视频''',
    PRIMARY KEY (`id`),
    KEY `idx_messages_deleted_at` (`deleted_at`),
    KEY `idx_messages_from_user_id` (`from_user_id`),
    KEY `idx_messages_to_user_id` (`to_user_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_0900_ai_ci COMMENT '消息表';


DROP TABLE IF EXISTS `groups`;
CREATE TABLE IF NOT EXISTS `groups`
(
    `id`         int          NOT NULL AUTO_INCREMENT,
    `created_at` datetime(3)     DEFAULT NULL,
    `updated_at` datetime(3)     DEFAULT NULL,
    `deleted_at` bigint unsigned DEFAULT NULL,
    `user_id`    int             DEFAULT NULL COMMENT '''群主ID''',
    `name`       varchar(150)    DEFAULT NULL COMMENT '''群名称',
    `notice`     varchar(350)    DEFAULT NULL COMMENT '''群公告',
    `uuid`       varchar(150) NOT NULL COMMENT '''uuid''',
    PRIMARY KEY (`id`),
    KEY `idx_groups_user_id` (`user_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_0900_ai_ci COMMENT '群组表';


DROP TABLE IF EXISTS `group_members`;
CREATE TABLE IF NOT EXISTS `group_members`
(
    `id`         int NOT NULL AUTO_INCREMENT,
    `created_at` datetime(3)     DEFAULT NULL,
    `updated_at` datetime(3)     DEFAULT NULL,
    `deleted_at` bigint unsigned DEFAULT NULL,
    `user_id`    int             DEFAULT NULL COMMENT '''用户ID''',
    `group_id`   int             DEFAULT NULL COMMENT '''群组ID''',
    `nickname`   varchar(350)    DEFAULT NULL COMMENT '''昵称',
    `mute`       smallint        DEFAULT NULL COMMENT '''是否禁言''',
    PRIMARY KEY (`id`),
    KEY `idx_group_members_user_id` (`user_id`),
    KEY `idx_group_members_group_id` (`group_id`)
) ENGINE = InnoDB
  DEFAULT CHARSET = utf8mb4
  COLLATE = utf8mb4_0900_ai_ci COMMENT '群组成员表';
```

我们将使用`gorm`做数据库连接的部分：

- https://github.com/go-gorm/gorm
- https://gorm.io/docs

`app/common/pool/mysql.go`

```go
package pool

import (
	"fmt"
	"goChat/app/config"
	"gorm.io/driver/mysql"
	"gorm.io/gorm"
	"gorm.io/gorm/logger"
)

var db *gorm.DB

// MustInitDB should be called after config.MustInitConfig
func MustInitDB(timeout string) {
	dbConfig := config.GetConfig().DBConfig
	dsn := fmt.Sprintf(
		"%s:%s@tcp(%s:%d)/%s?charset=utf8&parseTime=True&loc=Local&timeout=%s",
		dbConfig.UserName, dbConfig.Password, dbConfig.Host, dbConfig.Port, dbConfig.DBName, timeout)

	var err error
	db, err = gorm.Open(mysql.Open(dsn), &gorm.Config{
		Logger: logger.Default.LogMode(logger.Info),
	})
	if err != nil {
		panic("Fail to connect the database, error: " + err.Error())
	}

	sqlDB, _ := db.DB()
	sqlDB.SetMaxOpenConns(100)
	sqlDB.SetMaxIdleConns(20)
}

func GetDB() *gorm.DB {
	return db
}
```

创建`dsn`打开连接，并使用底层的`database/sql`设置连接池。

