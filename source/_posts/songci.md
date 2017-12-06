---
title: 宋词词频分析
date: 2015-05-03 09:48:20
tags: Python
---

　　今天早上在微信公众号『程序猿』看到一篇关于对古诗词词频分析的文章，很感兴趣，于是自己也实现了一个\~\~\~
　　首先在<a href='http://www.gushiwen.org/gushi/quansong.aspx' target='_blank'><u>这个页面</u></a>爬下《全宋词》，这一步通过`Python`和其丰富的库，很快就可以完成了，就不再赘述。爬下数据后，我将其保存到了一个文本文件中：
    ![请输入图片描述][1]


<!--more-->


　　《全宋词》的文本大小大约是4948KB，我保存的格式为每行一句（`，|。|、`等符号都看作每句的分隔符），文本文件共包含302385行宋词，这样保存方便之后的分析。
```Python
def main():
    with open('./song.txt', 'r') as songci_file:
        for line in songci_file:
            parse(line)
```
　　通过对每行宋词分析，汇总成最后的结果，使用一个全局的`dict`保存结果，其中dict的key为某个字或词，value为在《全宋词》中出现的次数。
```Python
def parse(line):
    global parsed_dict
    line = unicode(line.replace('\n', ''))
    for x in xrange(len(line)):
        for y in xrange(x, len(line) + 1):
            corrent = line[x:y]
            if not corrent:   # 爬下来数据中存在空字符
                continue
            if corrent in parsed_dict:  # 如果key已经在dict中，直接将value+1
                parsed_dict[corrent] += 1
            else:     # 否则，加入一个新的key
                parsed_dict[corrent] = 1
            # print 'now ------->>>>>>>', corrent
```
　　通过代码可以看出，对宋词的分析中，分词的处理并不是很圆满，目前我采用的办法是通过对每行内容进行排列组合，得到所有的结果，比如对『深院日斜』的分词就是：
```
深 深院 深院日 深院日斜
院 院日 院日斜
日 日斜
斜
```
　　这里分析的结果有2308541条，所以上面注释了`print`语句，因为大量的输出会严重影响程序的效率。
　　分析词频结束后，我将结果写入到了sqlite中：
```Python

def save_2_db():
    conn = sqlite3.connect('./song.db')
    total = len(parsed_dict)
    for index, key in enumerate(parsed_dict):
        # print '>>>>>>>>>>', key, parsed_dict[key], '<<<<<<<<<<'
        if parsed_dict[key] < 10:  # 低于10次出现的不保存
            continue
        print key, ':', parsed_dict[key], '<', index, '/', total, '>'
        if conn.execute('select * from song where key=?', (key,)).fetchall():
            continue
        conn.execute('insert into song values(?,?)', (key, parsed_dict[key]))
    conn.commit()
    conn.close()
```
　　在代码中，我限制了在《全宋词》中出现次数低于10次的词保存，因为大量的词出现次数很低，保存下来会使IO的时间变长且不必要。最后统计的结果，《全宋词》中超过10次出现的词语有25157个。
　　　　![请输入图片描述][2]
　　上图是宋词中出现最多的三个词(没有与单字比较)，正所谓『东风何处在人间』\~\~\~
　　　　![请输入图片描述][3] 
　　最后，实现一个函数，完成从一串数字字符串到对应宋词的转换：
```Python
def num_2_songci(text):
    to_search = []
    for x in xrange(0, len(text), 2):
        to_search.append(int(text[x:x+2]))

    conn = sqlite3.connect('song.db')
    for key in to_search:
        sql = 'select * from song where length(key)=2 order by value desc limit 1 offset ' + str(key-1)
        print conn.execute(sql).fetchall()[0][0],
    conn.close()
```
　　比如，试一下我的生日就是`num_2_songci('19940805')`，结果是`明月十分，春风风流。`，当然符号是我自己加上去的，是不是还有点宋词的韵味呢\~\~\~
　　下面是一首通过程序生成的“宋词”：
```
num_2_songci('4142135623730950488016887242097') # 根号2小数部分
风吹江仙
归来少年
阑干芙蓉梅花
不知不见
今日鹧鸪
玉楼功名江仙
梅花归去
```


  [1]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2015/02/13/songci.png
  [2]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2015/02/13/songcitop3.png
  [3]: http://7xkpi6.com1.z0.glb.clouddn.com/blog/2015/02/13/ssongresulr.png
