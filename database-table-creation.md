```sql
-- 1. 用户表 (users)
CREATE TABLE `users` (
  `id` INT NOT NULL COMMENT '用户表 ID，主键，非自增，简化版雪花id',
  `uname` VARCHAR(255) NOT NULL COMMENT '用户名，可重复',
  `account` VARCHAR(255) NOT NULL COMMENT '用户账号，不可重复',
  `password` VARCHAR(255) NOT NULL COMMENT '用户密码',
  `mail` VARCHAR(255) DEFAULT NULL COMMENT '用户邮箱',
  `power` INT NOT NULL DEFAULT 3 COMMENT '用户权限，位掩码存储，默认3',
  `is_disable` TINYINT NOT NULL DEFAULT 0 COMMENT '是否禁用：0=未禁用，1=已禁用',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间，默认为当前时间',
  `created_by` VARCHAR(255) DEFAULT NULL COMMENT '创建人',
  `update_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间，默认为当前时间，更新时自动刷新',
  `update_by` VARCHAR(255) DEFAULT NULL COMMENT '修改人',
  `disable_at` TIMESTAMP NULL DEFAULT NULL COMMENT '禁用时间，程序控制，不自动设置',
  `disable_by` VARCHAR(255) DEFAULT NULL COMMENT '禁用人',
  PRIMARY KEY (`id`),
  UNIQUE KEY `idx_account` (`account`),
  KEY `idx_uname` (`uname`),
  KEY `idx_mail` (`mail`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='用户表'; 

-- 内置数据示例（可择机插入，若需要请单独执行 INSERT 语句）：
-- INSERT INTO `users` (`id`, `uname`, `account`, `password`, `mail`, `power`, `is_disable`, `created_at`, `created_by`, `update_at`, `update_by`)
-- VALUES (1, 'honlnk', 'root', 'eb00f6599576b03df5f992244a062946533cac3a0928addec1da580cc8482437', 'jihongshen1@outlook.com', 2047, 0, NOW(), 'root', NOW(), 'root');

-- 2. 权限表 (power)
CREATE TABLE `power` (
  `id` INT NOT NULL AUTO_INCREMENT COMMENT '权限表 ID，主键，自增',
  `name` VARCHAR(255) NOT NULL COMMENT '权限名',
  `value` INT NOT NULL COMMENT '权限值',
  `cn_name` VARCHAR(255) DEFAULT NULL COMMENT '中文权限名',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='权限表';

-- 固有数据示例：
-- INSERT INTO `power` (`id`, `name`, `value`, `cn_name`) VALUES
-- (1, 'GET', 1, '允许查看'),
-- (2, 'COMMENT', 2, '允许评论'),
-- (3, 'ADD', 4, '允许新增'),
-- (4, 'UPDATE', 8, '允许修改'),
-- (5, 'DELETE', 16, '允许删除'),
-- (6, 'WORD_TEST', 32, '允许使用单词测试功能'),
-- (7, 'MANAGE_USERS', 64, '允许管理其他用户');

-- 3. 单词表 (words)
CREATE TABLE `words` (
  `id` INT NOT NULL COMMENT '单词表 ID，主键，非自增，简化版雪花id',
  `word` VARCHAR(255) NOT NULL COMMENT '单词，不可为空',
  `explain` TEXT DEFAULT NULL COMMENT '单词解释或辅助记忆方法',
  `sense` TEXT DEFAULT NULL COMMENT '单词含义（牛津词典式解释）',
  `phonetic_symbol` VARCHAR(255) DEFAULT NULL COMMENT '美式音标',
  `is_alone` TINYINT NOT NULL DEFAULT 1 COMMENT '是否独立单词：0=不是，1=是；默认1',
  `superordinate_id` INT DEFAULT NULL COMMENT '上义词 ID，非外键',
  `translations_id_list` JSON DEFAULT NULL COMMENT '该单词翻译 ID 列表（JSON）',
  `root_translations_id_list` JSON DEFAULT NULL COMMENT '该单词词根翻译 ID 列表（JSON）',
  `prefixes_translations_id_list` JSON DEFAULT NULL COMMENT '该单词前缀翻译 ID 列表（JSON）',
  `suffixes_translations_id_list` JSON DEFAULT NULL COMMENT '该单词后缀翻译 ID 列表（JSON）',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间，默认为当前时间',
  `created_by` VARCHAR(255) DEFAULT NULL COMMENT '创建人',
  `update_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间，默认为当前时间，更新时刷新',
  `update_by` VARCHAR(255) DEFAULT NULL COMMENT '修改人',
  PRIMARY KEY (`id`),
  KEY `idx_word` (`word`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='单词表';

-- 4. 前缀表 (prefixes)
CREATE TABLE `prefixes` (
  `id` INT NOT NULL COMMENT '前缀表 ID，主键，非自增，简化版雪花id',
  `prefix` VARCHAR(255) NOT NULL COMMENT '前缀，不可为空',
  `explain` TEXT DEFAULT NULL COMMENT '前缀解释或辅助记忆方法',
  `from_word_id` INT NOT NULL COMMENT '来自哪个单词，外键到 words(id)',
  `count` INT NOT NULL DEFAULT 0 COMMENT '使用该前缀的单词数量',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间，默认为当前时间',
  `created_by` VARCHAR(255) DEFAULT NULL COMMENT '创建人',
  `update_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间，默认为当前时间，更新时刷新',
  `update_by` VARCHAR(255) DEFAULT NULL COMMENT '修改人',
  PRIMARY KEY (`id`),
  KEY `idx_from_word_id` (`from_word_id`),
  CONSTRAINT `fk_prefixes_from_word`
    FOREIGN KEY (`from_word_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='前缀表';

-- 5. 后缀表 (suffixes)
CREATE TABLE `suffixes` (
  `id` INT NOT NULL COMMENT '后缀表 ID，主键，非自增，简化版雪花id',
  `suffix` VARCHAR(255) NOT NULL COMMENT '后缀，不可为空',
  `explain` TEXT DEFAULT NULL COMMENT '后缀解释或辅助记忆方法',
  `from_word_id` INT NOT NULL COMMENT '来自哪个单词，外键到 words(id)',
  `count` INT NOT NULL DEFAULT 0 COMMENT '使用该后缀的单词数量',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间，默认为当前时间',
  `created_by` VARCHAR(255) DEFAULT NULL COMMENT '创建人',
  `update_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间，默认为当前时间，更新时刷新',
  `update_by` VARCHAR(255) DEFAULT NULL COMMENT '修改人',
  PRIMARY KEY (`id`),
  KEY `idx_from_word_id` (`from_word_id`),
  CONSTRAINT `fk_suffixes_from_word`
    FOREIGN KEY (`from_word_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='后缀表';

-- 6. 词根表 (roots)
CREATE TABLE `roots` (
  `id` INT NOT NULL COMMENT '词根表 ID，主键，非自增，简化版雪花id',
  `root` VARCHAR(255) NOT NULL COMMENT '词根，不可为空',
  `explain` TEXT DEFAULT NULL COMMENT '词根解释或辅助记忆方法',
  `from_word_id` INT NOT NULL COMMENT '来自哪个单词，外键到 words(id)',
  `count` INT NOT NULL DEFAULT 0 COMMENT '使用该词根的单词数量',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间，默认为当前时间',
  `created_by` VARCHAR(255) DEFAULT NULL COMMENT '创建人',
  `update_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间，默认为当前时间，更新时刷新',
  `update_by` VARCHAR(255) DEFAULT NULL COMMENT '修改人',
  PRIMARY KEY (`id`),
  KEY `idx_from_word_id` (`from_word_id`),
  CONSTRAINT `fk_roots_from_word`
    FOREIGN KEY (`from_word_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='词根表';

-- 7. 词性表 (part_speech)
CREATE TABLE `part_speech` (
  `id` TINYINT NOT NULL AUTO_INCREMENT COMMENT '词性表 ID，主键，自增',
  `name` VARCHAR(10) NOT NULL COMMENT '词性名称，如：名词、动词、形容词',
  `simple` VARCHAR(10) NOT NULL COMMENT '词性简写，如：n.、v.、adj.',
  `explain` TEXT DEFAULT NULL COMMENT '词性解释',
  `simplify_explain` VARCHAR(255) DEFAULT NULL COMMENT '词性解释精简版',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='词性表';

-- 固有数据示例：
-- INSERT INTO `part_speech` (`id`, `name`, `simple`, `explain`, `simplify_explain`) VALUES
-- ('1','名词','n.','表示人、地点、事物或抽象概念的词语','表示实体或概念'),
-- ('2','动词','v.','表示动作、状态或事件的词语','表示行为或状态'),
-- ('3','形容词','adj.','用来描述名词特征或状态的词语','描述名词特征'),
-- ('4','副词','adv.','修饰动词、形容词或其他副词，表示程度、方式等','修饰其他词类'),
-- ('5','介词','prep.','表示名词与其他词之间的关系','表示名词间的关系'),
-- ('6','连词','conj.','用来连接单词、短语或句子的词语','连接不同的语法单位'),
-- ('7','感叹词','interj.','表达强烈情感或反应的词语','表达情感'),
-- ('8','冠词','art.','用在名词前帮助说明名词指的是哪一类或哪一个','限定名词'),
-- ('9','代词','pron.','代替名词使用的词语','替代名词使用'),
-- ('10','数词','num.','表示数量或顺序的词语','表示数或序'),
-- ('11','助词','part.','附着于其他词汇以改变其含义或功能','改变词义或功能'),
-- ('12','语气词','mod.','用来表达说话者的态度或意图','表达态度或意图'),
-- ('13','拟声词','onom.','模拟自然界声响而造的词汇','模拟声音的词语'),
-- ('14','量词','meas.','表示人、事物或动作的数量单位的词语','表数量单位');

-- 8. 翻译表 (translations)
CREATE TABLE `translations` (
  `id` INT NOT NULL COMMENT '翻译表 ID，主键，非自增，简化版雪花id',
  `translation` VARCHAR(255) NOT NULL COMMENT '翻译文本，不可为空',
  `part_id` TINYINT NOT NULL COMMENT '词性 ID，外键关联 part_speech(id)',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间，默认为当前时间',
  `created_by` VARCHAR(255) DEFAULT NULL COMMENT '创建人',
  `update_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间，默认为当前时间，更新时刷新',
  `update_by` VARCHAR(255) DEFAULT NULL COMMENT '修改人',
  PRIMARY KEY (`id`),
  KEY `idx_translation` (`translation`),
  KEY `idx_part_id` (`part_id`),
  CONSTRAINT `fk_translations_part`
    FOREIGN KEY (`part_id`) REFERENCES `part_speech`(`id`)
    ON DELETE RESTRICT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='翻译表';

-- 9. 留言表 (messages)
CREATE TABLE `messages` (
  `id` BIGINT NOT NULL COMMENT '留言表 ID，主键，非自增，雪花id',
  `message_text` TEXT NOT NULL COMMENT '留言文本，不可为空',
  `user_id` INT NOT NULL COMMENT '用户 ID，外键关联 users(id)，级联删除',
  `word_id` INT NOT NULL COMMENT '单词 ID，外键关联 words(id)，级联删除',
  `father_id` BIGINT DEFAULT NULL COMMENT '父留言 ID，外键关联 messages(id)，级联删除',
  `above_id` BIGINT DEFAULT NULL COMMENT '上文留言 ID，用于当前留言是回复哪条留言（未明确要求级联）',
  `created_at` TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间，默认为当前时间',
  PRIMARY KEY (`id`),
  KEY `idx_user_id` (`user_id`),
  KEY `idx_word_id` (`word_id`),
  KEY `idx_father_id` (`father_id`),
  KEY `idx_created_at` (`created_at`),
  CONSTRAINT `fk_messages_user`
    FOREIGN KEY (`user_id`) REFERENCES `users`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_messages_word`
    FOREIGN KEY (`word_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_messages_father`
    FOREIGN KEY (`father_id`) REFERENCES `messages`(`id`)
    ON DELETE CASCADE
  -- 若需对 above_id 也做外键，可加：
  -- ,CONSTRAINT `fk_messages_above`
  --   FOREIGN KEY (`above_id`) REFERENCES `messages`(`id`)
  --   ON DELETE SET NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='留言表';

-- 10. 最近复习的单词 (recent_reviews)
CREATE TABLE `recent_reviews` (
  `id` BIGINT NOT NULL COMMENT '最近复习表 ID，主键，非自增，雪花id',
  `word_id` INT NOT NULL COMMENT '单词 ID，外键关联 words(id)，级联删除',
  `user_id` INT NOT NULL COMMENT '用户 ID，外键关联 users(id)，级联删除',
  `review_time` TIMESTAMP NOT NULL COMMENT '复习时间，精确到天',
  `is_error` TINYINT NOT NULL DEFAULT 0 COMMENT '是否记错：0=未记错，1=记错',
  `error_reason` TEXT DEFAULT NULL COMMENT '记错原因',
  PRIMARY KEY (`id`),
  KEY `idx_user_word` (`user_id`, `word_id`),
  KEY `idx_review_date` (`review_time`),
  KEY `idx_user_review` (`user_id`, `review_time`),
  CONSTRAINT `fk_recent_reviews_word`
    FOREIGN KEY (`word_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_recent_reviews_user`
    FOREIGN KEY (`user_id`) REFERENCES `users`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='最近复习的单词';

-- 11. 复习结果 (reviews_outcome)
CREATE TABLE `reviews_outcome` (
  `id` INT NOT NULL COMMENT '复习结果表 ID，主键，非自增，简化版雪花id',
  `word_id` INT NOT NULL COMMENT '单词 ID，外键关联 words(id)，级联删除',
  `user_id` INT NOT NULL COMMENT '用户 ID，外键关联 users(id)，级联删除',
  `success_count` INT NOT NULL DEFAULT 0 COMMENT '记对总次数',
  `error_count` INT NOT NULL DEFAULT 0 COMMENT '记错总次数',
  `error_reason_list` JSON DEFAULT NULL COMMENT '错误原因列表（JSON）',
  `next_review_time` DATETIME DEFAULT NULL COMMENT '下次复习时间',
  `finally_review_time` DATETIME DEFAULT NULL COMMENT '最后复习时间',
  PRIMARY KEY (`id`),
  KEY `idx_user_word` (`user_id`, `word_id`),
  KEY `idx_next_review` (`next_review_time`),
  CONSTRAINT `fk_reviews_outcome_word`
    FOREIGN KEY (`word_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_reviews_outcome_user`
    FOREIGN KEY (`user_id`) REFERENCES `users`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='复习结果表';

-- 12. 单词与前缀关系表 (word_prefix)
CREATE TABLE `word_prefix` (
  `word_id` INT NOT NULL COMMENT '单词 ID，外键关联 words(id)，级联删除',
  `prefix_id` INT NOT NULL COMMENT '前缀 ID，外键关联 prefixes(id)，级联删除',
  PRIMARY KEY (`word_id`, `prefix_id`),
  KEY `idx_prefix` (`prefix_id`),
  CONSTRAINT `fk_word_prefix_word`
    FOREIGN KEY (`word_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_word_prefix_prefix`
    FOREIGN KEY (`prefix_id`) REFERENCES `prefixes`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='单词与前缀关系表';

-- 13. 单词与后缀关系表 (word_suffix)
CREATE TABLE `word_suffix` (
  `word_id` INT NOT NULL COMMENT '单词 ID，外键关联 words(id)，级联删除',
  `suffix_id` INT NOT NULL COMMENT '后缀 ID，外键关联 suffixes(id)，级联删除',
  PRIMARY KEY (`word_id`, `suffix_id`),
  KEY `idx_suffix` (`suffix_id`),
  CONSTRAINT `fk_word_suffix_word`
    FOREIGN KEY (`word_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_word_suffix_suffix`
    FOREIGN KEY (`suffix_id`) REFERENCES `suffixes`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='单词与后缀关系表';

-- 14. 单词与词根关系表 (word_root)
CREATE TABLE `word_root` (
  `word_id` INT NOT NULL COMMENT '单词 ID，外键关联 words(id)，级联删除',
  `root_id` INT NOT NULL COMMENT '词根 ID，外键关联 roots(id)，级联删除',
  PRIMARY KEY (`word_id`, `root_id`),
  KEY `idx_root` (`root_id`),
  CONSTRAINT `fk_word_root_word`
    FOREIGN KEY (`word_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_word_root_root`
    FOREIGN KEY (`root_id`) REFERENCES `roots`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='单词与词根关系表';

-- 15. 单词与翻译关系表 (word_translation)
CREATE TABLE `word_translation` (
  `word_id` INT NOT NULL COMMENT '单词 ID，外键关联 words(id)，级联删除',
  `translation_id` INT NOT NULL COMMENT '翻译 ID，外键关联 translations(id)，级联删除',
  PRIMARY KEY (`word_id`, `translation_id`),
  KEY `idx_translation` (`translation_id`),
  CONSTRAINT `fk_word_translation_word`
    FOREIGN KEY (`word_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_word_translation_translation`
    FOREIGN KEY (`translation_id`) REFERENCES `translations`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='单词与翻译关系表';

-- 16. 单词与词性关系表 (word_part_speech)
CREATE TABLE `word_part_speech` (
  `word_id` INT NOT NULL COMMENT '单词 ID，外键关联 words(id)，级联删除',
  `part_speech_id` TINYINT NOT NULL COMMENT '词性 ID，外键关联 part_speech(id)，级联删除',
  PRIMARY KEY (`word_id`, `part_speech_id`),
  KEY `idx_part_speech` (`part_speech_id`),
  CONSTRAINT `fk_word_part_speech_word`
    FOREIGN KEY (`word_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_word_part_speech_part`
    FOREIGN KEY (`part_speech_id`) REFERENCES `part_speech`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='单词与词性关系表';


-- 17. 前缀与翻译关系表 (prefix_translation)
CREATE TABLE `prefix_translation` (
  `prefix_id` INT NOT NULL COMMENT '前缀 ID，外键关联 prefixes(id)，级联删除',
  `translation_id` INT NOT NULL COMMENT '翻译 ID，外键关联 translations(id)，级联删除',
  PRIMARY KEY (`prefix_id`, `translation_id`),
  KEY `idx_translation` (`translation_id`),
  CONSTRAINT `fk_prefix_translation_prefix`
    FOREIGN KEY (`prefix_id`) REFERENCES `prefixes`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_prefix_translation_translation`
    FOREIGN KEY (`translation_id`) REFERENCES `translations`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='前缀与翻译关系表';

-- 18. 后缀与翻译关系表 (suffix_translation)
CREATE TABLE `suffix_translation` (
  `suffix_id` INT NOT NULL COMMENT '后缀 ID，外键关联 suffixes(id)，级联删除',
  `translation_id` INT NOT NULL COMMENT '翻译 ID，外键关联 translations(id)，级联删除',
  PRIMARY KEY (`suffix_id`, `translation_id`),
  KEY `idx_translation` (`translation_id`),
  CONSTRAINT `fk_suffix_translation_suffix`
    FOREIGN KEY (`suffix_id`) REFERENCES `suffixes`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_suffix_translation_translation`
    FOREIGN KEY (`translation_id`) REFERENCES `translations`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='后缀与翻译关系表';

-- 19. 词根与翻译关系表 (root_translation)
CREATE TABLE `root_translation` (
  `root_id` INT NOT NULL COMMENT '词根 ID，外键关联 roots(id)，级联删除',
  `translation_id` INT NOT NULL COMMENT '翻译 ID，外键关联 translations(id)，级联删除',
  PRIMARY KEY (`root_id`, `translation_id`),
  KEY `idx_translation` (`translation_id`),
  CONSTRAINT `fk_root_translation_root`
    FOREIGN KEY (`root_id`) REFERENCES `roots`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_root_translation_translation`
    FOREIGN KEY (`translation_id`) REFERENCES `translations`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='词根与翻译关系表';

-- 20. 单词派生关系表 (derivative_words)
CREATE TABLE `derivative_words` (
  `original_id` INT NOT NULL COMMENT '最原始单词 ID，外键关联 words(id)，级联删除',
  `derivative_id` INT NOT NULL COMMENT '派生单词 ID，外键关联 words(id)，级联删除',
  `derivative_type` VARCHAR(255) DEFAULT NULL COMMENT '派生类型',
  `explain` TEXT DEFAULT NULL COMMENT '对相关派生的解释',
  PRIMARY KEY (`original_id`, `derivative_id`),
  KEY `idx_derivative_id` (`derivative_id`),
  CONSTRAINT `fk_derivative_words_original`
    FOREIGN KEY (`original_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE,
  CONSTRAINT `fk_derivative_words_derivative`
    FOREIGN KEY (`derivative_id`) REFERENCES `words`(`id`)
    ON DELETE CASCADE
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='单词派生关系表';

-- 21. 百度翻译配置表 (baidu_serve_info)
CREATE TABLE `baidu_serve_info` (
  `id` TINYINT NOT NULL AUTO_INCREMENT COMMENT 'ID，主键，自增',
  `api_key` VARCHAR(24) NOT NULL COMMENT '百度云服务的 api_key',
  `secret_key` VARCHAR(32) NOT NULL COMMENT '百度云服务的 secret_key',
  `access_token` VARCHAR(255) DEFAULT NULL COMMENT '百度云服务请求的合法 token 标识',
  `name` VARCHAR(255) DEFAULT NULL COMMENT '自定义服务名',
  `api_url` VARCHAR(255) DEFAULT NULL COMMENT '百度云服务 API URL',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='百度翻译配置表';

-- 固有数据示例：
-- INSERT INTO `baidu_serve_info` (`api_key`, `secret_key`, `access_token`, `name`, `api_url`) VALUES
-- ('*****', '*****','','百度机器翻译-文本翻译-词典版','https://aip.baidubce.com/rpc/2.0/mt/texttrans-with-dict/v1');

-- 22. 百度翻译结果 (baidu_dist)
CREATE TABLE `baidu_dist` (
  `id` INT NOT NULL COMMENT '翻译结果 ID，主键，非自增，简化版雪花id',
  `word` VARCHAR(255) DEFAULT NULL COMMENT '单词本身',
  `data_oss_url` VARCHAR(255) DEFAULT NULL COMMENT '翻译结果的 OSS 路径',
  `audo_zh_oss_url` VARCHAR(255) DEFAULT NULL COMMENT '中文音频 OSS 路径',
  `audo_en_oss_url` VARCHAR(255) DEFAULT NULL COMMENT '英文音频 OSS 路径',
  PRIMARY KEY (`id`),
  KEY `idx_word` (`word`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='百度翻译结果表';

-- 23. AI 讲解单词 (ai_word_explain)
CREATE TABLE `ai_word_explain` (
  `id` INT NOT NULL COMMENT '单词讲解数据 ID，主键，非自增，简化版雪花id',
  `word` VARCHAR(255) DEFAULT NULL COMMENT '单词本身',
  `data_oss_url` VARCHAR(255) DEFAULT NULL COMMENT '单词讲解数据 OSS 路径',
  PRIMARY KEY (`id`),
  KEY `idx_word` (`word`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='AI 讲解单词表';

```