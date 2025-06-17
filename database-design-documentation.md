# 数据库设计文档

#### 1 用户表（`users`）

| 字段名             | 数据类型         | 说明                           | 备注                     |
| --------------- | ------------ | ---------------------------- | ---------------------- |
| \`id\`          | INT          | 用户表的 ID，主键，不自增，自定义的简化版雪花id   |                        |
| \`uname\`       | VARCHAR(255) | 用户名，可重复，不可为空；                |                        |
| \`account\`     | VARCHAR(255) | 用户账号，不可重复，不可为空               |                        |
| \`password\`    | VARCHAR(255) | 用户密码，不可为空                    |                        |
| \`mail\`        | VARCHAR(255) | 用户邮箱                         |                        |
| \`power\`       | INT          | 用户的权限，使用位掩码的权限存储机制存储权限，默认值为3 |                        |
| \`is\_disable\` | TINYINT      | 是否为禁用状态，默认值为0；0：没有禁用，1：已禁用   |                        |
| \`created\_at\` | TIMESTAMP    | 创建时间，默认当前时间                  | 创建时自动载入当前时间            |
| \`created\_by\` | VARCHAR(255) | 创建人                          |                        |
| \`update\_at\`  | TIMESTAMP    | 修改时间，默认当前时间                  | 创建时自动载入当前时间，修改时更新当前时间  |
| \`update\_by\`  | VARCHAR(255) | 修改人                          |                        |
| \`disable\_at\` | TIMESTAMP    | 禁用时间                         | 创建、修改时均不自动载入任何时间，由程序控制 |
| \`disable\_by\` | VARCHAR(255) | 禁用人                          |                        |

**索引**：

- `idx_account` UNIQUE (`account`) -- 账号唯一索引，用于登录和用户查询
- `idx_uname` (`uname`) -- 用户名普通索引，用于用户搜索
- `idx_mail` (`mail`) -- 邮箱普通索引，用于找回密码等操作

表中的固有数据：

| ID                 | UNAME  | ACCOUNT | PASSWORD                                                         | MAIL                      | POWER | IS\\\_DISABLE | CREATED\\\_AT | CREATED\\\_BY | UPDATE\\\_AT | UPDATE\\\_BY |
| ------------------ | ------ | ------- | ---------------------------------------------------------------- | ------------------------- | ----- | ------------- | ------------- | ------------- | ------------ | ------------ |
| 1（root用户特殊不使用雪花id） | honlnk | root    | eb00f6599576b03df5f992244a062946533cac3a0928addec1da580cc8482437 | jihongshen1\\@outlook.com | 2047  | 0             | <默认当前时间>      | root          | <默认当前时间>     | root         |

#### 2 权限表(`power`)

| 字段名          | 数据类型         | 说明          |
| ------------ | ------------ | ----------- |
| \`id\`       | INT          | 权限表ID，主键，自增 |
| \`name\`     | VARCHAR(255) | 权限名，不可为空    |
| \`value\`    | INT          | 权限值，不可为空    |
| \`cn\_name\` | VARCHAR(255) | 中文权限名       |

表中的固有数据：

| ID | 权限名             | 权限数值 | 权限中文名      |
| -- | --------------- | ---- | ---------- |
| 1  | GET             | 1    | 允许查看       |
| 2  | COMMENT         | 2    | 允许评论       |
| 3  | ADD             | 4    | 允许新增       |
| 4  | UPDATE          | 8    | 允许修改       |
| 5  | DELETE          | 16   | 允许删除       |
| 6  | WORD\\\_TEST    | 32   | 允许使用单词测试功能 |
| 7  | MANAGE\\\_USERS | 64   | 允许管理其他用户   |

#### 3 单词表（`words`）

| 字段名                             | 数据类型         | 说明                                                                      |
| ------------------------------- | ------------ | ----------------------------------------------------------------------- |
| `id`                            | INT          | 单词表的 ID，主键，不自增，自定义的简化版雪花id                                              |
| `word`                          | VARCHAR(255) | 单词，不可为空                                                                 |
| `base_translation`              | VARCHAR(255) | 基础翻译字符串                                                                 |
| `explain`                       | TEXT         | 对单词的解释，或者辅助记忆方法                                                         |
| `sense`                         | TEXT         | 单词表达的含义，比如在牛津词典中的解释；这个字段存储的并不是单词的直译，而是用一段文本对这个单词进行一个通用的解释。讲解这个单词究竟想表达什么 |
| `us_symbol`                     | VARCHAR(255) | 美式音标                                                                    |
| `en_symbol`                     | VARCHAR(255) | 英式音标                                                                    |
| `is_alone`                      | TINYINT      | 这个单词是否是独立(没有词根词缀，没用与其存在派生关系的单词等)单词；默认值为1；0表示不是独立的，1表示是独立的               |
| `superordinate_id`              | INT          | 这个单词的“上义词”ID，不需要外键。                                                     |
| `translations_id_list`          | JSON         | 这个单词的翻译的先后顺序（翻译表中对应的ID）                                                 |
| `roots_id_list`                 | JSON         | 这个单词的词根的ID列表                                                            |
| `prefixes_id_list`              | JSON         | 这个单词的前缀的ID列表                                                            |
| `suffixes_id_list`              | JSON         | 这个单词的后缀的ID列表                                                            |
| `roots_translations_id_list`    | JSON         | 这个单词的词根的翻译的先后顺序（翻译表中对应的ID）                                              |
| `prefixes_translations_id_list` | JSON         | 这个单词的前缀的翻译的先后顺序（翻译表中对应的ID）                                              |
| `suffixes_translations_id_list` | JSON         | 这个单词的后缀的翻译的先后顺序（翻译表中对应的ID）                                              |
| `dict_data_json_url`            | VARCHAR(255) | 词典查询结果json文件地址                                                          |
| `ai_explain_md_url`             | VARCHAR(255) | AI解释单词md文件地址                                                            |
| `en_audio_url`                  | VARCHAR(255) | 英文语音文件地址                                                                |
| `zh_audio_url`                  | VARCHAR(255) | 中文语音文件地址                                                                |
| `created_at`                    | TIMESTAMP    | 创建时间，默认当前时间                                                             |
| `created_by`                    | VARCHAR(255) | 创建人                                                                     |
| `update_at`                     | TIMESTAMP    | 修改时间，默认当前时间                                                             |
| `update_by`                     | VARCHAR(255) | 修改人                                                                     |


**索引**：

- `idx_word` (`word`) -- 单词名索引，用于单词搜索和查询

#### 4 前缀表（`prefixes`）

`from_word_id` 外键关联 `words(id)`

| 字段名            | 数据类型         | 说明                         |
| -------------- | ------------ | -------------------------- |
| `id`           | INT          | 前缀表的 ID，主键，不自增，自定义的简化版雪花id |
| `prefix`       | VARCHAR(255) | 前缀，不可为空                    |
| `explain`      | TEXT         | 对前缀的解释，或者辅助记忆方法            |
| `from_word_id` | INT          | 来自哪个单词，外键到`words`          |
| `count`        | INT          | 一共有多少单词使用了这个前缀             |
| `created_at`   | TIMESTAMP    | 创建时间，默认当前时间                |
| `created_by`   | VARCHAR(255) | 创建人                        |
| `update_at`    | TIMESTAMP    | 修改时间，默认当前时间                |
| `update_by`    | VARCHAR(255) | 修改人                        |

#### 5 后缀表（`suffixes`）

`from_word_id` 外键关联 `words(id)`

| 字段名            | 数据类型         | 说明                         |
| -------------- | ------------ | -------------------------- |
| `id`           | INT          | 后缀表的 ID，主键，不自增，自定义的简化版雪花id |
| `suffix`       | VARCHAR(255) | 后缀，不可为空                    |
| `explain`      | TEXT         | 对后缀的解释，或者辅助记忆方法            |
| `from_word_id` | INT          | 来自哪个单词，外键到`words`          |
| `count`        | INT          | 一共有多少单词使用了这个后缀             |
| `created_at`   | TIMESTAMP    | 创建时间，默认当前时间                |
| `created_by`   | VARCHAR(255) | 创建人                        |
| `update_at`    | TIMESTAMP    | 修改时间，默认当前时间                |
| `update_by`    | VARCHAR(255) | 修改人                        |

#### 6 词根表（`roots`）

`from_word_id` 外键关联 `words(id)`

| 字段名            | 数据类型         | 说明                         |
| -------------- | ------------ | -------------------------- |
| `id`           | INT          | 词根表的 ID，主键，不自增，自定义的简化版雪花id |
| `root`         | VARCHAR(255) | 词根，不可为空                    |
| `explain`      | TEXT         | 对词根的解释，或者辅助记忆方法            |
| `from_word_id` | INT          | 来自哪个单词，外键到`words`          |
| `count`        | INT          | 一共有多少单词使用了这个词根             |
| `created_at`   | TIMESTAMP    | 创建时间，默认当前时间                |
| `created_by`   | VARCHAR(255) | 创建人                        |
| `update_at`    | TIMESTAMP    | 修改时间，默认当前时间                |
| `update_by`    | VARCHAR(255) | 修改人                        |

#### 7 词性表（`part_speech`）

| 字段名                   | 数据类型         | 说明                  |
| --------------------- | ------------ | ------------------- |
| \`id\`                | TINYINT      | 词性表的ID，主键，自增        |
| \`name\`              | VARCHAR(10)  | 词性的名字，比如：名词、动词、形容词  |
| \`simple\`            | VARCHAR(10)  | 词性的简写，比如：n.、v.、adj. |
| \`explain\`           | TEXT         | 对词根的解释              |
| \`simplify\_explain\` | VARCHAR(255) | 词性解释的精简版            |

表中的固有数据：

| name | simple  | explain                | simplify\\\_explain |
| ---- | ------- | ---------------------- | ------------------- |
| 无    | null.   | 无词性                    | 无词性                 |
| 名词   | n.      | 表示人、地点、事物或抽象概念的词语      | 表示实体或概念             |
| 动词   | v.      | 表示动作、状态或事件的词语          | 表示行为或状态             |
| 形容词  | adj.    | 用来描述名词特征或状态的词语         | 描述名词特征              |
| 副词   | adv.    | 修饰动词、形容词或其他副词，表示程度、方式等 | 修饰其他词类              |
| 介词   | prep.   | 表示名词与其他词之间的关系          | 表示名词间的关系            |
| 连词   | conj.   | 用来连接单词、短语或句子的词语        | 连接不同的语法单位           |
| 感叹词  | interj. | 表达强烈情感或反应的词语           | 表达情感                |
| 冠词   | art.    | 用在名词前帮助说明名词指的是哪一类或哪一个  | 限定名词                |
| 代词   | pron.   | 代替名词使用的词语              | 替代名词使用              |
| 数词   | num.    | 表示数量或顺序的词语             | 表示数或序               |
| 助词   | part.   | 附着于其他词汇以改变其含义或功能       | 改变词义或功能             |
| 语气词  | mod.    | 用来表达说话者的态度或意图          | 表达态度或意图             |
| 拟声词  | onom.   | 模拟自然界声响而造的词汇           | 模拟声音的词语             |
| 量词   | meas.   | 表示人、事物或动作的数量单位的词语      | 表数量单位               |

#### 8 翻译表（`translations`）

`part_id` 外键关联 `part_speech(id)`

| 字段名           | 数据类型         | 说明                         |
| ------------- | ------------ | -------------------------- |
| `id`          | INT          | 翻译表的 ID，主键，不自增，自定义的简化版雪花id |
| `translation` | VARCHAR(255) | 单词、前缀、后缀、词根的翻译，不可为空        |
| `part_id`     | TINYINT      | 词性id，外键关联到`part_speech`表。  |
| `created_at`  | TIMESTAMP    | 创建时间，默认当前时间                |
| `created_by`  | VARCHAR(255) | 创建人                        |
| `update_at`   | TIMESTAMP    | 修改时间，默认当前时间                |
| `update_by`   | VARCHAR(255) | 修改人                        |

**索引**：

- `idx_translation` (`translation`) -- 翻译文本索引，用于翻译内容搜索

#### 9 留言表（`messages`）

- `user_id` 外键关联 `users(id)`，**级联删除**
- `word_id` 外键关联 `words(id)`，**级联删除**
- `father_id` 外键关联 `messages(id)`，**级联删除**

| 字段名            | 数据类型      | 说明                         |
| -------------- | --------- | -------------------------- |
| `id`           | BIGINT    | 留言表的 ID，主键，不自增，雪花id        |
| `message_text` | TEXT      | 留言文本，不可为空                  |
| `user_id`      | INT       | 用户 ID，不可为空，外键关联到 `users` 表 |
| `word_id`      | INT       | 单词 ID，不可为空，外键关联到 `words` 表 |
| `father_id`    | INT       | 父留言 ID                     |
| `above_id`     | INT       | 上文留言ID：这个留言是回复的那个留言的id     |
| `created_at`   | TIMESTAMP | 创建时间，默认当前时间                |

**索引**：

- `idx_word_id` (`word_id`) -- 单词ID索引，用于查询单词相关留言
- `idx_user_id` (`user_id`) -- 用户ID索引，用于查询用户留言
- `idx_father_id` (`father_id`) -- 父留言ID索引，用于层级关系查询
- `idx_created_at` (`created_at`) -- 创建时间索引，用于按时间排序

#### 10 最近复习的单词(`recent_reviews`)

- `word_id` 外键关联 `words(id)`，**级联删除**
- `user_id`外键关联 `users(id)`，**级联删除**

| 字段名               | 数据类型      | 说明                 |
| ----------------- | --------- | ------------------ |
| \`id\`            | BIGINT    | 最近复习ID，主键，不自增，雪花id |
| \`word\_id\`      | INT       | 外键，单词的id           |
| \`user\_id\`      | INT       | 外键，用户id            |
| \`review\_time\`  | TIMESTAMP | 复习时间，具体到天（包含天）     |
| \`is\_error\`     | TINYINT   | 是否记错               |
| \`error\_reason\` | TEXT      | 记错原因               |

**索引**：

- `idx_user_word` (`user_id`, `word_id`) -- 用户+单词联合索引，用于查询用户对特定单词的复习记录
- `idx_review_date` (`review_date`) -- 复习日期索引，用于按日期范围查询
- `idx_user_review` (`user_id`, `review_time`) -- 用户+时间联合索引，用于查询用户复习历史

#### 11 复习结果(`reviews_outcome`)

- `word_id` 外键关联 `words(id)`，**级联删除**
- `user_id`外键关联 `users(id)`，**级联删除**

| 字段                        | 数据类型     | 说明                        |
| ------------------------- | -------- | ------------------------- |
| \`id\`                    | INT      | 复习结果ID，主键，不自增，自定义的简化版雪花id |
| \`word\_id\`              | INT      | 外键，单词id                   |
| \`user\_id\`              | INT      | 外键，用户id                   |
| \`success\_count\`        | INT      | 记对总次数                     |
| \`error\_count\`          | INT      | 记错总次数                     |
| \`error\_reason\_list\`   | JSON     | 错误原因列表                    |
| \`next\_review\_time\`    | DATETIME | 下次复习时间                    |
| \`finally\_review\_time\` | DATETIME | 最后复习时间                    |

**索引**：

- `idx_user_word` (`user_id`, `word_id`) -- 用户+单词联合索引，用于查询用户对特定单词的复习结果
- `idx_next_review` (`next_review_time`) -- 下次复习时间索引，用于查询待复习单词

#### 12 多对多关系表 (9张关系表)

- **单词与前缀关系表（****`word_prefix`****）**
  - `word_id` 外键关联 `words(id)`，**级联删除**
  - `prefix_id` 外键关联 `prefixes(id)`，**级联删除**

|  字段名         | 数据类型                     | 说明                       |
| ----------- | ------------------------ | ------------------------ |
| `word_id`   | INT                      | 单词 ID，外键关联到 `words` 表    |
| `prefix_id` | INT                      | 前缀 ID，外键关联到 `prefixes` 表 |
| **主键**      | (`word_id`, `prefix_id`) |                          |

- **单词与后缀关系表（****`word_suffix`****）**
  - `word_id` 外键关联 `words(id)`，**级联删除**
  - `suffix_id`外键关联 `suffixes(id)`，**级联删除**

| 字段名         | 数据类型                     | 说明                       |
| ----------- | ------------------------ | ------------------------ |
| `word_id`   | INT                      | 单词 ID，外键关联到 `words` 表    |
| `suffix_id` | INT                      | 后缀 ID，外键关联到 `suffixes` 表 |
| **主键**      | (`word_id`, `suffix_id`) |                          |

- **单词与词根关系表（****`word_root`****）**
  - `word_id` 外键关联 `words(id)`，**级联删除**
  - `root_id`外键关联 `roots``(id)`，**级联删除**

| 字段名       | 数据类型                   | 说明                    |
| --------- | ---------------------- | --------------------- |
| `word_id` | INT                    | 单词 ID，外键关联到 `words` 表 |
| `root_id` | INT                    | 词根 ID，外键关联到 `roots` 表 |
| **主键**    | (`word_id`, `root_id`) |                       |

- **单词与翻译关系表（****`word_translation`****）**
  - `word_id` 外键关联 `words(id)`，**级联删除**
  - `translation_id`外键关联 `translations``(id)`，**级联删除**

| 字段名              | 数据类型                          | 说明                           |
| ---------------- | ----------------------------- | ---------------------------- |
| `word_id`        | INT                           | 单词 ID，外键关联到 `words` 表        |
| `translation_id` | INT                           | 含义 ID，外键关联到 `translations` 表 |
| **主键**           | (`word_id`, `translation_id`) |                              |

- **单词与词性关系表**（`word_part_speech`）
  - `word_id` 外键关联 `words(id)`，**级联删除**
  - `part_speech_id`外键关联 `part_speech(id)`，**级联删除**

| 字段名              | 数据类型                          | 说明                         |
| ---------------- | ----------------------------- | -------------------------- |
| `word_id`        | INT                           | 单词 ID，外键关联到 `words` 表      |
| `part_speech_id` | INT                           | 含义 ID，外键关联到 `part_speech`表 |
| **主键**           | (`word_id`, `part_speech_id`) |                            |

- **前缀与翻译关系表（****`prefix_translation`****）**
  - `prefix_id`外键关联 `prefixes``(id)`，**级联删除**
  - `translation_id`外键关联 `translations``(id)`，**级联删除**

| 字段名              | 数据类型                            | 说明                           |
| ---------------- | ------------------------------- | ---------------------------- |
| `prefix_id`      | INT                             | 前缀 ID，外键关联到 `prefixes` 表     |
| `translation_id` | INT                             | 含义 ID，外键关联到 `translations` 表 |
| **主键**           | (`prefix_id`, `translation_id`) |                              |

- **后缀与翻译关系表（****`suffix_translation`****）**
  - `suffix_id`外键关联 `suffixes``(id)`，**级联删除**
  - `translation_id`外键关联 `translations``(id)`，**级联删除**

| 字段名              | 数据类型                            | 说明                           |
| ---------------- | ------------------------------- | ---------------------------- |
| `suffix_id`      | INT                             | 后缀 ID，外键关联到 `suffixes` 表     |
| `translation_id` | INT                             | 含义 ID，外键关联到 `translations` 表 |
| **主键**           | (`suffix_id`, `translation_id`) |                              |
- **词根与翻译关系表（****`root_translation`****）**
  - `root_id`外键关联 `roots``(id)`，**级联删除**
  - `translation_id`外键关联 `translations``(id)`，**级联删除**

| 字段名              | 数据类型                          | 说明                           |
| ---------------- | ----------------------------- | ---------------------------- |
| `root_id`        | INT                           | 词根 ID，外键关联到 `roots` 表        |
| `translation_id` | INT                           | 含义 ID，外键关联到 `translations` 表 |
| **主键**           | (`root_id`, `translation_id`) |                              |

- **单词派生关系表**（`derivative_words`）

| 字段名                                | 数据类型         | 说明                                        |
| ---------------------------------- | ------------ | ----------------------------------------- |
| \`original\_id\`                   | INT          | 单词id，外键到\`words\`表，最原始的单词的id(大部分情况是最短的那个) |
| \`derivative\_id\`                 | INT          | 单词id，外键到\`words\`表，被派生的单词。                |
| \`derivative\_type\`               | VARCHAR(255) | 派生类型                                      |
| \`explain\`                        | TEXT         | 对相关派生的解释                                  |
| 所有多对多关系表均添加联合主键索引（已存在），同时补充反向查询索引： |              |                                           |
  
- **单词与前缀关系表（****`word_prefix`****）**

    **索引**：
    - 主键 (`word_id`, `prefix_id`)
    - `idx_prefix` (`prefix_id`) -- 前缀ID索引，用于查询使用特定前缀的单词
- **单词与后缀关系表（****`word_suffix`****）**

    **索引**：
    - 主键 (`word_id`, `suffix_id`)
    - `idx_suffix` (`suffix_id`) -- 后缀ID索引，用于查询使用特定后缀的单词
- **单词与词根关系表（****`word_root`****）**

    **索引**：
    - 主键 (`word_id`, `root_id`)
    - `idx_root` (`root_id`) -- 词根ID索引，用于查询使用特定词根的单词
- **单词与翻译关系表（****`word_translation`****）**

    **索引**：
    - 主键 (`word_id`, `translation_id`)
    - `idx_translation` (`translation_id`) -- 翻译ID索引，用于查询特定翻译对应的单词
      ...(其他多对多表索引模式相同)...

#### 13 百度翻译配置表(`baidu_serve_info`)

| 字段名             | 数据类型         | 说明                    |
| --------------- | ------------ | --------------------- |
| id              | tinyint      | ID，主键，自增              |
| api\\\_key      | VARCHAR(24)  | 百度云服务的api\\\_key      |
| secret\\\_key   | VARCHAR(32)  | 百度云服务的secret\\\_key   |
| access\\\_token | VARCHAR(255) | 百度云服务发送请求携带的合法token标识 |
| name            | VARCHAR(255) | 自定义的服务名               |
| api\\\_url      | VARCHAR(255) | 百度云服务的api\\\_url      |

表中的固有数据：

| ID | API\\\_KEY               | SECRET                           | ACCESS\\\_TOKEN | NAME            | API\\\_URL                                                                                                                                                                                 |
| -- | ------------------------ | -------------------------------- | --------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1  | TWWJpUzOIHtsWPic2JbfJV8t | 5QzWmx7BoD5tEMq2LxatfRPPzEoLtO5Y |                 | 百度机器翻译-文本翻译-词典版 | \[<https://aip.baidubce.com/rpc/2.0/mt/texttrans-with-dict/v1](https://aip.baidubce.com/rpc/2.0/mt/texttrans-with-dict/v1> "<https://aip.baidubce.com/rpc/2.0/mt/texttrans-with-dict/v1>") |

**索引**：

- `idx_word` (`word`) -- 单词索引，用于快速查询翻译结果

#### 14 百度翻译结果(`baidu_dist`)

| 字段名                  | 数据类型         | 说明                          |
| -------------------- | ------------ | --------------------------- |
| id                   | INT          | 翻译结果的 ID，主键，不自增，自定义的简化版雪花id |
| word                 | VARCHAR(255) | 单词本身                        |
| data\_oss\_ url      | VARCHAR(255) | 单词翻译结果的OSS路径                |
| zh\_audio\_ oss\_url | VARCHAR(255) | 单词的中文音频OSS路径                |
| en\_audio\_oss\_ url | VARCHAR(255) | 单词的英文音频OSS路径                |

#### 15 AI讲解单词(`ai_word_explain`)

| 字段名             | 数据类型         | 说明                            |
| --------------- | ------------ | ----------------------------- |
| id              | INT          | 单词讲解数据的 ID，主键，不自增，自定义的简化版雪花id |
| word            | VARCHAR(255) | 单词本身                          |
| data\_oss\_ url | VARCHAR(255) | 单词讲解数据的OSS路径                  |


