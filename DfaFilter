<?php
/**
 * DFA有穷状态自动机
 * Created by PhpStorm.
 * User: Wxz
 * Date: 2020/7/8
 * Time: 11:13
 */
 
class DfaFilter
{
    /**
     * 哈希表变量
     * @var array|null
     */
    private $wordTree = [];
    /**
     * 待检测语句长度
     * @var int
     */
    private $contentLength = 0;
    /**
     * 符号不进行查找
     * @var array
     */
    private $InvalidWords = [' ', ',', '~', '!', '@', '#', '$', '%', '^', '*', '_', '=', '?', '<', '>', '，', '。', '/', '\\', '|', '《', '》', '？', ';', ':', '：', '\'', '‘', '；', '“'];

    private static $_instance = null;

    private function __construct()
    {
    }

    private function __clone()
    {
        // TODO: Implement __clone() method.
    }

    /**
     * @Notes:
     * @Function 单例
     * @author: Wxz
     * @Time: 2020/7/8 11:24
     * @return DfaFilter
     */
    public static function init()
    {
        if (!self::$_instance instanceof self) {
            self::$_instance = new self();
        }
        return self::$_instance;
    }

    /**
     * @Notes:
     * @Function getHashMap
     * @author: Wxz
     * @Time: 2020/7/8 11:24
     * @return array|null
     */
    public function getWordTree()
    {
        return $this->wordTree;
    }

    /**
     * @Notes:
     * @Function setHashMap
     * @author: Wxz
     * @Time: 2020/7/8 11:24
     * @param array $wordTree
     * @return bool
     */
    public function setWordTree($wordTree = [])
    {
        $this->wordTree = $wordTree;
        return true;
    }

    /**
     * @Notes:
     * @Function 构建铭感词树【文件模式】
     * @author: Wxz
     * @Time: 2020/7/8 11:21
     * @param string $filepath
     * @return $this
     * @throws \Exception
     */
    public function setTreeByFile($file_path = '')
    {
        if (!file_exists($file_path) || !is_readable($file_path)) {
            throw new \Exception('词库文件不存在');
        }

        // 词库树初始化
        $this->wordTree = $this->wordTree ?: [];

        foreach ($this->yieldToReadFile($file_path) as $word) {
            $this->buildWordToTree(trim($word));
        }

        return $this;
    }

    /**
     * @Notes:
     * @Function yieldToReadFile
     * @author: Wxz
     * @Time: 2020/7/8 11:22
     * @param $filepath
     * @return \Generator
     */
    protected function yieldToReadFile($file_path = '')
    {
        $fp = fopen($file_path, 'r');
        while (!feof($fp)) {
            yield fgets($fp);
        }
        fclose($fp);
    }

    /**
     * @Notes:
     * @Function 数组形式设置敏感词树
     * @author: Wxz
     * @Time: 2020/7/8 11:28
     * @param array $sensitiveWords
     * @return $this
     * @throws \Exception
     */
    public function setTree($sensitiveWords = [])
    {
        if (empty($sensitiveWords)) {
            throw new \Exception('词库不能为空');
        }

        $this->wordTree = [];

        foreach ($sensitiveWords as $word) {
            $word = Forum::TermBlankSpace($word);
            $this->buildWordToTree($word);
        }
        return $this;
    }

    /**
     * @Notes:
     * @Function 构建敏感词库
     * @author: Wxz
     * @Time: 2020/7/8 11:19
     * @param $strWord
     */
    public function buildWordToTree($strWord)
    {
        if (empty($strWord)) return;

        $length = mb_strlen($strWord, 'UTF-8');

        // 传址递归添加子树
        $arrHashMap = &$this->wordTree;
        for ($i = 0; $i < $length; $i++) {
            $word = mb_substr($strWord, $i, 1, 'UTF-8');

            if (!isset($arrHashMap[$word])) {
                // 不存在
                $arrHashMap[$word] = [];
                $arrHashMap[$word]['end'] = false;
            }
            //判断是否为最后一个字
            if ($i == ($length - 1)) {
                $arrHashMap[$word]['end'] = true;
            }
            // 传址
            $arrHashMap = &$arrHashMap[$word];
        }
    }

    /**
     * 检测文字中的敏感词
     * @author: Wxz
     * @param string $content 待检测内容
     * @param int $matchType 匹配类型 [默认为最小匹配规则]
     * @return array
     */
    public function getBadWord($content = '', $matchType = 1)
    {
        if (empty($content)) {
            return [];
        }

        $this->contentLength = mb_strlen($content, 'utf-8');
        $badWordList = [];

        for ($length = 0; $length < $this->contentLength; $length++) {
            $matchFlag = 0;
            //存放结束的词结束位置
            $flag = [];
            $tempMap = $this->wordTree;
            for ($i = $length; $i < $this->contentLength; $i++) {
                $keyChar = mb_substr($content, $i, 1, 'utf-8');
                //标点符号跳出
                if (in_array($keyChar, $this->InvalidWords)) {
                    break;
                }
                // 获取指定节点树
                $nowMap = $tempMap[$keyChar];

                // 不存在节点树，直接返回
                if (empty($nowMap)) {
                    break;
                }

                // 存在，则判断是否为最后一个
                $tempMap = $nowMap;

                // 找到相应key，偏移量+1
                $matchFlag++;

                // 如果为最后一个匹配规则,结束循环，返回匹配标识数
                if (false === $nowMap['end']) {
                    continue;
                }
                //记录铭感次的结束位置
                $flag[] = $matchFlag;

                // 最小规则，直接退出
                if (1 === $matchType) {
                    break;
                }
            }
            //判断该是否有敏感词
            if (empty($flag)) {
                continue;
            }
            //敏感词
            foreach ($flag as $value) {
                $badWordList[] = mb_substr($content, $length, $value, 'utf-8');
            }
            // 需匹配内容标志位往后移
            $length = $length + end($flag) - 1;
        }
        return $badWordList;
    }

    /**
     * @Notes:
     * @Function 被检测内容是否合法
     * @author: Wxz
     * @Time: 2020/7/8 11:29
     * @param $content
     * @return bool
     */
    public function isLegal($content)
    {
        $this->contentLength = mb_strlen($content, 'utf-8');

        for ($length = 0; $length < $this->contentLength; $length++) {
            $matchFlag = 0;

            $tempMap = $this->wordTree;
            for ($i = $length; $i < $this->contentLength; $i++) {
                $keyChar = mb_substr($content, $i, 1, 'utf-8');

                // 获取指定节点树
                $nowMap = $tempMap[$keyChar];

                // 不存在节点树，直接返回
                if (empty($nowMap)) {
                    break;
                }

                // 找到相应key，偏移量+1
                $tempMap = $nowMap;
                $matchFlag++;

                // 如果为最后一个匹配规则,结束循环，返回匹配标识数
                if (false === $nowMap['end']) {
                    continue;
                }

                return true;
            }

            // 找到相应key
            if ($matchFlag <= 0) {
                continue;
            }

            // 需匹配内容标志位往后移
            $length = $length + $matchFlag - 1;
        }
        return false;
    }
}
