# dfa
//检测敏感词并返回敏感词数据
$word_list = array(
    '蚂蚁',
    '大象',
    '秘密',
    '打卡',
    '测试',
);
$content = '树上有蚂蚁，草原有大象';
$filter_words = DfaFilter::init()->setTree($word_list)->getBadWord($content);
